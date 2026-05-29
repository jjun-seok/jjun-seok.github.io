---
title: "Python 기초 - 자료형, 조건문, 리스트"
date: 2026-05-28 13:00:00 +0900
categories: [이어드림스쿨, Python]
tags: [python, 자료형, 조건문, 리스트]
---

# 1. 기초 자료형

## 핵심 개념
- `print()` 로 값을 출력, 콤마(`,`)로 여러 값을 한 줄에 출력
- **숫자형(Number):** 정수 `int`, 실수 `float` — 사칙연산 및 `//` `%` `**` 특수연산 지원
- **문자열(String):** 따옴표로 감싼 텍스트 — `+`(이어붙이기), `*`(반복) 연산 가능
- **리스트(List):** `[]` 안에 여러 자료를 순서대로 보관 — 다른 자료형 혼합 가능
- **변수(Variable):** `변수명 = 값` 으로 선언
- `type()` 으로 자료형 확인, `str()` · `int()` · `float()` 으로 형 변환

## 1.1 출력 — `print()`

```python
# 기본 출력
print("이어드림스쿨6기 화이팅")

# 콤마로 여러 값 한 줄 출력
print(3, "안녕")

# 계산식도 바로 출력 가능
print("1+1=", 1+1)
```

## 1.2 기본 자료형

```python
print(type(3))       # <class 'int'> 정수
print(type(3.14))    # <class 'float'> 실수
print(type('a'))     # <class 'str'> 문자열
print(type([1,2,3])) # <class 'list'> 리스트
print(type(True))    # <class 'bool'> 논리형
```

## 1.3 변수 (Variable)

- **선언 방법:** `변수명 = 값`
- **이름 규칙:** 숫자로 시작 불가 / 예약어 금지 / 공백·연산자 불가
- **표기법 권장:** 스네이크(`my_name`) — Python 공식 스타일

```python
num = 10
name = "Dennis"
grade = ["A+", "A", "A0"]

# 스네이크 표기법 (권장)
prod_name = "milk"
prod_date = "2026-05-28"
```

### 변수 값 변경

```python
score = 50
print("처음 점수:", score)   # 50

score = 100  # 덮어쓰기
print("바뀐 점수:", score)   # 100
```

## 1.4 자료형의 연산

| 연산자 | 의미 | 예시 | 결과 |
|--------|------|------|------|
| `+` `-` `*` `/` | 사칙연산 | `3 + 5` | `8` |
| `//` | 몫 | `13 // 5` | `2` |
| `%` | 나머지 | `13 % 5` | `3` |
| `**` | 제곱 | `2 ** 4` | `16` |
| `+` (문자열) | 이어붙이기 | `"안녕" + "하세요"` | `"안녕하세요"` |
| `*` (문자열) | 반복 | `"안녕" * 3` | `"안녕안녕안녕"` |

```python
print(3 * 5)       # 15 (정수)
print(3 * 5.0)     # 15.0 (실수)
print(10 / 5)      # 2.0 (나눗셈은 항상 실수)

print("안녕" + "하세요")  # 안녕하세요
print("안녕" * 3)         # 안녕안녕안녕
```

## 1.5 인덱싱 & 슬라이싱

- **인덱스:** 0부터 시작, 음수 인덱스는 뒤에서부터
- **인덱싱:** `자료[인덱스]` → 특정 위치의 원소 하나
- **슬라이싱:** `자료[시작:끝]` → 시작 이상, 끝 **미만** 범위

```
"Ready"  →  R  e  a  d  y
인덱스   →  0  1  2  3  4  (양수)
           -5 -4 -3 -2 -1  (음수)
```

```python
word    = "Ready"
numbers = [2, 4, 6, 8, 10]

# 인덱싱
print(word[0])      # R
print(word[2])      # a
print(numbers[-3])  # 6 (뒤에서 3번째)

# 슬라이싱
beta = [2, 4, 6, 8, 10, 12, 14]
print(beta[0:3])  # [2, 4, 6]
print(beta[:3])   # [2, 4, 6]
print(beta[2:])   # [6, 8, 10, 12, 14]
print(beta[:])    # [2, 4, 6, 8, 10, 12, 14]
```

---

# 2. 조건문

## 핵심 개념
- `input()` 반환값은 항상 **문자열** → 숫자로 쓰려면 `int()` / `float()` 변환 필수
- **비교 연산자:** `==` `!=` `<` `>` `<=` `>=` → 결과는 `True` or `False`
- **논리 연산자:** `and`(모두 True), `or`(하나라도 True), `not`(반전)
- **들여쓰기(4칸)** 로 블록 구분 — Python의 핵심 문법

