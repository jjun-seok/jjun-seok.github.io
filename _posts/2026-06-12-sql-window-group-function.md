---
title: "SQL - 윈도우 함수 & 그룹 함수 완전 정리"
date: 2026-06-12 10:00:00 +0900
categories: [이어드림스쿨, SQL]
tags: [sql, window function, rank, groupby, rollup, cube]
---

# 데이터 분석을 위한 함수 3종류

| 종류 | 설명 |
|------|------|
| 윈도우 함수 | 행과 행 사이의 관계 정의 (순위, 누적 등) |
| 집계 함수 | SUM, AVG, COUNT, MAX, MIN |
| 그룹 함수 | ROLLUP, CUBE, GROUPING SETS |

---

# 1. 윈도우 함수 (Window Function)

> `OVER` 구문을 필수로 사용하는 함수

```sql
SELECT WINDOW_FUNCTION(ARGUMENTS)
OVER ([PARTITION BY 컬럼] [ORDER BY 절] [WINDOWING 절])
FROM 테이블명;
```

| 구조 | 설명 |
|------|------|
| `ARGUMENTS` | 함수에 따라 필요한 인수 |
| `PARTITION BY` | 소그룹으로 나누는 기준 |
| `ORDER BY` | 소그룹 내 정렬 기준 |
| `WINDOWING` | 행의 범위 기준 |

---

## 1-1. 순위 함수

| 함수 | 설명 |
|------|------|
| `RANK()` | 동일한 값에 동일한 순위 부여 (다음 순위 건너뜀) |
| `DENSE_RANK()` | 동일한 값에 동일한 순위 부여 (다음 순위 안 건너뜀) |
| `ROW_NUMBER()` | 동일한 값이라도 고유한 순위 부여 |

```sql
SELECT
    ID, NAME, SALARY,
    RANK()       OVER (ORDER BY SALARY DESC) AS RANK,
    DENSE_RANK() OVER (ORDER BY SALARY DESC) AS DENSE_RANK,
    ROW_NUMBER() OVER (ORDER BY SALARY DESC) AS ROW_NUMBER
FROM EMPLOYEE;
```

### 결과 예시

| NAME | SALARY | RANK | DENSE_RANK | ROW_NUMBER |
|------|--------|------|------------|------------|
| ELICE | 10000 | 1 | 1 | 1 |
| JAMES | 8000 | 2 | 2 | 2 |
| JESSICA | 6000 | 3 | 3 | 3 |
| JANNET | 6000 | 3 | 3 | 4 |
| STEVE | 4000 | 5 | 4 | 5 |
| ROBERT | 1500 | 6 | 5 | 6 |

> JESSICA, JANNET이 동점 → RANK는 5위 건너뜀, DENSE_RANK는 4위 이어감

---

## 1-2. 일반 집계 함수 + OVER

> `GROUP BY` 없이 집계 함수 사용 가능 → 원본 행 유지하면서 집계 가능

```sql
-- 부서별 평균 급여를 각 행에 함께 출력
SELECT
    ID, NAME, SALARY,
    AVG(SALARY) OVER (PARTITION BY DEPARTMENT_ID) AS DEPARTMENT_AVG
FROM EMPLOYEE;
```

| NAME | SALARY | DEPARTMENT_ID | DEPARTMENT_AVG |
|------|--------|---------------|----------------|
| JAMES | 8000 | 1 | 5875 |
| STEVE | 4000 | 1 | 5875 |
| ELICE | 10000 | 1 | 5875 |
| JANNET | 6000 | 2 | 6000 |

---

## 1-3. 행 순서 함수

| 함수 | 설명 |
|------|------|
| `FIRST_VALUE(컬럼)` | 가장 먼저 나온 값 |
| `LAST_VALUE(컬럼)` | 가장 나중에 나온 값 |
| `LAG(컬럼, N)` | 이전 N번째 행 값 |
| `LEAD(컬럼, N)` | 이후 N번째 행 값 |

```sql
-- 부서별 최소/최대 급여 함께 출력
SELECT
    ID, NAME, SALARY,
    FIRST_VALUE(SALARY) OVER(
        PARTITION BY DEPARTMENT_ID ORDER BY SALARY
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS DEPT_MIN_SALARY,
    LAST_VALUE(SALARY) OVER(
        PARTITION BY DEPARTMENT_ID ORDER BY SALARY
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS DEPT_MAX_SALARY
FROM EMPLOYEE;
```

