---
title: "머신러닝 기초 I - NumPy·Pandas·Matplotlib·Seaborn과 타이타닉 로지스틱 회귀"
date: 2026-07-02 18:00:00 +0900
categories: [이어드림스쿨, 머신러닝]
tags: [python, numpy, pandas, matplotlib, seaborn, logistic-regression, titanic, statsmodels]
---

## 들어가며

이번 수업은 머신러닝을 시작하기 전에 꼭 알아야 하는 4개 라이브러리(NumPy, Pandas, Matplotlib, Seaborn)를 훑고, 그걸로 **타이타닉 생존 분석 + 로지스틱 회귀 모델**을 처음부터 끝까지 만들어보는 내용이었다. 특이한 점은 scikit-learn을 안 쓰고 **NumPy + Statsmodels**만으로 모델을 학습·평가했다는 것 — 그래서 "라이브러리가 알아서 해주는" 부분 없이 로지스틱 회귀가 실제로 뭘 하는 건지 눈으로 볼 수 있었다.

---

## Part 1. NumPy — 배열 연산의 기본

**NumPy**는 같은 자료형 데이터를 연속된 메모리에 저장해서 빠르게 연산하는 다차원 배열(array) 패키지다. 파이썬 리스트는 타입이 섞여도 되는 대신 `[1,2,3]*2`가 `[1,2,3,1,2,3]`이 되어버리는데(원소별 곱이 아님), NumPy 배열은 진짜 벡터 연산이 된다.

```python
import numpy as np

a = np.array([0, 1, 2, 3])              # 리스트 → 배열
b = np.arange(1, 8, 2)                  # 1~8, step=2 → [1,3,5,7]
c = np.zeros((3, 4))                    # 0으로 채운 3x4
d = np.random.randint(0, 10, size=5)    # 0~9 정수 난수 5개
np.random.seed(42)                      # 재현성 확보 (ML 필수)
```

핵심 속성·조작:

| 문법 | 의미 |
|---|---|
| `.shape` | 각 차원 크기 (행, 열) |
| `.reshape((r, c))` | 형태 변경, 원소 수는 그대로 유지해야 함 |
| `arr[:, 0]` | 모든 행, 0번째 열(슬라이싱) |
| `np.r_[a, b]` / `np.c_[a, b]` | 행/열 방향 연결 |
| `a + b`, `a * b` | 원소별(element-wise) 연산 |
| `a @ b`, `np.dot(a, b)` | 진짜 행렬 곱 |

> `*`는 원소별 곱이고 행렬 곱이 아니다. 행렬 곱은 반드시 `@` 또는 `np.dot()`을 써야 한다. 여기서 헷갈리면 shape 에러로 바로 티가 난다.
{: .prompt-tip }

---

## Part 2. Pandas — 표 형태 데이터 다루기

**Pandas**는 행·열로 이루어진 데이터를 다루는 데 특화된 패키지다. 두 가지 핵심 자료구조가 있다.

- **Series**: 인덱스가 붙은 1차원 배열
- **DataFrame**: Series 여러 개가 모인 2차원 표 (딕셔너리로 만드는 게 기본)

```python
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35]
})
```

### loc vs iloc — 여기서 실수 제일 많이 남

- **`loc[idx]`**: 라벨(이름) 기준. 슬라이싱하면 **끝 라벨도 포함**됨
- **`iloc[idx]`**: 정수 위치 기준. 파이썬 규칙대로 **끝 인덱스 미포함**

```python
df.loc['a':'b', ['A', 'C']]   # 'b'까지 포함
df.iloc[0:2, [0, 2]]          # 인덱스 2는 미포함
```

이 둘의 "끝 포함 여부" 차이를 모르고 쓰면 데이터가 한 줄씩 밀리는 버그가 생긴다.

### 자주 쓰는 함수

```python
df.drop(labels='A', axis=1)                 # 열 삭제 (axis=0 행, axis=1 열)
df.sort_values(by='A', ascending=False)      # 값 기준 정렬
df.to_csv('data.csv', index=False)           # 저장
pd.read_csv('data.csv')                      # 불러오기
```

---

## Part 3. Matplotlib — 기본 시각화

`import matplotlib.pyplot as plt` 관례로 불러오고, 그래프 종류별 함수만 갈아끼우면 된다.

