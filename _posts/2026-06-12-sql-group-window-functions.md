---
title: "SQL로 데이터 다루기 2 - 그룹 함수 & 윈도우 함수"
date: 2025-06-12
categories: [SQL, Database]
tags: [sql, window-function, group-function, mariadb, mysql]
---

## 📌 데이터 분석을 위한 함수 3종

| 분류 | 설명 |
|------|------|
| 윈도우 함수 (Window Function) | 행과 행 사이의 관계 정의 (순위, 집계 등) |
| 집계 함수 (Aggregate Function) | SUM, AVG, COUNT 등 |
| 그룹 함수 (Group Function) | ROLLUP, CUBE, GROUPING SETS |

---

## 02. 윈도우 함수 (Window Function)

> 순위, 집계 등 **행과 행 사이의 관계**를 정의하는 함수.  
> `OVER` 구문을 **필수**로 사용한다.

**지원 버전**
- MySQL 8.0.22 이상
- MariaDB 10.2.0 이상

### 기본 문법

```sql
SELECT WINDOW_FUNCTION(ARGUMENTS)
OVER (
  [PARTITION BY 칼럼]
  [ORDER BY 절]
  [WINDOWING 절]
)
FROM 테이블명;
```

### OVER 절 구조

| 구조 | 설명 |
|------|------|
| ARGUMENTS | 윈도우 함수에 따라 필요한 인수 |
| PARTITION BY | 전체 집합을 소그룹으로 나누는 기준 |
| ORDER BY | 소그룹에 대한 정렬 기준 |
| WINDOWING | 행에 대한 범위 기준 |

### WINDOWING 절 옵션

| 구조 | 설명 |
|------|------|
| ROWS | 물리적 단위로 행의 집합 지정 |
| UNBOUNDED PRECEDING | 윈도우 시작 위치 = 첫 번째 행 |
| UNBOUNDED FOLLOWING | 윈도우 마지막 위치 = 마지막 행 |
| CURRENT ROW | 윈도우 시작 위치 = 현재 행 |

---

### 🏆 순위 함수

| 함수 | 설명 |
|------|------|
| `RANK()` | 동일한 값에 동일한 순위 부여 (다음 순위는 건너뜀) |
| `DENSE_RANK()` | 동일한 값에 같은 순위, 한 건으로 취급 (순위 안 건너뜀) |
| `ROW_NUMBER()` | 동일한 값이라도 고유한 순위 부여 |

```sql
SELECT
    ID, NAME, SALARY,
    RANK()       OVER (ORDER BY SALARY DESC) AS RANK,
    DENSE_RANK() OVER (ORDER BY SALARY DESC) AS DENSE_RANK,
    ROW_NUMBER() OVER (ORDER BY SALARY DESC) AS ROW_NUMBER
FROM EMPLOYEE;
```

**결과 예시**

| NAME | SALARY | RANK | DENSE_RANK | ROW_NUMBER |
|------|--------|------|------------|------------|
| ELICE | 10000 | 1 | 1 | 1 |
| JAMES | 8000 | 2 | 2 | 2 |
| JESSICA | 6000 | 3 | 3 | 3 |
| JANNET | 6000 | 3 | 3 | 4 |
| STEVE | 4000 | 5 | 4 | 5 |
| ROBERT | 1500 | 6 | 5 | 6 |

> 💡 JESSICA, JANNET이 동점일 때 RANK는 5를 건너뛰지만, DENSE_RANK는 바로 4로 간다.

---

### 📊 일반 집계 함수 (윈도우로 사용)

`GROUP BY` 없이 `SUM`, `AVG`, `MAX`, `MIN` 등을 행 단위로 함께 출력할 수 있다.

```sql
-- 서브쿼리 방식 (복잡)
SELECT ID, NAME, SALARY, DEPARTMENT_ID,
    (SELECT AVG(SALARY) FROM EMPLOYEE B
     WHERE B.DEPARTMENT_ID = A.DEPARTMENT_ID) AS DEPARTMENT_AVG
FROM EMPLOYEE A;

-- 윈도우 함수 방식 (간결)
SELECT ID, NAME, SALARY,
    AVG(SALARY) OVER (PARTITION BY DEPARTMENT_ID) AS DEPARTMENT_AVG
FROM EMPLOYEE;
```

---

### 🔀 그룹 내 행 순서 함수

| 함수 | 설명 |
|------|------|
| `FIRST_VALUE(col)` | 파티션 내 가장 먼저 나온 값 |
| `LAST_VALUE(col)` | 파티션 내 가장 나중에 나온 값 |
| `LAG(col, n)` | 이전 n번째 행의 값 |
| `LEAD(col, n)` | 이후 n번째 행의 값 |

#### FIRST_VALUE / LAST_VALUE

