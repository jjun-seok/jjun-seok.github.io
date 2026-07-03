---
title: "Scikit-learn 핵심 도구 정리 - train_test_split부터 Pipeline까지"
date: 2026-07-03 12:00:00 +0900
categories: [이어드림스쿨, 머신러닝, scikit-learn]
tags: [scikit-learn, python, pipeline, preprocessing, columntransformer]
---

## 왜 이 도구들이 필요한가

모델을 학습시키기 전에 데이터를 "모델이 먹을 수 있는 형태"로 다듬어야 한다. 결측치가 있으면 연산이 아예 안 되고, 문자 데이터는 모델이 이해를 못 하고, 숫자 단위가 제각각이면 모델이 엉뚱한 걸 중요하다고 착각한다. 이번 글은 이 전처리 문제들을 해결하는 scikit-learn 핵심 도구 6가지를 정리한다.

---

## 1. train_test_split — 시험 문제 미리 안 보기

### 왜 나눠야 하나

모델을 학습시킨 데이터로 그대로 평가하면, 답안지를 미리 본 상태로 시험 보는 것과 같다. 모델이 진짜 실력이 좋은 건지, 그냥 외운 건지 구분이 안 된다. 그래서 데이터를 **훈련용**과 **검증용**으로 나눠서, 검증용은 모델이 한 번도 본 적 없는 "새 문제"로 취급한다.

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
# X_train, y_train: 80% — 훈련용
# X_test, y_test: 20% — 검증용 (모델이 못 본 데이터)
```

---

## 2. SimpleImputer — 빈칸 채우기

### 왜 필요한가

설문 누락, 기기 오류 등으로 데이터에 빈 칸(NaN)이 생기는 건 흔한 일이다. 빈 칸이 있으면 대부분의 연산이 에러를 낸다. `SimpleImputer`는 정해진 전략대로 빈 칸을 자동으로 채워준다.

| strategy | 채우는 값 | 언제 쓰나 |
|---|---|---|
| `'mean'` | 평균값 | 이상치 없는 숫자형 |
| `'median'` | 중앙값 | 이상치 있는 숫자형 (나이, 소득 등) |
| `'most_frequent'` | 최빈값 | 문자형(범주형) 데이터 |
| `'constant'` | 지정값(`fill_value`) | 특정 값으로 통일하고 싶을 때 |

```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy='median')
age_filled = imputer.fit_transform(age_column)
```

> 나이처럼 극단값(100살 등)이 섞일 수 있는 숫자형 데이터는 평균보다 **중앙값**이 안전하다. 평균은 극단값에 쉽게 끌려가지만 중앙값은 잘 안 흔들린다.
{: .prompt-tip }

---

## 3. StandardScaler — 저울 눈금 맞추기

### 왜 필요한가

키(150~190)와 몸무게(40~100)처럼 단위가 다른 두 변수를 그대로 모델에 넣으면, 모델은 숫자가 크다는 이유만으로 키를 더 중요한 변수로 착각할 수 있다. `StandardScaler`는 모든 변수의 **평균을 0, 표준편차를 1**로 맞춰서 공평한 저울 위에 올려놓는다.

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)   # 평균 0, 표준편차 1로 변환
```

---

## 4. OneHotEncoder — 글자를 숫자로 번역

### 왜 필요한가

모델은 수학 연산 기반이라 `'남자'`, `'서울'` 같은 문자를 계산에 못 쓴다. `OneHotEncoder`는 범주 하나당 새 컬럼을 만들어서 해당하면 1, 아니면 0으로 표시한다.

```python
from sklearn.preprocessing import OneHotEncoder

# ['NewYork', 'LosAngeles', 'Chicago'] → 3개의 0/1 컬럼으로 분리
encoder = OneHotEncoder(handle_unknown='ignore')
city_encoded = encoder.fit_transform(city_column)
```

`handle_unknown='ignore'`는 학습 때 못 본 새 범주가 검증 데이터에 나타나도 에러 없이 전부 0으로 처리하라는 옵션이다 — 실전 데이터에는 항상 예상 못 한 값이 섞여 들어오기 때문에 거의 필수로 켜둔다.

---

## 5. DecisionTreeClassifier — 스무고개로 분류하기

### 왜 필요한가

전처리가 끝났으면 실제로 학습할 모델이 필요하다. `DecisionTreeClassifier`는 "몸무게가 50kg 이상인가요?" → "줄무늬가 있나요?"처럼 데이터를 보고 질문을 스스로 만들어가며 분류 규칙(트리)을 학습하는 모델이다.

