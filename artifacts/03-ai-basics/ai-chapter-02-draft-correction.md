# Chapter 02: AI 초안 수정하기

## 이 장에서 배우는 것

- AI 초안을 받았을 때 왜 수정해야 하는지
- 수정 전/후 코드 비교 3쌍
- 4가지 수정 패턴: 변수 이름 개선, 에러 처리 추가, 불필요한 코드 제거, 보안 개선
- AI에게 "이 코드를 개선해줘"라는 2단계 요청 방법
- 실습: 실제 AI가 생성한 문제 있는 코드를 직접 수정

---

## 왜 AI 초안을 수정해야 하는가

AI는 코드를 "그럴듯하게" 만들어낸다. 실행은 되고, 문법도 맞고, 구조도 있어 보인다. 그런데 그것이 좋은 코드인지는 다른 문제다.

AI가 생성한 초안을 그대로 쓰면 안 되는 이유가 있다.

**첫째, AI는 맥락을 모른다.** AI는 내 프로젝트 전체를 모른다. 변수 이름, 팀 코딩 스타일, 다른 코드와의 연결 방식을 알지 못한다.

**둘째, AI는 엣지케이스를 빠뜨린다.** 요청한 기능의 "정상적인 경우"만 처리하고, 예외 상황을 빠뜨리는 경우가 많다.

**셋째, AI는 보안을 기본으로 고려하지 않는다.** 예시 코드 수준에서는 보안을 생략하는 경우가 많다.

**넷째, AI는 단순화하지 않는다.** 같은 기능을 더 간단하게 쓸 수 있어도 복잡한 방식을 쓰는 경우가 있다.

수정하는 행위 자체가 공부다. AI 초안을 보고 "왜 이렇게 됐지?", "더 좋은 방법은 없을까?"라고 생각하는 과정에서 실력이 쌓인다.

---

## 1. 수정 패턴 1: 변수 이름 개선

### 수정 전 (AI 초안)

```python
def calc(a, b, c):
    x = a * b
    y = x + c
    z = y / 100
    return z
```

### 문제점

- `a`, `b`, `c`가 무엇인지 알 수 없다
- `x`, `y`, `z`가 무엇을 저장하는지 알 수 없다
- 함수 이름 `calc`가 무엇을 계산하는지 알 수 없다

### 수정 후

```python
def calculate_total_price_with_tax(quantity, unit_price, tax_amount):
    """
    수량, 단가, 세금을 받아서 세금 포함 총 금액을 반환한다.
    반환값은 원 단위(소수점 없음)다.
    """
    subtotal = quantity * unit_price
    total_with_tax = subtotal + tax_amount
    total_in_won = total_with_tax / 100  # 센트 단위를 원 단위로 변환
    return total_in_won
```

### 수정 원칙

- 인자 이름은 무엇을 받는지 말한다 (`a` → `quantity`)
- 변수 이름은 무엇을 담는지 말한다 (`x` → `subtotal`)
- 함수 이름은 무엇을 하는지 말한다 (`calc` → `calculate_total_price_with_tax`)
- 이름이 길어지는 것을 두려워하지 않는다

---

## 2. 수정 패턴 2: 에러 처리 추가

### 수정 전 (AI 초안)

```python
def read_user_data(user_id):
    with open(f"users/{user_id}.json", "r") as f:
        data = json.load(f)
    return data["name"], data["email"]
```

### 문제점

- 파일이 없으면 `FileNotFoundError`로 즉시 종료
- JSON 형식이 잘못됐으면 `json.JSONDecodeError`로 즉시 종료
- `data`에 `name`이나 `email`이 없으면 `KeyError`로 즉시 종료
- 어떤 user_id가 문제였는지 오류 메시지에 없다

### 수정 후

```python
import json

def read_user_data(user_id):
    """
    user_id에 해당하는 사용자 데이터를 읽어서 (이름, 이메일)을 반환한다.
    파일이 없거나 데이터에 문제가 있으면 (None, None)을 반환한다.
    """
    filepath = f"users/{user_id}.json"

    try:
        with open(filepath, "r", encoding="utf-8") as f:
            data = json.load(f)
    except FileNotFoundError:
        print(f"사용자 파일을 찾을 수 없습니다: {filepath}")
        return None, None
    except json.JSONDecodeError as e:
        print(f"JSON 형식이 올바르지 않습니다: {filepath} - {e}")
        return None, None

    name = data.get("name")      # get()은 키가 없으면 None 반환
    email = data.get("email")

    if not name or not email:
        print(f"사용자 {user_id}: name 또는 email 데이터가 없습니다.")
        return None, None

    return name, email

# 사용 예시
name, email = read_user_data(123)
if name is not None:
    print(f"이름: {name}, 이메일: {email}")
else:
    print("사용자 데이터를 불러오지 못했습니다.")
```