## 2.1 비교·논리 연산자

| 연산자 | 의미 | 예시 | 결과 |
|--------|------|------|------|
| `==` | 같다 | `3 == 3` | `True` |
| `!=` | 다르다 | `3 != 5` | `True` |
| `<` `>` | 크기 비교 | `3 < 5` | `True` |
| `and` | 모두 True | `True and False` | `False` |
| `or` | 하나라도 True | `True or False` | `True` |
| `not` | 반전 | `not True` | `False` |

```python
print(3 < 5)             # True
print(3==3 and 4<=5)     # True
print(3==4 or 4<=5)      # True
print(not 3==4)          # True
```

## 2.2 if / if-else / if-elif-else

```python
# if-else 기본
i = 1
if i == 1:
    print("i는 1입니다")
else:
    print("i는 1이 아닙니다")

# if-elif-else: 성적 판별
score = input("점수를 입력하세요: ")
if score == "A":
    print("참 잘했어요")
elif score == "B":
    print("잘했어요")
elif score == "C":
    print("노력하세요")
else:
    print("꽝")
```

### BMI 계산기 예제

```python
weight = float(input("체중을 입력하세요:"))
height = float(input("키를 입력하세요:"))

bmi = weight / ((height / 100) ** 2)

if bmi < 18.5:
    print("저체중")
elif bmi < 23.0:
    print("정상")
else:
    print("비만")
```

---

# 3. 리스트

## 핵심 개념
- **`append(d)`:** 맨 끝에 원소 하나 추가
- **`insert(i, d)`:** 인덱스 `i` 위치에 원소 삽입
- **`remove(d)`:** 첫 번째로 등장하는 `d` 삭제
- **`sort()`:** 오름차순 정렬, `reverse=True` 로 내림차순

## 3.1 리스트 핵심 메서드

```python
# append - 맨 끝에 추가
shopping = []
shopping.append("치킨")
shopping.append("치킨무")
print(shopping)  # ['치킨', '치킨무']

# insert - 원하는 위치에 삽입
fruits = ["banana", "cherry"]
fruits.insert(0, "사과")
print(fruits)  # ['사과', 'banana', 'cherry']

# remove - 첫 번째 해당 원소 삭제
my_list = [3, 1, 2, 3]
my_list.remove(3)
print(my_list)  # [1, 2, 3] (첫 번째 3만 삭제)

# sort - 정렬
e = [6, 2, 4, 1, 3, 5]
e.sort()
print("오름차순", e)        # [1, 2, 3, 4, 5, 6]
e.sort(reverse=True)
print("내림차순", e)        # [6, 5, 4, 3, 2, 1]
```

## 3.2 시퀀스 자료형 공통 특징

문자열(`str`)과 리스트(`list`) 모두 아래 기능 지원:

```python
a = "Hello"
b = ["e", "l", "i", "c", "e"]

# in 연산자
print("e" in a)      # True
print("z" not in a)  # True

# len()
print(len("Hello World"))  # 11
print(len(b))               # 5

# + 이어붙이기
print("Good" + "Day")               # GoodDay
print(["H", "e"] + ["l", "l", "o"]) # ['H','e','l','l','o']

# * 반복
print("Go!!" * 3)          # Go!!Go!!Go!!
print([1, 2] * 3)           # [1, 2, 1, 2, 1, 2]
```

> ⚠️ `list.append()` 는 가능하지만 `str.append()` 는 불가! (`AttributeError` 발생)

---

# 정리

| 주제 | 핵심 포인트 |
|------|------------|
| **기초 자료형** | `int` `float` `str` `list` `bool` · `type()` · 형 변환 |
| **변수** | `변수 = 값` · 스네이크 표기법 권장 · 덮어쓰기 가능 |
| **연산** | 숫자: `//` `%` `**` · 문자열: `+` 이어붙이기, `*` 반복 |
| **인덱싱·슬라이싱** | 0부터 시작 · 음수 인덱스 · `[시작:끝]` |
| **조건문** | `if` - `elif` - `else` · 들여쓰기 필수 · `and` `or` `not` |
| **리스트 메서드** | `append` · `insert` · `remove` · `sort(reverse=True)` |
| **시퀀스 공통** | `in` · `len()` · `+` · `*` |

> 다음 단계: 반복문(`for`, `while`) → 함수(`def`) → 딕셔너리(`dict`)