```python
from sklearn.tree import DecisionTreeClassifier

model = DecisionTreeClassifier(random_state=42)
model.fit(X_train_scaled, y_train)
accuracy = model.score(X_test_scaled, y_test)
```

---

## 6. Pipeline — 전처리와 모델을 하나로 묶기

### 왜 필요한가

전처리를 따로따로 실행하면 두 가지 문제가 생긴다.

1. **순서 실수**: train에는 `fit_transform`, test에는 `transform`만 써야 하는데 사람이 매번 헷갈리기 쉽다.
2. **데이터 누수**: test 데이터까지 포함해서 평균·표준편차를 계산해버리면, 모델이 실전엔 있을 수 없는 정보(미래 데이터)를 미리 알고 학습한 꼴이 된다.

`Pipeline`은 전처리와 모델 학습을 하나의 "컨베이어 벨트"로 묶어서, `fit()` 한 번으로 순서를 강제한다.

```python
from sklearn.pipeline import Pipeline

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('tree', DecisionTreeClassifier())
])

pipeline.fit(X_train, y_train)          # 스케일링 + 학습이 한 번에
pipeline.score(X_test, y_test)          # 스케일링 + 예측이 한 번에
```

> `Pipeline` 안의 이름(`'scaler'`, `'tree'`)은 그냥 이 단계를 부르는 별명이다. 나중에 `pipeline.named_steps['scaler']`처럼 특정 단계에 접근할 때 이 이름을 쓴다.
{: .prompt-tip }

**오류 코드 vs 정상 코드**

```python
# ❌ 데이터 누수 위험
scaler = StandardScaler()
X_all_scaled = scaler.fit_transform(X)  # train+test 전체로 fit → test 정보가 새어들어감
X_train, X_test = train_test_split(X_all_scaled, ...)
```

```python
# ✅ Pipeline은 train에만 fit, test는 transform만 자동으로 적용
pipeline.fit(X_train, y_train)
pipeline.predict(X_test)
```

---

## 7. ColumnTransformer — 컬럼별로 다른 전처리 적용하기

### 왜 필요한가

실제 데이터셋에는 숫자형과 문자형이 섞여 있다. 숫자형엔 `SimpleImputer(mean)` + `StandardScaler`, 문자형엔 `SimpleImputer(most_frequent)` + `OneHotEncoder`처럼 **컬럼마다 다른 전처리**가 필요하다. `ColumnTransformer`는 "어떤 컬럼엔 어떤 전처리"를 지정해서 한 번에 병렬로 적용해준다.

```python
from sklearn.compose import ColumnTransformer

numeric_features = ['pclass', 'fare', 'age']
categorical_features = ['embarked']

numeric_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

categorical_transformer = Pipeline([
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))
])

preprocessor = ColumnTransformer([
    ('num', numeric_transformer, numeric_features),
    ('cat', categorical_transformer, categorical_features)
])

# 전처리 전체 + 모델까지 하나로
full_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('model', DecisionTreeClassifier(random_state=42))
])
full_pipeline.fit(X_train, y_train)
```

### Pipeline vs ColumnTransformer — 헷갈리기 쉬운 지점

| | 처리 방식 | 비유 |
|---|---|---|
| **Pipeline** | 직렬 — 한 라인에서 순서대로 | 빈칸 채우기 → 단위 맞추기 |
| **ColumnTransformer** | 병렬 — 컬럼 종류별로 다른 라인 | 숫자 라인 / 문자 라인 따로 돌리고 합침 |

즉 `ColumnTransformer`가 컬럼을 종류별로 나눠서 각각의 `Pipeline`에 흘려보내고, 그 결과를 다시 합쳐서 하나의 표로 만드는 상위 개념이다.

---

## 오늘의 핵심 정리

| 도구 | 역할 | 한 줄 요약 |
|---|---|---|
| `train_test_split` | 데이터 분할 | 시험 문제 미리 안 보기 |
| `SimpleImputer` | 결측치 처리 | 빈칸을 규칙대로 채움 |
| `StandardScaler` | 스케일링 | 단위를 공평하게 맞춤 |
| `OneHotEncoder` | 인코딩 | 글자를 0/1 컬럼으로 번역 |
| `DecisionTreeClassifier` | 모델 | 스무고개로 분류 |
| `Pipeline` | 직렬 자동화 | 전처리+모델을 순서대로 묶음 |
| `ColumnTransformer` | 병렬 자동화 | 컬럼 종류별로 다른 전처리 적용 |

실무에서는 `ColumnTransformer`로 컬럼별 전처리를 정의하고, 그걸 다시 `Pipeline`에 넣어 모델까지 하나로 묶는 조합이 사실상 표준 패턴이다.