### 수정 원칙

- 파일, 네트워크, 사용자 입력을 다루는 코드는 반드시 `try/except`로 감싼다
- 오류 메시지에 무엇이 문제였는지 구체적으로 담는다
- 실패 시 반환값을 명확하게 정한다 (`None`, 빈 리스트, 기본값 등)

---

## 3. 수정 패턴 3: 불필요한 코드 제거 + 보안 개선

### 수정 전 (AI 초안)

```python
import sqlite3
import hashlib

def create_user(username, password):
    # 데이터베이스 연결
    conn = sqlite3.connect("users.db")
    cursor = conn.cursor()

    # 테이블 확인
    cursor.execute("SELECT name FROM sqlite_master WHERE type='table' AND name='users'")
    result = cursor.fetchone()
    if result is None:
        cursor.execute("CREATE TABLE users (id INTEGER PRIMARY KEY, username TEXT, password TEXT)")

    # 중복 확인
    cursor.execute("SELECT * FROM users WHERE username = '" + username + "'")  # SQL Injection 취약점
    existing = cursor.fetchone()
    if existing is not None:
        print("이미 있는 사용자입니다")
        conn.close()
        return False

    # 비밀번호 해시 (MD5 사용 - 보안 취약)
    hashed = hashlib.md5(password.encode()).hexdigest()

    # 사용자 추가
    cursor.execute("INSERT INTO users VALUES (NULL, '" + username + "', '" + hashed + "')")  # SQL Injection 취약점
    conn.commit()
    conn.close()
    return True
```

### 문제점

- SQL Injection 취약점: 사용자 입력을 직접 SQL 문자열에 붙임
- MD5 해시는 보안에 취약함 (현대 기준으로 사용하면 안 됨)
- 테이블 생성 코드가 create_user 함수 안에 있어서 매번 실행됨 (역할 혼재)
- `conn.close()`를 반환하기 전에 매번 직접 호출해야 해서 누락 위험이 있음

### 수정 후

```python
import sqlite3
import hashlib
import secrets

def initialize_db(conn):
    """데이터베이스 초기화: 필요한 테이블을 생성한다."""
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT UNIQUE NOT NULL,
            password_hash TEXT NOT NULL,
            salt TEXT NOT NULL
        )
    """)
    conn.commit()

def hash_password(password):
    """비밀번호를 솔트와 함께 안전하게 해시한다."""
    salt = secrets.token_hex(16)  # 랜덤 솔트 생성
    hashed = hashlib.sha256((password + salt).encode()).hexdigest()
    return hashed, salt

def create_user(conn, username, password):
    """
    새 사용자를 생성한다.
    성공하면 True, 이미 존재하면 False를 반환한다.
    """
    password_hash, salt = hash_password(password)

    try:
        cursor = conn.cursor()
        # 파라미터화된 쿼리로 SQL Injection 방지
        cursor.execute(
            "INSERT INTO users (username, password_hash, salt) VALUES (?, ?, ?)",
            (username, password_hash, salt)
        )
        conn.commit()
        return True
    except sqlite3.IntegrityError:
        # UNIQUE 제약 조건 위반 = 이미 있는 사용자
        return False

# 사용 예시
with sqlite3.connect("users.db") as conn:  # with 문으로 자동 닫기
    initialize_db(conn)
    if create_user(conn, "alice", "my_password"):
        print("사용자가 생성됐습니다.")
    else:
        print("이미 존재하는 사용자입니다.")
```

### 수정 원칙

- SQL에 사용자 입력을 직접 붙이지 않는다 (항상 `?` 플레이스홀더 사용)
- MD5 대신 SHA-256 이상 + 솔트를 쓴다
- `with` 문으로 파일/연결을 자동으로 닫는다
- 하나의 함수는 하나의 역할만 한다

---

## 4. AI에게 코드 개선을 요청하는 2단계 방법

AI 초안을 직접 수정하는 것 외에, AI에게 스스로 수정을 요청할 수도 있다. 단, 이때도 결과를 검토하는 역할은 내가 한다.

