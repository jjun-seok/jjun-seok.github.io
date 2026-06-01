---
title: "Python 기초 자료형 II - Tuple, Dictionary, 메서드"
date: 2026-05-28 15:00:00 +0900
categories: [이어드림스쿨, Python]
tags: [python, tuple, dictionary, split, join, pop]
---

# 1. 문자열 / 리스트 활용 메서드

## list.pop(i) — 원소 제거 & 반환
- 인덱스 `i`의 원소를 **제거**하고 그 값을 **반환**
- 괄호를 비우면 **마지막** 원소 처리

```python
my_list = [2, 3, 5, 7, 11]

removed_first = my_list.pop(0)   # 인덱스 0 제거
print("pop(0) 반환값:", removed_first)   # 2

removed_last = my_list.pop()     # 마지막 원소 제거
print("pop() 반환값:", removed_last)     # 11

print("남은 리스트:", my_list)   # [3, 5, 7]
```

## seq.count(d) — 원소 개수 세기

```python
# 리스트에서 count
my_seq = [2, 2, 2, 4, 4]
print("2의 개수:", my_seq.count(2))   # 3

# 문자열에서 count
my_str = "elice and alice"
print("'e' 개수:", my_str.count("e"))  # 3
```

## str.split(c) — 문자열 쪼개기
- 구분자 `c`를 기준으로 문자열을 **리스트**로 반환
- 괄호를 비우면 **공백** 기준

```python
# 공백 기준 분리
my_str = "1 2 3 4 5"
print(my_str.split())           # ['1', '2', '3', '4', '5']

# 쉼표 기준 분리
elements = "Na,Mg,Al,Si"
print(elements.split(","))      # ['Na', 'Mg', 'Al', 'Si']
```

## str.join(list) — 리스트 이어붙이기
- `str`을 **구분자**로 삼아 리스트 원소들을 하나의 문자열로 합침

```python
chars = ["e", "l", "i", "c", "e"]
print("".join(chars))          # elice

stations = ["서울", "대전", "대구"]
print("→".join(stations))      # 서울→대전→대구
```

## 실습 예제: split + join 응용
```python
sentence = "파이썬 공부 열심히 하자"
words = sentence.split()

new_words = []
for w in words:
    new_words.append("★" + w + "★")

print(" ".join(new_words))
# ★파이썬★ ★공부★ ★열심히★ ★하자★
```

---

# 2. Tuple (튜플)

> 값을 바꿀 수 없으면서 여러 자료를 담고 싶을 때!

- 소괄호 `()` 로 표현
- 원소가 1개일 때는 반드시 **콤마** 필요: `(1,)`
- **추가·삭제·변경 불가** → 불변(immutable) 자료형

```python
tuple_zero   = ()
tuple_one    = (1,)           # 원소 1개 → 반드시 쉼표!
tuple_sample = (1, 2, 3, 4, 5)

print(type(tuple_sample))     # <class 'tuple'>
```

## 인덱싱, 슬라이싱, in, len
```python
my_tuple = ("A", "C", "E")

print(my_tuple[1])       # C
print(my_tuple[:2])      # ('A', 'C')
print("A" in my_tuple)   # True
print(len(my_tuple))     # 3

t1 = ("G", "O")
t2 = ("O", "D")
print(t1 + t2)           # ('G', 'O', 'O', 'D')
print(t1 * 3)            # ('G', 'O', 'G', 'O', 'G', 'O')
```

## 리스트 vs 튜플 비교

| | 리스트 | 튜플 |
|---|---|---|
| 표현 | `[1, 2, 3]` | `(1, 2, 3)` |
| 변경 | **가능** (mutable) | **불가** (immutable) |
| 용도 | 자주 변경되는 데이터 | 변경 없는 고정 데이터 |

```python
my_list  = [1, 2, 3]
my_list[0] = 99
print("리스트 변경 후:", my_list)   # [99, 2, 3]

my_tuple = (1, 2, 3)
my_tuple[0] = 99   # TypeError 발생!
```

---

# 3. Dictionary (딕셔너리)

> 사전처럼 **"단어(Key)"** 와 **"뜻(Value)"** 이 짝을 이루는 자료형

- 중괄호 `{}` + 콜론 `:` 으로 표현
- **Key**는 변할 수 없는 자료형 (문자열, 숫자, 튜플 가능 / 리스트 불가)
- `dict[Key]` 로 Value 접근

```python
person = {"name": "Rabbit", "age": 22}
print(person)          # {'name': 'Rabbit', 'age': 22}
print(type(person))    # <class 'dict'>
```

## CRUD (생성, 조회, 수정, 삭제)
```python
person = {"name": "Rabbit", "age": 22}

# 조회
print(person["name"])   # Rabbit

# 추가
person["mail"] = "rabbit@elice.com"
print(person)

# 수정
person["name"] = "Dennis"
print(person)

# 삭제
del person["age"]
print(person)
```

## keys(), values(), items()
```python
person = {"name": "Rabbit", "age": 22, "mail": "rabbit@elice.com"}

print(person.keys())    # dict_keys(['name', 'age', 'mail'])
print(person.values())  # dict_values(['Rabbit', 22, 'rabbit@elice.com'])
print(person.items())   # dict_items([('name', 'Rabbit'), ...])
```

## 실습 예제: 단어 빈도 세기
```python
sentence = "apple banana apple cherry banana apple"
words = sentence.split()

freq = {}
for word in words:
    if word in freq:
        freq[word] = freq[word] + 1
    else:
        freq[word] = 1

print("단어 빈도:", freq)
# {'apple': 3, 'banana': 2, 'cherry': 1}
```

---

# 4. for 문과 Dictionary

## 방법 1: zip (키, 값 묶기)
```python
my_dict = {"a": 11, "b": 13, "c": 17}

for key, value in zip(my_dict.keys(), my_dict.values()):
    print(f"key: {key}, value: {value}")
```

## 방법 2: items() — 권장 ✅
```python
for key, value in my_dict.items():
    print(f"key: {key}, value: {value}")
```

## 두 리스트를 딕셔너리로
```python
list1 = ["A", "B", "C", "D"]
list2 = [11, 13, 17, 19]

result = dict(zip(list1, list2))
print(result)   # {'A': 11, 'B': 13, 'C': 17, 'D': 19}
```

## 실습 예제: 성적표 분석
```python
grades = {"수학": 95, "영어": 88, "과학": 92, "국어": 79, "역사": 85}

# 평균
total = sum(grades.values())
average = total / len(grades)
print(f"평균: {average:.1f}")   # 87.8

# 90점 이상 과목
print("90점 이상 과목:")
for subject, score in grades.items():
    if score >= 90:
        print(f"  - {subject}: {score}점")
```

---

# 핵심 정리

| 자료형 | 표현 | 변경 | 특징 |
|--------|------|------|------|
| List | `[1,2,3]` | 가능 | 순서O, 중복O |
| Tuple | `(1,2,3)` | **불가** | 순서O, 불변 |
| Dictionary | `{k:v}` | 가능 | Key-Value 쌍 |

```python
list.pop(i)        # 인덱스 i 원소 제거 & 반환
seq.count(d)       # d의 개수
str.split(c)       # c 기준 분리 → 리스트
str.join(list)     # str 기준 리스트 합치기 → 문자열
dict.keys()        # 키 목록
dict.values()      # 값 목록
dict.items()       # (키, 값) 쌍 목록
```
