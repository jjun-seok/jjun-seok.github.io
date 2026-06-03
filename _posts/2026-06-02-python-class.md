---
title: "Python 클래스(Class) - OOP 기초 완전 정리"
date: 2026-06-02 09:00:00 +0900
categories: [이어드림스쿨, Python]
tags: [python, class, oop, 상속, 다형성, 인스턴스]
---

# 클래스(Class)란?

> **객체를 만들기 위한 설계도(청사진)**

- **속성(Attribute)**: 이름, 직책, 연봉 등 → 객체가 가지는 '데이터'
- **메서드(Method)**: 자기소개, 업무수행 등 → 객체가 할 수 있는 '행동'
- **클래스** = 설계도 / **인스턴스(객체)** = 설계도로 만들어진 실제 물건

---

# 1. 기본 클래스 만들기

```python
class TeamMember:
    pass   # 빈 클래스

member1 = TeamMember()
member2 = TeamMember()

print(type(member1))          # <class '__main__.TeamMember'>
print(member1 is member2)     # False (서로 다른 객체)
```

---

# 2. __init__ 메서드 (생성자)

- 인스턴스가 생성될 때 **자동으로 실행**되는 특별한 메서드
- `self` = '나 자신(인스턴스)'을 가리키는 매개변수
- 인스턴스의 초기 속성(데이터)을 설정할 때 사용

```python
class TeamMember:
    def __init__(self, name, role, salary):
        self.name = name
        self.role = role
        self.salary = salary

alice = TeamMember(name="Alice", role="시니어 개발자", salary=6000)
bob   = TeamMember(name="Bob",   role="주니어 기획자", salary=3800)

print(f"{alice.name}의 직책: {alice.role}, 연봉: {alice.salary}만원")
# Alice의 직책: 시니어 개발자, 연봉: 6000만원
```

---

# 3. 인스턴스 메서드 추가

- 클래스 내부에 정의된 함수
- 첫 번째 매개변수는 항상 `self`

```python
class TeamMember:
    def __init__(self, name, role, salary):
        self.name = name
        self.role = role
        self.salary = salary
        self.projects = []   # 빈 리스트로 초기화

    def introduce(self):
        print(f"안녕하세요! 저는 {self.role} {self.name}입니다.")

    def join_project(self, project_name):
        self.projects.append(project_name)
        print(f"{self.name}이(가) '{project_name}' 프로젝트에 합류했습니다.")

    def show_projects(self):
        if self.projects:
            print(f"{self.name}의 현재 프로젝트: {', '.join(self.projects)}")
        else:
            print(f"{self.name}은(는) 현재 참여 중인 프로젝트가 없습니다.")

    def get_raise(self, amount):
        self.salary += amount
        print(f"{self.name}의 연봉이 {amount}만원 인상되어 {self.salary}만원이 되었습니다.")

# 실습
alice = TeamMember("Alice", "시니어 개발자", 6000)
alice.introduce()
alice.join_project("AI 챗봇 개발")
alice.join_project("백엔드 리팩토링")
alice.show_projects()
alice.get_raise(500)
```

---

# 4. 클래스 변수 vs 인스턴스 변수

| 구분 | 설명 | 선언 위치 |
|------|------|-----------|
| 클래스 변수 | 모든 인스턴스가 **공유** | 클래스 내부, `__init__` 바깥 |
| 인스턴스 변수 | 각 인스턴스마다 **독립적** | `__init__` 내부, `self.변수명` |

```python
class TeamMember:
    company_name = "TechCorp"   # 클래스 변수: 모두 공유
    total_members = 0           # 클래스 변수: 팀원 수 카운터

    def __init__(self, name, role, salary):
        self.name = name        # 인스턴스 변수
        self.role = role
        self.salary = salary
        TeamMember.total_members += 1   # 생성될 때마다 +1

alice = TeamMember("Alice", "시니어 개발자", 6000)
bob   = TeamMember("Bob",   "주니어 기획자", 3800)

print(f"총 팀원 수: {TeamMember.total_members}명")   # 2명

# 클래스 변수 변경 → 모든 인스턴스에 반영
TeamMember.company_name = "TechCorp Korea"
```

---

# 5. __str__과 __repr__

- `__str__`: `print()` 호출 시 **사람이 읽기 좋은** 형태로 출력
- `__repr__`: 디버깅용 **공식적인** 표현 반환

```python
class TeamMember:
    def __init__(self, name, role, salary):
        self.name = name
        self.role = role
        self.salary = salary

    def __str__(self):
        return f"{self.role} {self.name}"

    def __repr__(self):
        return f"TeamMember(name='{self.name}', role='{self.role}', salary={self.salary})"

alice = TeamMember("Alice", "시니어 개발자", 6000)
print(str(alice))    # 시니어 개발자 Alice
print(repr(alice))   # TeamMember(name='Alice', role='시니어 개발자', salary=6000)
print(alice)         # 시니어 개발자 Alice
```

---

# 6. 클래스 메서드 & 정적 메서드

| 종류 | 데코레이터 | 첫 번째 매개변수 | 용도 |
|------|-----------|----------------|------|
| 인스턴스 메서드 | 없음 | `self` | 인스턴스 데이터 사용 |
| 클래스 메서드 | `@classmethod` | `cls` | 클래스 변수 사용, 대체 생성자 |
| 정적 메서드 | `@staticmethod` | 없음 | 클래스와 관련은 있지만 독립적인 기능 |