### 1단계: 현재 코드의 문제점을 먼저 파악하게 요청한다

무작정 "개선해줘"라고 하면 AI는 표면적인 변경만 한다. 먼저 분석을 시키면 더 구체적인 수정이 나온다.

```
이 코드의 문제점을 3가지 이상 찾아줘.
각 문제점에 대해 왜 문제인지 설명해줘.

[코드 붙여넣기]
```

AI가 분석 결과를 주면, 나도 같은 부분을 직접 확인해 본다. AI가 못 찾은 문제가 있을 수 있고, AI가 찾은 문제가 실제로는 문제가 아닐 수도 있다.

### 2단계: 구체적인 개선 방향을 지정해서 수정을 요청한다

```
위에서 찾은 문제점을 바탕으로 코드를 수정해줘.
수정할 때 다음을 지켜줘:
1. 변수 이름은 하는 일이 명확하게 드러나도록 바꿔줘
2. 파일을 열 때 try/except를 추가해줘
3. SQL 쿼리는 파라미터화된 방식으로 바꿔줘

원본 코드와 수정된 코드를 나란히 보여주고,
무엇을 왜 바꿨는지 각 항목별로 설명해줘.
```

이 방법의 장점:
- 수정 이유를 AI가 설명하기 때문에 배울 수 있다
- 어떤 부분을 바꿀지 내가 지정하기 때문에 의도와 다른 수정을 막는다
- 수정 전/후 비교가 있어서 검토하기 쉽다

---

## 실습: 문제 있는 코드를 직접 수정하기

### 따라 하기

다음은 AI가 생성한 "학생 관리 프로그램" 코드다. 여러 문제가 있다. 먼저 직접 읽고 문제를 찾아보자.

```python
# AI가 생성한 코드 (수정 전)

def manage_students(s, op, d=None):
    if op == "add":
        s.append(d)
        print("추가됨")
    elif op == "get_avg":
        t = 0
        for i in s:
            t = i["s"]
        a = t / len(s)
        print("평균:", a)
    elif op == "search":
        for i in s:
            if i["n"] == d:
                print(i)
                return
        print("없음")
    elif op == "delete":
        for i in range(len(s)):
            if s[i]["n"] == d:
                s.remove(s[i])
                print("삭제됨")
                return
```

#### 문제 찾기

이 코드에는 최소 5가지 문제가 있다. 읽으면서 어떤 문제가 있는지 목록을 만들어 보자.

```
발견된 문제:
1. ?
2. ?
3. ?
4. ?
5. ?
```

아래는 정답 목록이다. 먼저 직접 찾아보고 비교한다.

```
정답:
1. 변수 이름: s, op, d, t, a, i 등이 의미를 전달하지 못함
2. 논리 오류: t = i["s"] 에서 += 대신 = 를 써서 누적이 안 됨
3. 엣지케이스: 빈 리스트에서 평균을 구하면 ZeroDivisionError 발생
4. 키 불일치: "s" 와 "n" 이 점수(score)와 이름(name)의 약어인데, 불명확함
5. 하나의 함수가 추가/평균/검색/삭제를 모두 처리함 (단일 책임 원칙 위반)
```

### 수정하기

발견한 문제를 하나씩 수정해 보자.

```python
# 수정된 코드

def add_student(students, student_data):
    """학생 목록에 새 학생을 추가한다."""
    students.append(student_data)
    print(f"학생 추가 완료: {student_data['name']}")

def calculate_average_score(students):
    """학생들의 평균 점수를 계산한다. 학생이 없으면 None을 반환한다."""
    if not students:
        print("학생 데이터가 없습니다.")
        return None

    total_score = 0
    for student in students:
        total_score += student["score"]  # 수정: = 에서 += 로 변경

    average = total_score / len(students)
    print(f"평균 점수: {average:.1f}")
    return average

def search_student(students, name):
    """이름으로 학생을 검색한다. 없으면 None을 반환한다."""
    for student in students:
        if student["name"] == name:
            return student
    print(f"'{name}' 학생을 찾을 수 없습니다.")
    return None

def delete_student(students, name):
    """이름으로 학생을 삭제한다. 성공하면 True, 없으면 False를 반환한다."""
    for i, student in enumerate(students):
        if student["name"] == name:
            students.pop(i)
            print(f"'{name}' 학생을 삭제했습니다.")
            return True
    print(f"'{name}' 학생을 찾을 수 없습니다.")
    return False

# 사용 예시
students = []

add_student(students, {"name": "김철수", "score": 85})
add_student(students, {"name": "이영희", "score": 92})
add_student(students, {"name": "박민준", "score": 78})

calculate_average_score(students)

found = search_student(students, "이영희")
if found:
    print(f"검색 결과: {found}")

delete_student(students, "박민준")
print(f"현재 학생 수: {len(students)}")

# 엣지케이스 테스트
calculate_average_score([])  # 빈 리스트
search_student(students, "없는학생")
delete_student(students, "없는학생")
```

