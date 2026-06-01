---
title: "Python 함수와 메서드 - def, return, 타입힌트"
date: 2026-05-28 16:00:00 +0900
categories: [이어드림스쿨, Python]
tags: [python, 함수, def, return, 타입힌트, docstring]
---

# 1. 함수란?

> **특정 기능을 수행하는 코드(들의 모임)**

- 입력(매개변수) → 작용(기능) → 출력(반환값)
- 반복되는 코드를 **묶어서 재사용** 가능
- 프로그램을 **모듈화**하여 가독성 향상

---

# 2. 내장 함수 (Built-in Functions)

파이썬이 미리 만들어 둔 함수 → 그냥 가져다 쓰면 됨!

```python
# len(), int(), str(), float()
my_list = ["a", "b", "c"]
print(len(my_list))          # 3
print(int("1234") + 1)       # 1235
print(str(5678))              # "5678"

# max(), min()
data = (3, 1, 4, 1, 5, 9)
print(max(data))   # 9
print(min(data))   # 1

# sum() + len() 으로 평균
scores = [85, 92, 78, 95, 88]
average = sum(scores) / len(scores)
print(f"평균: {average:.2f}")   # 87.60
```

---

# 3. 사용자 정의 함수

```python
def 함수이름(매개변수):
    <수행할 명령>
    return 반환값
```

- `def` : define(정의하다)의 약자
- **매개변수(parameter)**: 함수 내부로 값을 전달하는 변수
- `return`: 함수 외부로 값을 전달 (없으면 `None` 반환)

```python
def plusDouble(a, b):
    c = a + b
    return 2 * c

result = plusDouble(3, 4)
print(result)   # 14
```

## return이 왜 필요할까?

```python
# return 없을 때 → 함수 밖에서 결과 사용 불가
def formula_no_return(a, b):
    c = (a**2) + (b**2)
    print("함수 안:", c)

result = formula_no_return(2, 4)
print("함수 밖:", result)    # None ← 반환값 없으면 None!

# return 있을 때 → 결과를 변수에 저장 가능
def formula_with_return(a, b):
    c = (a**2) + (b**2)
    return c

result = formula_with_return(2, 4)
print("함수 밖:", result)    # 20
```

## 매개변수 여러 개
```python
def greet(name, age):
    message = f"안녕하세요! {name}님, {age}살이시군요."
    return message

print(greet("Alice", 25))
print(greet("Bob", 30))
```

## 실습 예제: 섭씨 → 화씨 변환
```python
def celsius_to_fahrenheit(celsius):
    fahrenheit = celsius * 9 / 5 + 32
    return fahrenheit

print(celsius_to_fahrenheit(0))    # 32.0
print(celsius_to_fahrenheit(100))  # 212.0
```

## 실습 예제: 회문(Palindrome) 판별
```python
def is_palindrome(text):
    reversed_text = text[::-1]      # 문자열 뒤집기
    return text == reversed_text

print(is_palindrome("level"))    # True
print(is_palindrome("racecar"))  # True
print(is_palindrome("hello"))    # False
```

---

# 4. 전역 변수와 지역 변수

| 구분 | 전역 변수 | 지역 변수 |
|------|-----------|-----------|
| 정의 위치 | 함수 **밖** | 함수 **안** |
| 사용 범위 | **어디서든** 사용 가능 | **함수 안에서만** 사용 |
| 함수 종료 후 | 유지 | **소멸** |

```python
score = 100   # 전역 변수

def update_score(new_score):
    score = new_score   # 지역 변수 (전역과 이름만 같음)
    print("함수 안 score:", score)   # 200

update_score(200)
print("함수 밖 score:", score)   # 100 ← 전역은 그대로!
```

---

# 5. 메서드 vs 함수

| | 함수 | 메서드 |
|---|---|---|
| 호출 방식 | `함수명(인자)` | `자료.메서드명(인자)` |
| 특징 | 범용적 기능 | **특정 자료형**에 속한 기능 |
| 예시 | `len(my_list)` | `my_list.append(4)` |

```python
my_list = [3, 1, 4, 1, 5]

# 함수
print(len(my_list))    # 5
print(sum(my_list))    # 14

# 메서드
my_list.append(9)
my_list.sort()
print(my_list.count(1))   # 2
```

## 문자열 메서드
```python
text = "  Hello, Python World!  "

print(text.upper())                    # 대문자
print(text.lower())                    # 소문자
print(text.strip())                    # 앞뒤 공백 제거
print(text.replace("Python", "파이썬")) # 치환
```

---

# 6. 타입 힌트 & Docstring

## 타입 힌트 (Type Hints)
함수의 매개변수와 반환값의 자료형을 명시해 가독성 향상

```python
# 타입 힌트 없음 → 자료형 불명확
def add(a, b):
    return a + b

# 타입 힌트 있음 → int 두 개를 받아 int를 반환
def add(a: int, b: int) -> int:
    return a + b

# 다양한 예시
def greet(name: str) -> str:
    return "안녕하세요, " + name

def is_even(n: int) -> bool:
    return n % 2 == 0

def average(nums: list) -> float:
    return sum(nums) / len(nums)

def log_message(msg: str) -> None:    # 반환값 없으면 None
    print("[LOG]", msg)
```

## Docstring — 함수 설명 문서
함수의 역할, 매개변수, 반환값을 자연어로 설명. `help(함수명)` 으로 확인 가능

```python
def celsius_to_fahrenheit(celsius: float) -> float:
    """섭씨 온도를 화씨 온도로 변환합니다.

    Args:
        celsius (float): 변환할 섭씨 온도 값

    Returns:
        float: 변환된 화씨 온도 값

    Example:
        >>> celsius_to_fahrenheit(100)
        212.0
    """
    return celsius * 9 / 5 + 32
```

## 타입 힌트 + Docstring 완전체 (실무 스타일)

```python
def get_grade(score: int) -> str:
    """점수를 받아 학점 문자열을 반환합니다.

    Args:
        score (int): 0~100 사이의 시험 점수

    Returns:
        str: 학점 문자열 ("A", "B", "C", "D", "F")

    Raises:
        ValueError: score가 0~100 범위를 벗어난 경우

    Example:
        >>> get_grade(95)
        'A'
    """
    if not (0 <= score <= 100):
        raise ValueError(f"score는 0~100이어야 합니다.")
    if score >= 90:   return "A"
    elif score >= 80: return "B"
    elif score >= 70: return "C"
    elif score >= 60: return "D"
    else:             return "F"
```

---

# 핵심 정리

## 함수 구조
```python
def 함수이름(매개변수1, 매개변수2):
    결과 = 처리 로직
    return 결과      # 반환값 없으면 None
```

## 내장 함수 요약

| 함수 | 기능 |
|------|------|
| `print()` | 출력 |
| `input()` | 입력 (항상 str 반환) |
| `len()` | 길이 |
| `sum()` | 합계 |
| `max()` / `min()` | 최대/최소 |
| `int()` / `str()` / `float()` | 형 변환 |

## 변수 범위
- **전역 변수**: 함수 밖에서 선언 → 어디서나 사용
- **지역 변수**: 함수 안에서 선언 → 함수 안에서만 사용