```python
class TeamMember:
    company_name = "TechCorp"
    total_members = 0

    def __init__(self, name, role, salary):
        self.name = name
        self.role = role
        self.salary = salary
        TeamMember.total_members += 1

    @classmethod
    def get_total_members(cls):
        return cls.total_members

    @classmethod
    def from_string(cls, member_string):
        """'이름-직책-연봉' 형식의 문자열로 팀원 생성"""
        name, role, salary = member_string.split("-")
        return cls(name, role, int(salary))

    @staticmethod
    def is_valid_salary(salary):
        """연봉 유효성 검사 (1000~20만)"""
        return 1000 <= salary <= 200000

# 클래스 메서드 사용
carol = TeamMember.from_string("Carol-데이터 분석가-4500")
print(f"총 팀원 수: {TeamMember.get_total_members()}명")

# 정적 메서드 사용
print(TeamMember.is_valid_salary(6000))   # True
print(TeamMember.is_valid_salary(500))    # False
```

---

# 7. 상속(Inheritance)

> 기존 클래스의 속성과 메서드를 물려받아 새로운 클래스를 만드는 것

```
Department (부서 기본 클래스)
    ├── DevTeam       (개발팀)
    ├── PlanningTeam  (기획팀)
    ├── SalesTeam     (영업팀)
    └── AnalyticsTeam (분석팀)
```

```python
# 부모 클래스
class Department:
    def __init__(self, team_name, budget):
        self.team_name = team_name
        self.budget = budget
        self.members = []

    def add_member(self, member):
        self.members.append(member)
        print(f"[{self.team_name}] {member.name}님이 합류했습니다.")

    def team_meeting(self):
        print(f"[{self.team_name}] 정기 팀 회의를 시작합니다.")

# 자식 클래스
class DevTeam(Department):
    def __init__(self, budget, tech_stack):
        super().__init__("개발팀", budget)   # 부모 __init__ 호출
        self.tech_stack = tech_stack

    def team_meeting(self):   # 오버라이딩
        print(f"[{self.team_name}] 스프린트 계획 회의 (기술 스택: {', '.join(self.tech_stack)})")

class SalesTeam(Department):
    def __init__(self, budget, monthly_target):
        super().__init__("영업팀", budget)
        self.monthly_target = monthly_target
        self.monthly_achievement = 0

    def close_deal(self, amount):
        self.monthly_achievement += amount
        rate = (self.monthly_achievement / self.monthly_target) * 100
        print(f"[{self.team_name}] 계약 성사! +{amount}만원 (달성률: {rate:.1f}%)")

    def team_meeting(self):   # 오버라이딩
        rate = (self.monthly_achievement / self.monthly_target) * 100
        print(f"[{self.team_name}] 영업 현황 회의 (목표: {self.monthly_target}만원, 달성률: {rate:.1f}%)")
```

---

# 8. 다형성(Polymorphism)

> 같은 메서드 호출, 팀마다 **다르게 동작**

```python
all_teams = [dev_team, planning_team, sales_team, analytics_team]

for team in all_teams:
    team.team_meeting()   # 각 팀의 오버라이딩된 메서드가 호출됨
```

```
[개발팀] 스프린트 계획 회의를 시작합니다.
[기획팀] 기획 검토 회의를 시작합니다.
[영업팀] 영업 현황 회의 (목표: 100000만원, 달성: 75000만원, 75.0%)
[분석팀] 데이터 리뷰 회의
```

---

# 9. isinstance() & issubclass()

```python
# isinstance(객체, 클래스): 인스턴스 여부 확인
print(isinstance(dev_team, DevTeam))       # True
print(isinstance(dev_team, Department))    # True (상속 관계도 True)
print(isinstance(alice, Department))       # False

# issubclass(자식, 부모): 상속 관계 확인
print(issubclass(DevTeam, Department))     # True
print(issubclass(SalesTeam, DevTeam))      # False
```

---

# 핵심 정리

| 개념 | 설명 | 예시 |
|------|------|------|
| 클래스 | 객체의 설계도 | `class TeamMember:` |
| 인스턴스 | 클래스로 만든 실제 객체 | `alice = TeamMember(...)` |
| `__init__` | 인스턴스 생성 시 자동 실행 | 초기 속성 설정 |
| 인스턴스 변수 | 각 인스턴스마다 독립적인 데이터 | `self.name` |
| 클래스 변수 | 모든 인스턴스가 공유하는 데이터 | `total_members = 0` |
| 인스턴스 메서드 | `self`를 받는 일반 메서드 | `def introduce(self):` |
| 클래스 메서드 | `@classmethod`, `cls`를 받음 | 대체 생성자 등 |
| 정적 메서드 | `@staticmethod`, 독립적 기능 | 유틸리티 함수 |
| 상속 | 부모 클래스를 물려받아 확장 | `class DevTeam(Department):` |
| 오버라이딩 | 부모 메서드를 자식에서 재정의 | `team_meeting()` |
| 다형성 | 같은 메서드 호출, 다른 동작 | `for team in all_teams: team.team_meeting()` |