```python
plt.plot(x, y, 'ro-')      # 선 그래프 (빨강, 원마커, 실선)
plt.scatter(x, y, s=크기, c=색상, cmap='viridis')  # 산점도
plt.hist(data, bins=10)    # 히스토그램 (연속형 데이터용)
plt.bar(x, height)         # 막대그래프 (범주형 데이터용)
```

**히스토그램 vs 막대그래프** 헷갈리기 쉬운데: 히스토그램은 x축이 값의 **구간**(연속형), 막대그래프는 x축이 **범주**(카테고리)다.

여러 그래프를 한 화면에 그릴 땐 두 가지 방식이 있다.

```python
# 방식 1: subplot(행, 열, 번호) — 번호는 1부터
plt.subplot(221)
plt.scatter(x, y)

# 방식 2: fig, axes — 더 명시적이고 요즘 더 많이 씀
fig, axes = plt.subplots(1, 2, figsize=(10, 4))
axes[0].plot(x, np.sin(x))
axes[1].plot(x, np.cos(x))
fig.suptitle('제목')
```

---

## Part 4. Seaborn — Matplotlib + 통계 시각화

Seaborn은 Matplotlib 위에 통계 차트·예쁜 테마를 얹은 패키지다 (`import seaborn as sns`). 내장 데이터셋(`sns.load_dataset('iris')`, `'titanic'`, `'tips'`)이 있어서 연습하기 좋다.

```python
sns.regplot(x='sepal_length', y='petal_length', data=df)   # 산점도 + 회귀선
sns.countplot(x='class', hue='survived', data=titanic)      # 범주형 빈도 + 그룹 구분
sns.jointplot(x='total_bill', y='tip', data=tips, kind='reg')  # 산점도+히스토그램 동시
```

`hue` 파라미터가 핵심이다 — x, y 외에 색으로 한 축을 더 쪼개 보여줘서, 예를 들어 "등급별 생존" 같은 3변수 관계를 한 그래프에 담을 수 있다.

---

## Part 5. 타이타닉 생존 분석 & 로지스틱 회귀

여기서부터가 진짜 메인. **scikit-learn 없이** NumPy + Statsmodels로 이진 분류 모델을 처음부터 끝까지 만든다.

### 로지스틱 회귀란?

생존(1)/사망(0)처럼 **이진 분류**를 위한 통계 모델. 선형회귀처럼 계수를 곱해 더한 값을 시그모이드 함수에 넣어서 0~1 사이 확률로 변환한다.

$$P(y=1) = \frac{1}{1 + e^{-(b_0 + b_1x_1 + b_2x_2 + \cdots)}}$$

값이 클수록 시그모이드가 1에 가까워지고, 작을수록(음수로 갈수록) 0에 가까워지는 구조다.

### 1) 데이터 탐색 (EDA)

```python
titanic = sns.load_dataset('titanic')
titanic.isnull().sum()          # 결측치 확인
titanic['survived'].mean()      # 전체 생존율
```

살펴본 질문들: 등급이 생존에 영향을 주는가? 성별은? 나이 분포는? → `countplot`, `groupby().mean()`, 히스토그램, 상관 히트맵(`sns.heatmap`)으로 하나씩 확인했다. 결과적으로 **여성·1등급일수록 생존율이 높다**는, 잘 알려진 "여성과 어린이 먼저" 패턴이 데이터로도 확인됐다.

### 2) 전처리 — 모델이 이해할 수 있는 숫자로

모델은 문자와 결측치를 못 먹는다. 세 단계로 정리했다.

```python
# 1. 특성 선택
cols = ['survived', 'pclass', 'sex', 'age', 'sibsp', 'parch', 'fare']
df_model = titanic[cols].copy()

# 2. 결측치 → 중앙값으로 채움 (평균보다 이상치에 덜 흔들림)
df_model['age'] = df_model['age'].fillna(df_model['age'].median())
df_model['fare'] = df_model['fare'].fillna(df_model['fare'].median())

# 3. 범주형 인코딩: male→0, female→1
df_model['sex'] = df_model['sex'].map({'male': 0, 'female': 1})
```

### 3) 학습/검증 분할 (직접 구현)

