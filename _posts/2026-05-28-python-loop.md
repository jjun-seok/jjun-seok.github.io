---
title: "Python 반복문 - for, while, break"
date: 2026-05-28 14:00:00 +0900
categories: [이어드림스쿨, Python]
tags: [python, for, while, break, range]
---

# 반복문이 필요한 이유

별을 100개 출력해야 한다면? 일일이 `print`를 100번 쓰는 건 비효율적이다. 반복문으로 해결!

```python
# 반복 없이 → 코드가 길어짐
print("*")
print("**")
print("***")

# 반복문 사용 → 간결하게!
for i in range(1, 6):
    print("*" * i)
```

---

# 1. for 문

```
for 변수 in 시퀀스:
    <수행할 명령>
```

- 시퀀스(리스트 등)의 원소를 **하나씩** 변수에 넣어가며 실행
- **들여쓰기**로 반복 범위 구분

```python
# 기본 예제: 합계 구하기
sum_val = 0
for i in [1, 3, 5]:
    sum_val = sum_val + i
print("합계:", sum_val)   # 9

# 리스트 원소 개수 직접 세기
length = 0
for i in [1, 3, 5]:
    length = length + 1
print("길이:", length)    # 3
```

---

# 2. for-range 문

## range() 함수
- `range(a, b)` → a 이상 **(b-1)** 이하 숫자 생성
- `range(a)` → 0부터 **(a-1)** 까지 (a번 반복)

```python
print(list(range(1, 9)))  # [1, 2, 3, 4, 5, 6, 7, 8]
print(list(range(5)))     # [0, 1, 2, 3, 4]
```

## 패턴 1: 구간 반복 `range(a, b)`
```python
num_list = [1]
for i in range(2, 5):     # 2, 3, 4
    num_list.append(i)
print(num_list)            # [1, 2, 3, 4]
```

## 패턴 2: 횟수 반복 `range(a)`
```python
count = 0
for i in range(10):        # 10번 반복
    count += 1
print("반복 횟수:", count)  # 10
```

## 패턴 3: `len()` + 인덱싱
```python
str_list = ["a", "b", "c", "d"]

for idx in range(len(str_list)):
    print(str_list[idx], end=" ")   # a b c d
```

## 실습 예제: 구구단
```python
dan = 3
for i in range(1, 10):
    print(f"{dan} x {i} = {dan * i}")
```

## 실습 예제: 짝수 합
```python
even_sum = 0
for i in range(1, 21):
    if i % 2 == 0:
        even_sum += i
print("1~20 짝수 합:", even_sum)   # 110
```

---

# 3. while 문

```
while 조건:
    <수행할 명령>
```

- **조건이 True인 동안** 반복 실행
- `for` → 횟수/범위가 정해진 경우 / `while` → 조건이 있는 경우

```python
# 카운트다운
i = 5
while i > 0:
    print(i)
    i = i - 1    # 반드시 i를 줄여야 함!
print("Launch!")

# 1부터 4까지 더하기
i = 1
total = 0
while i < 5:
    total = total + i
    i = i + 1
print("합계:", total)   # 10
```

> ⚠️ **무한루프 주의!** `i`가 변하지 않으면 조건이 항상 True → 무한 반복

---

# 4. break 문

- 반복문을 **강제로 탈출**할 때 사용
- 보통 `if` 조건과 함께 사용

```python
i = 1
while i > 0:
    print(i)
    if i >= 5:    # i가 5 이상이 되면 탈출
        break
    i = i + 1
print("종료!")
```

---

# 종합 실습

## FizzBuzz
```python
for i in range(1, 31):
    if i % 15 == 0:     # 15의 배수 먼저 확인!
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

## 최대값 찾기 (max() 없이)
```python
numbers = [34, 78, 12, 99, 45, 67, 23]
current_max = numbers[0]

for n in numbers:
    if n > current_max:
        current_max = n

print("최대값:", current_max)   # 99
```

## 소수 판별기
```python
primes = []

for num in range(2, 31):
    is_prime = True
    for divisor in range(2, num):
        if num % divisor == 0:
            is_prime = False
            break
    if is_prime:
        primes.append(num)

print("2~30 소수:", primes)
```

---

# 핵심 정리

| 구문 | 용도 | 패턴 |
|------|------|------|
| `for i in 시퀀스` | 원소 순회 | 리스트, 문자열 등 |
| `for i in range(n)` | n번 반복 | 횟수 기반 |
| `for i in range(a,b)` | 구간 반복 | a 이상 b 미만 |
| `while 조건` | 조건 기반 반복 | 조건이 True인 동안 |
| `break` | 반복 탈출 | if 와 조합 |