```sql
-- 앞뒤 직원 이름 함께 출력
SELECT
    ID, NAME, SALARY,
    LAG(NAME, 1)  OVER (ORDER BY ID) AS PREV_EMPLOYEE,
    LEAD(NAME, 1) OVER (ORDER BY ID) AS NEXT_EMPLOYEE
FROM EMPLOYEE;
```

---

## 1-4. 비율 함수

| 함수 | 설명 |
|------|------|
| `PERCENT_RANK()` | 순위를 백분율로 (0~1) |
| `CUME_DIST()` | 누적 백분율 (0 초과 1 이하) |
| `NTILE(N)` | N등분한 그룹 번호 |

```sql
-- 급여 순위 백분율
SELECT
    ID, NAME, SALARY,
    PERCENT_RANK() OVER (ORDER BY SALARY DESC) AS PERCENT_RANK,
    ROUND(CUME_DIST() OVER (ORDER BY SALARY DESC), 4) AS CUME_DIST
FROM EMPLOYEE;

-- 3등분으로 나누기
SELECT
    ID, NAME, SALARY,
    NTILE(3) OVER (ORDER BY SALARY DESC) AS NTILE
FROM EMPLOYEE;
```

---

# 2. 그룹 함수 (Group Function)

> 소계/중계/합계 등 다양한 통계를 한 번에 계산

---

## 2-1. ROLLUP

> 그룹화 컬럼에 대한 **부분 소계 + 전체 합계** 자동 생성

```sql
-- Oracle/MySQL
SELECT D.NAME, J.NAME, AVG(E.SALARY)
FROM EMPLOYEE E
JOIN DEPARTMENT D ON E.DEPARTMENT_ID = D.ID
JOIN JOB J ON E.JOB_ID = J.ID
GROUP BY ROLLUP(D.NAME, J.NAME);

-- MariaDB
GROUP BY D.NAME, J.NAME WITH ROLLUP;
```

### 결과 예시

| DEPARTMENT | JOB | AVG_SALARY |
|------------|-----|------------|
| 개발 | 사원 | 1500 |
| 개발 | 대리 | 3000 |
| 개발 | **(null)** | **4083** ← 개발팀 소계 |
| 영업 | 사원 | 1800 |
| 영업 | **(null)** | **5060** ← 영업팀 소계 |
| **(null)** | **(null)** | **4527** ← 전체 합계 |

---

## 2-2. CUBE

> ROLLUP + **모든 컬럼 조합**의 통계까지 생성

```sql
-- Oracle/MySQL
GROUP BY CUBE(D.NAME, J.NAME);

-- MariaDB (ROLLUP 두 번 + UNION으로 구현)
GROUP BY D.NAME, J.NAME WITH ROLLUP
UNION
GROUP BY J.NAME, D.NAME WITH ROLLUP;
```

> ROLLUP과의 차이: 직군별 전체 통계 `(null, 과장, 3750)` 같은 행도 추가 생성

---

## 2-3. GROUPING SETS

> 명시된 컬럼에 대해 **각각 독립적인 통계** 생성

```sql
-- Oracle/MySQL
GROUP BY GROUPING SETS(D.NAME, J.NAME);

-- MariaDB (UNION ALL로 구현)
SELECT D.NAME, NULL, AVG(E.SALARY) FROM ... GROUP BY D.NAME
UNION ALL
SELECT NULL, J.NAME, AVG(E.SALARY) FROM ... GROUP BY J.NAME;
```

### 세 함수 비교

| 함수 | 생성하는 통계 |
|------|--------------|
| `ROLLUP(A, B)` | (A,B) + (A) + 전체 |
| `CUBE(A, B)` | (A,B) + (A) + (B) + 전체 |
| `GROUPING SETS(A, B)` | (A) + (B) 만 |

---

# 핵심 정리

```sql
-- 순위
RANK() OVER (ORDER BY 컬럼 DESC)
DENSE_RANK() OVER (ORDER BY 컬럼 DESC)
ROW_NUMBER() OVER (ORDER BY 컬럼 DESC)

-- 파티션별 집계
AVG(컬럼) OVER (PARTITION BY 그룹컬럼)

-- 앞뒤 행 접근
LAG(컬럼, 1) OVER (ORDER BY 컬럼)
LEAD(컬럼, 1) OVER (ORDER BY 컬럼)

-- 소계/합계
GROUP BY ROLLUP(A, B)    -- 부분 소계 + 전체
GROUP BY CUBE(A, B)      -- 모든 조합
GROUP BY GROUPING SETS(A, B)  -- 개별 통계
```