sklearn의 `train_test_split()` 없이 NumPy로 직접 셔플해서 나눴다.

```python
np.random.seed(42)
indices = np.arange(len(df_model))
np.random.shuffle(indices)              # 순서 편향 방지 — 꼭 필요

train_size = int(0.8 * len(df_model))
train_idx, test_idx = indices[:train_size], indices[train_size:]
```

### 4) Statsmodels로 로지스틱 회귀 적합

```python
import statsmodels.api as sm

X_train_sm = sm.add_constant(X_train)   # 절편(intercept) 항 직접 추가해야 함
logit_model = sm.Logit(y_train, X_train_sm)
result = logit_model.fit()              # 최대우도추정(MLE)으로 계수 계산
print(result.summary())
```

> `sm.add_constant()`를 빼먹으면 절편 없는 모델이 되어버린다. scikit-learn은 절편을 자동으로 넣어주지만 Statsmodels는 명시적으로 추가해야 하는 게 큰 차이점.
{: .prompt-warning }

### 5) 계수 해석 — 오즈비(Odds Ratio)

계수 자체보다 `np.exp(계수)`인 **오즈비**로 해석하는 게 직관적이다.

- 계수 > 0 → 생존 확률에 긍정적 영향
- 오즈비가 1보다 크면 "해당 변수가 1 늘 때 생존 오즈가 몇 배"라는 뜻

예를 들어 `sex`(여성=1)의 오즈비가 컸다는 건, 여성일 때 생존 오즈가 남성보다 훨씬 높다는 뜻이다.

### 6) 예측과 평가 — 전부 NumPy로 수동 계산

```python
y_pred_prob = result.predict(X_test_sm)      # 확률(0~1) 예측
y_pred = (y_pred_prob >= 0.5).astype(int)    # 임계값 0.5로 이진 변환

# 혼동 행렬 직접 계산
TP = np.sum((y_pred == 1) & (y_test == 1))
TN = np.sum((y_pred == 0) & (y_test == 0))
FP = np.sum((y_pred == 1) & (y_test == 0))
FN = np.sum((y_pred == 0) & (y_test == 1))

accuracy = np.mean(y_pred == y_test)
```

혼동 행렬(Confusion Matrix)은 결국 이 4가지 경우의 수 조합이다.

| | 예측 0 | 예측 1 |
|---|---|---|
| **실제 0** | TN (맞춤) | FP (헛다리) |
| **실제 1** | FN (놓침) | TP (맞춤) |

### 7) 특성 중요도 & 모델 저장

계수의 절댓값이 클수록 생존 예측에 영향이 큰 변수다. 마지막으로 로지스틱 회귀는 **계수만 있으면 모델이 완전히 재현**되기 때문에, 계수를 CSV로 저장해두면 재학습 없이 Streamlit 같은 앱에서 바로 불러와 예측 함수만 다시 짜면 된다.

```python
model_params = pd.DataFrame({
    'feature': ['const'] + feature_cols,
    'coefficient': np.asarray(result.params)
})
model_params.to_csv('titanic_model_params.csv', index=False)
```

---

## 오늘의 핵심 정리

1. **NumPy**는 벡터 연산, **Pandas**는 표 데이터, **Matplotlib/Seaborn**은 시각화 — 이 넷이 데이터 분석의 기본 스택이다.
2. `loc`은 끝 포함, `iloc`은 끝 미포함 — 헷갈리면 무조건 버그.
3. 로지스틱 회귀는 선형결합 값을 시그모이드에 넣어 확률로 바꾸는 이진분류 모델.
4. Statsmodels는 절편을 수동으로 추가해야 하고, 대신 `summary()`로 p-value·계수 유의성까지 통계적으로 확인할 수 있다는 게 sklearn과 다른 장점.
5. 정확도 하나만 보지 말고 혼동 행렬로 어떤 종류의 오답(FP/FN)이 많은지 봐야 한다.

---

## 참고

- [NumPy 공식 문서](https://numpy.org/doc/stable/)
- [Pandas 공식 문서](https://pandas.pydata.org/docs/)
- [Seaborn 공식 문서](https://seaborn.pydata.org/)
- [Statsmodels - Logit 공식 문서](https://www.statsmodels.org/stable/generated/statsmodels.discrete.discrete_model.Logit.html)