```sql
-- 부서별 최소/최대 급여를 각 행에 함께 출력
SELECT ID, DEPARTMENT_ID, NAME, SALARY,
    FIRST_VALUE(SALARY) OVER (
        PARTITION BY DEPARTMENT_ID ORDER BY SALARY
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS DEPARTMENT_MIN_SALARY,
    LAST_VALUE(SALARY) OVER (
        PARTITION BY DEPARTMENT_ID ORDER BY SALARY
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS DEPARTMENT_MAX_SALARY
FROM EMPLOYEE
ORDER BY ID;
```

> ⚠️ `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` 없이 쓰면
> `LAST_VALUE`가 현재 행까지만 보므로 반드시 명시할 것!

#### LAG / LEAD

```sql
-- 앞/뒤 직원 이름 함께 출력
SELECT ID, NAME, SALARY,
    LAG(NAME, 1)  OVER (ORDER BY ID) AS PREV_EMPLOYEE,
    LEAD(NAME, 1) OVER (ORDER BY ID) AS NEXT_EMPLOYEE
FROM EMPLOYEE;
```

---

### 📈 그룹 내 비율 함수

| 함수 | 설명 |
|------|------|
| `RATIO_TO_REPORT(col)` | 전체 SUM 대비 각 행의 비율 |
| `PERCENT_RANK()` | 순위를 백분율로 표현 (0~1, 최고=0) |
| `CUME_DIST()` | 현재 행 이하 값들의 누적 백분율 (0초과~1) |
| `NTILE(n)` | 파티션 내 행들을 n등분 |

```sql
-- RATIO_TO_REPORT (MySQL 지원, MariaDB는 수동 구현)
SELECT ID, NAME, SALARY,
    SUM(SALARY) OVER() AS TOTAL_SALARY,
    RATIO_TO_REPORT(SALARY) OVER() AS RATIO
FROM EMPLOYEE;

-- MariaDB 대체 방법
SELECT ID, NAME, SALARY,
    SUM(SALARY) OVER() AS TOTAL_SALARY,
    (SALARY / SUM(SALARY) OVER()) AS RATIO
FROM EMPLOYEE;
```

```sql
-- PERCENT_RANK, CUME_DIST
SELECT ID, NAME, SALARY,
    PERCENT_RANK() OVER (ORDER BY SALARY DESC) AS PERCENT_RANK,
    ROUND(CUME_DIST() OVER (ORDER BY SALARY DESC), 4) AS CUME_DIST
FROM EMPLOYEE;
```

```sql
-- NTILE: 급여 기준 3등분
SELECT ID, NAME, SALARY,
    NTILE(3) OVER (ORDER BY SALARY DESC) AS NTILE
FROM EMPLOYEE;
```

---

## 03. 그룹 함수 (Group Function)

> 소계, 중계, 전체 합계 등 **다차원 집계**를 한 번의 쿼리로 처리한다.
> Oracle 중심 개념이지만 MySQL/MariaDB에서도 유사하게 사용 가능.

---

### ROLLUP

**그룹화 컬럼에 대한 소계 + 전체 합계** 를 자동으로 추가한다.

```sql
-- Oracle / MySQL
GROUP BY ROLLUP(D.NAME, J.NAME);

-- MariaDB
GROUP BY D.NAME, J.NAME WITH ROLLUP;
```

결과: 각 부서별 직책 평균 → 부서 소계(JOB_NAME = null) → 전체 합계(둘 다 null)

---

### CUBE

ROLLUP의 결과 + **결합 가능한 모든 경우의 수**에 대한 다차원 집계를 생성한다.

```sql
-- Oracle / MySQL
GROUP BY CUBE(D.NAME, J.NAME);

-- MariaDB (CUBE 미지원 → ROLLUP 2번 UNION)
GROUP BY D.NAME, J.NAME WITH ROLLUP
UNION
GROUP BY J.NAME, D.NAME WITH ROLLUP;
```

---

### GROUPING SETS

명시한 **각 컬럼에 대해 독립적인 통계**만 생성한다.  
`GROUP BY D.NAME` + `GROUP BY J.NAME` 을 `UNION ALL` 한 것과 동일.

```sql
-- Oracle / MySQL
GROUP BY GROUPING SETS(D.NAME, J.NAME);

-- MariaDB (수동 UNION ALL)
SELECT D.NAME, NULL, AVG(E.SALARY) FROM ... GROUP BY D.NAME
UNION ALL
SELECT NULL, J.NAME, AVG(E.SALARY) FROM ... GROUP BY J.NAME;
```

---

### 📌 그룹 함수 비교 요약

| 함수 | 생성하는 집계 |
|------|-------------|
| `ROLLUP(A, B)` | (A,B) → (A) → () |
| `CUBE(A, B)` | (A,B) → (A) → (B) → () |
| `GROUPING SETS(A, B)` | (A) → (B) |

> CUBE는 컬럼 수가 많아질수록 경우의 수가 기하급수적으로 늘어나니 주의!