### 직접 해보기

1. 위의 수정된 코드를 직접 타이핑해서 실행해 보자 (복사 금지)
2. 수정 전 코드에서 `t = i["s"]` 부분을 그대로 두고 실행하면 어떤 결과가 나오는지 확인해 보자 (버그 확인)
3. `delete_student` 함수를 `enumerate()` 없이 다른 방법으로 구현해 보자

---

## 수정 요약: 4가지 패턴 정리

| 수정 패턴 | 수정 전 | 수정 후 | 효과 |
|-----------|---------|---------|------|
| 변수 이름 개선 | `a`, `x`, `calc` | `average`, `subtotal`, `calculate_price` | 코드를 읽는 사람이 이해하기 쉬워짐 |
| 에러 처리 추가 | 파일 열기, JSON 파싱에 try/except 없음 | `try/except` + 구체적인 메시지 + 안전한 반환값 | 오류 발생 시 프로그램이 친절하게 대응 |
| 불필요한 코드 제거 | 한 함수에서 DB 초기화 + 사용자 추가 | 역할별로 함수 분리 | 읽기 쉽고 테스트하기 쉬워짐 |
| 보안 개선 | SQL 문자열 직접 결합, MD5 해시 | 파라미터화된 쿼리, SHA-256 + 솔트 | SQL Injection, 비밀번호 탈취 방지 |

---

## 초보자가 자주 막히는 지점

### 막힘 1: "실행이 되면 됐다"는 생각

코드가 오류 없이 실행되는 것과 올바르게 동작하는 것은 다르다.

```python
# 실행은 되지만 결과가 틀린 코드
total = 0
for score in [85, 92, 78]:
    total = score   # 마지막 값인 78만 남는다
print(total / 3)   # 26.0 (기대: 85.0)
```

테스트는 항상 "내가 예상한 결과와 일치하는가"로 확인해야 한다.

### 막힘 2: AI에게 "개선해줘"라고만 요청하는 것

```
나쁜 요청: "이 코드 개선해줘"

좋은 요청:
"이 코드에서 다음 3가지를 개선해줘:
1. 변수 이름을 의미 있게 바꿔줘
2. 빈 리스트 엣지케이스를 처리해줘
3. try/except로 에러 처리를 추가해줘
각 항목별로 왜 바꿨는지 설명해줘."
```

### 막힘 3: 수정 이유를 설명하지 못하는 것

코드를 바꿨다면, 왜 바꿨는지 한 문장으로 설명할 수 있어야 한다. 설명하지 못하면 아직 이해하지 못한 것이다.

```
나쁜 대답: "AI가 이렇게 하라고 해서요"

좋은 대답: "total = score는 매번 덮어쓰기 때문에 마지막 값만 남습니다.
점수를 누적하려면 total += score로 바꿔야 합니다."
```

---

## 확인 체크리스트

- AI 초안을 실행하기 전에 먼저 읽었는가
- 변수 이름이 하는 일을 표현하고 있는가
- 예외 상황(빈 리스트, 없는 파일 등)이 처리되어 있는가
- 사용자 입력을 SQL이나 시스템 명령에 직접 넣고 있지 않은가
- 하나의 함수가 하나의 역할만 하고 있는가
- 수정한 내용을 이유와 함께 설명할 수 있는가

---

## 한 번 더 생각해 보기

1. 변수 이름을 잘 짓는 것이 왜 중요할까? 이름이 나쁘면 어떤 문제가 생길까?
2. 에러 처리를 추가하면 코드가 길어진다. 그래도 추가해야 할까?
3. AI에게 코드 개선을 요청할 때 구체적으로 지정하는 이유는 무엇일까?
4. 수정 이유를 설명하는 연습이 실력 향상에 왜 도움이 될까?
