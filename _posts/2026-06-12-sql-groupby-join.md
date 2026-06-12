---
title: "SQL - GROUP BY, HAVING, JOIN 완전 정리"
date: 2026-06-12 09:00:00 +0900
categories: [이어드림스쿨, SQL]
tags: [sql, groupby, having, join, innerjoin, leftjoin, rightjoin]
---

# 1. GROUP BY — 데이터 그룹 짓기

> 같은 값을 가진 행끼리 묶어서 집계하는 명령어

```sql
SELECT user_id, COUNT(*)
FROM rental
GROUP BY user_id;
```

## 그룹 함수와 함께 사용

| 함수 | 설명 |
|------|------|
| `COUNT(*)` | 행 개수 |
| `SUM(컬럼)` | 합계 |
| `AVG(컬럼)` | 평균 |
| `MAX(컬럼)` | 최대값 |
| `MIN(컬럼)` | 최소값 |

```sql
-- user_id별 대여 합계
SELECT user_id, SUM(book_id) FROM rental GROUP BY user_id;

-- user_id별 평균
SELECT user_id, AVG(book_id) FROM rental GROUP BY user_id;

-- user_id별 최대/최소
SELECT user_id, MAX(book_id) FROM rental GROUP BY user_id;
SELECT user_id, MIN(book_id) FROM rental GROUP BY user_id;
```

---

# 2. HAVING — 그룹에 조건 적용

> `GROUP BY` 결과에 조건을 걸 때 사용 (`WHERE`는 그룹 이전, `HAVING`은 그룹 이후)

```sql
-- 2번 이상 대여한 user_id만 조회
SELECT user_id, COUNT(*)
FROM rental
GROUP BY user_id
HAVING COUNT(user_id) > 1;
```

> ⚠️ `WHERE`는 개별 행에 조건, `HAVING`은 그룹에 조건!

---

# 3. INNER JOIN — 두 테이블 연결

> 두 테이블에서 **조건이 일치하는 행만** 조회

```sql
-- 기본 문법
SELECT *
FROM rental
INNER JOIN user
ON user.id = rental.user_id;
```

## 실행 결과 예시

| rental_id | user_id | book_id | name | email |
|-----------|---------|---------|------|-------|
| 1 | 1 | 1000 | chanhwan | choich@elice.com |
| 2 | 1 | 1001 | chanhwan | choich@elice.com |
| 3 | 3 | 1004 | hyungon | gone@elice.com |

> `ON 테이블A.컬럼 = 테이블B.컬럼` 으로 연결 조건 지정

---

# 4. LEFT JOIN

> **왼쪽 테이블의 모든 행** + 오른쪽 테이블에서 일치하는 행 (없으면 NULL)

```sql
SELECT *
FROM user
LEFT JOIN rental
ON user.id = rental.user_id;
```

## 실행 결과 예시

| user_id | name | email | rental_id | book_id |
|---------|------|-------|-----------|---------|
| 1 | chanhwan | choich@elice.com | 1 | 1000 |
| 1 | chanhwan | choich@elice.com | 2 | 1001 |
| 2 | haesol | sunsol@elice.com | **null** | **null** |
| 3 | hyungon | gone@elice.com | 3 | 1004 |

> 대여 기록이 없는 haesol도 포함되고, rental 컬럼은 NULL로 표시

---

# 5. RIGHT JOIN

> **오른쪽 테이블의 모든 행** + 왼쪽 테이블에서 일치하는 행 (없으면 NULL)

```sql
SELECT *
FROM user
RIGHT JOIN rental
ON user.id = rental.user_id;
```

## 실행 결과 예시

| id | name | email | rental_id | book_id |
|----|------|-------|-----------|---------|
| 1 | chanhwan | choich@elice.com | 1 | 1000 |
| 1 | chanhwan | choich@elice.com | 2 | 1001 |
| **null** | **null** | **null** | 3 | 1004 |

> 탈퇴 회원(user_id=4)의 대여 기록도 포함되고, user 컬럼은 NULL로 표시

---

# JOIN 비교 정리

| JOIN 종류 | 결과 |
|-----------|------|
| `INNER JOIN` | 두 테이블에서 **겹치는 부분만** 출력 |
| `LEFT JOIN` | **왼쪽 테이블 전체** + 겹치는 부분 출력 |
| `RIGHT JOIN` | **오른쪽 테이블 전체** + 겹치는 부분 출력 |

```sql
-- INNER JOIN: 일치하는 것만
SELECT * FROM rental INNER JOIN user ON user.id = rental.user_id;

-- LEFT JOIN: user 전체 포함
SELECT * FROM user LEFT JOIN rental ON user.id = rental.user_id;

-- RIGHT JOIN: rental 전체 포함
SELECT * FROM user RIGHT JOIN rental ON user.id = rental.user_id;
```
