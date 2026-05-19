## 이 장에서 배우는 것

- 데이터베이스가 왜 필요한지, 언제 써야 하는지 이해한다
- 테이블, 컬럼, 행(row)의 개념을 설명할 수 있다
- 기본 키(Primary Key)와 외래 키(Foreign Key)가 무엇인지 안다
- 작은 서비스에서 자주 쓰는 테이블 설계 패턴을 적용할 수 있다
- SQLite로 간단한 테이블을 직접 만들고 데이터를 저장할 수 있다
- 1:N 관계(예: 사용자 - 주문)를 설계할 수 있다

---

## 먼저 쉬운 설명

메모장에 친구 전화번호를 적어 두면 처음에는 편합니다. 그런데 친구가 100명이 되면? 특정 친구를 찾으려면 처음부터 끝까지 다 읽어야 합니다. 게다가 같은 번호를 실수로 두 번 적거나, 오타를 내도 알 수가 없습니다.

**데이터베이스**는 이 문제를 해결해 줍니다. 데이터를 표(테이블) 형태로 정리하고, 빠르게 찾고, 중복 없이 저장할 수 있게 도와줍니다.

작은 서비스(개인 프로젝트, 사내 도구, 스타트업 MVP)를 만들 때 데이터베이스 설계를 처음부터 잘 해 두면, 나중에 기능을 추가할 때 코드를 뜯어고치는 고생을 줄일 수 있습니다.

이 장에서는 **SQLite**를 사용합니다. 설치가 필요 없고, 파일 하나가 곧 데이터베이스이기 때문에 배우기에 딱입니다.

---

## 1. 테이블이란 무엇인가

데이터베이스에서 데이터는 **테이블**이라는 2차원 표에 저장됩니다. 스프레드시트(엑셀)의 시트 하나와 비슷합니다.

| 개념 | 설명 | 스프레드시트 비유 |
|------|------|----------------|
| 테이블 (Table) | 같은 종류의 데이터 묶음 | 시트 한 장 |
| 컬럼 (Column) | 데이터의 속성 | 열 제목 (이름, 나이 등) |
| 행 (Row) | 실제 데이터 한 건 | 한 줄 |
| 기본 키 (Primary Key) | 행을 고유하게 구분하는 값 | 학번, 주민번호처럼 중복 없는 값 |

**Python 예제 — 테이블 만들기**

```python
# db_intro.py
import sqlite3

# 데이터베이스 파일 생성 (없으면 자동으로 만들어집니다)
conn = sqlite3.connect("my_service.db")
cursor = conn.cursor()

# 사용자 테이블 만들기
cursor.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id        INTEGER PRIMARY KEY AUTOINCREMENT,
        username  TEXT    NOT NULL UNIQUE,
        email     TEXT    NOT NULL UNIQUE,
        created_at TEXT   DEFAULT (datetime('now'))
    )
""")

conn.commit()
print("users 테이블이 만들어졌습니다.")
conn.close()
```

실행하면 `my_service.db` 파일이 생깁니다. 이 파일 하나가 곧 데이터베이스입니다.

> **포인트**: `PRIMARY KEY AUTOINCREMENT`를 쓰면 행이 추가될 때마다 id가 1, 2, 3 … 자동으로 늘어납니다. 직접 id 값을 신경 쓰지 않아도 됩니다.

---

## 2. 데이터 넣기, 읽기, 수정, 삭제 (CRUD)

데이터베이스의 4가지 기본 동작을 **CRUD**라고 부릅니다.

| 동작 | SQL 명령어 | 설명 |
|------|-----------|------|
| Create | `INSERT` | 새 데이터 추가 |
| Read | `SELECT` | 데이터 조회 |
| Update | `UPDATE` | 기존 데이터 수정 |
| Delete | `DELETE` | 데이터 삭제 |

**Python 예제 — CRUD 전체 흐름**

```python
# db_crud.py
import sqlite3

conn = sqlite3.connect("my_service.db")
cursor = conn.cursor()

# 1) CREATE — 사용자 추가
cursor.execute(
    "INSERT INTO users (username, email) VALUES (?, ?)",
    ("김철수", "chulsoo@example.com")
)
cursor.execute(
    "INSERT INTO users (username, email) VALUES (?, ?)",
    ("이영희", "younghee@example.com")
)
conn.commit()
print("사용자 2명 추가 완료")

# 2) READ — 전체 조회
cursor.execute("SELECT id, username, email FROM users")
rows = cursor.fetchall()
for row in rows:
    print(f"  id={row[0]}, 이름={row[1]}, 이메일={row[2]}")

# 3) UPDATE — 이메일 변경
cursor.execute(
    "UPDATE users SET email = ? WHERE username = ?",
    ("chulsoo_new@example.com", "김철수")
)
conn.commit()
print("이메일 수정 완료")

# 4) DELETE — 사용자 삭제
cursor.execute("DELETE FROM users WHERE username = ?", ("이영희",))
conn.commit()
print("이영희 삭제 완료")

conn.close()
```

> **중요**: SQL에 파이썬 변수를 넣을 때는 반드시 `?` 자리표시자(placeholder)를 사용하세요.  
> `f"... WHERE username = '{name}'"` 처럼 f-string으로 직접 넣으면 **SQL 인젝션** 보안 취약점이 생깁니다.

---

## 3. 기본 키와 외래 키 — 테이블 연결하기

실제 서비스에서는 테이블 하나로 모든 것을 담을 수 없습니다. 예를 들어 쇼핑몰을 만든다면 **사용자**와 **주문**은 별도 테이블로 분리하는 것이 맞습니다.

이때 두 테이블을 연결하는 것이 **외래 키(Foreign Key)**입니다.

```
users 테이블          orders 테이블
┌────────────┐        ┌──────────────────────┐
│ id (PK)    │◄───┐   │ id (PK)              │
│ username   │    └───│ user_id (FK)         │
│ email      │        │ product_name         │
└────────────┘        │ price                │
                      └──────────────────────┘
```

**Python 예제 — 1:N 관계 설계**

```python
# db_relations.py
import sqlite3

conn = sqlite3.connect("my_service.db")
cursor = conn.cursor()

# 주문 테이블 생성
cursor.execute("""
    CREATE TABLE IF NOT EXISTS orders (
        id           INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id      INTEGER NOT NULL,
        product_name TEXT    NOT NULL,
        price        INTEGER NOT NULL,
        ordered_at   TEXT    DEFAULT (datetime('now')),
        FOREIGN KEY (user_id) REFERENCES users(id)
    )
""")
conn.commit()

# 김철수(id=1)의 주문 추가
cursor.execute(
    "INSERT INTO orders (user_id, product_name, price) VALUES (?, ?, ?)",
    (1, "무선 키보드", 55000)
)
cursor.execute(
    "INSERT INTO orders (user_id, product_name, price) VALUES (?, ?, ?)",
    (1, "마우스패드", 12000)
)
conn.commit()

# 두 테이블 합쳐서 조회 (JOIN)
cursor.execute("""
    SELECT users.username, orders.product_name, orders.price
    FROM orders
    JOIN users ON orders.user_id = users.id
""")
for row in cursor.fetchall():
    print(f"{row[0]}님이 주문: {row[1]} ({row[2]:,}원)")

conn.close()
```

출력 예시:
```
김철수님이 주문: 무선 키보드 (55,000원)
김철수님이 주문: 마우스패드 (12,000원)
```

---

## 4. 작은 서비스를 위한 설계 원칙

### 원칙 1 — 한 테이블에 한 가지 주제만

나쁜 예:
```sql
-- 사용자와 주문 정보를 한 테이블에 섞으면 안 됩니다
CREATE TABLE user_orders (
    user_id       INTEGER,
    username      TEXT,
    email         TEXT,
    product_name  TEXT,   -- 주문마다 username, email이 반복 저장됨
    price         INTEGER
);
```

좋은 예: `users` 테이블과 `orders` 테이블을 분리하고 `user_id`로 연결합니다.

### 원칙 2 — NULL 허용 여부를 명확히

```python
# db_null_example.py
import sqlite3

conn = sqlite3.connect("my_service.db")
cursor = conn.cursor()

cursor.execute("""
    CREATE TABLE IF NOT EXISTS products (
        id          INTEGER PRIMARY KEY AUTOINCREMENT,
        name        TEXT    NOT NULL,          -- 반드시 있어야 하는 값
        description TEXT,                      -- 없어도 되는 값 (NULL 허용)
        price       INTEGER NOT NULL DEFAULT 0 -- 없으면 0으로 기본값 설정
    )
""")
conn.commit()
conn.close()
print("products 테이블 생성 완료")
```

### 원칙 3 — 자주 검색하는 컬럼에 인덱스 추가

```python
# db_index.py
import sqlite3

conn = sqlite3.connect("my_service.db")
cursor = conn.cursor()

# email로 자주 검색하므로 인덱스 추가
cursor.execute("""
    CREATE INDEX IF NOT EXISTS idx_users_email
    ON users(email)
""")
conn.commit()
print("인덱스 생성 완료 — email 검색이 빨라집니다")
conn.close()
```

> 인덱스는 책의 **색인(찾아보기)**과 같습니다. 색인이 있으면 원하는 단어를 처음부터 읽지 않고 바로 찾을 수 있습니다.

---

## 따라 하기 실습

### 실습 1 — 간단한 메모 앱 데이터베이스 만들기

`memo_app.py` 파일을 만들고 아래 코드를 작성하세요.

```python
# memo_app.py
import sqlite3

def get_connection():
    return sqlite3.connect("memo_app.db")

def setup_database():
    conn = get_connection()
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS memos (
            id         INTEGER PRIMARY KEY AUTOINCREMENT,
            title      TEXT NOT NULL,
            content    TEXT,
            created_at TEXT DEFAULT (datetime('now'))
        )
    """)
    conn.commit()
    conn.close()
    print("[완료] 데이터베이스 준비가 끝났습니다.")

if __name__ == "__main__":
    setup_database()
```

실행:
```bash
python memo_app.py
# [완료] 데이터베이스 준비가 끝났습니다.
```

### 실습 2 — 메모 추가 및 조회 기능 구현

실습 1의 `memo_app.py`에 아래 함수들을 추가하세요.

```python
# memo_app.py (계속)

def add_memo(title: str, content: str):
    conn = get_connection()
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO memos (title, content) VALUES (?, ?)",
        (title, content)
    )
    conn.commit()
    new_id = cursor.lastrowid  # 방금 추가된 행의 id
    conn.close()
    print(f"[추가] 메모 저장 완료 (id={new_id})")
    return new_id

def list_memos():
    conn = get_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT id, title, created_at FROM memos ORDER BY id DESC")
    rows = cursor.fetchall()
    conn.close()
    if not rows:
        print("저장된 메모가 없습니다.")
        return
    for row in rows:
        print(f"  [{row[0]}] {row[1]}  (작성: {row[2]})")

if __name__ == "__main__":
    setup_database()
    add_memo("오늘 할 일", "파이썬 공부, 운동 30분")
    add_memo("쇼핑 목록", "우유, 계란, 빵")
    print("\n--- 전체 메모 목록 ---")
    list_memos()
```

### 실습 3 — 태그 기능 추가 (1:N 관계 적용)

`memo_app.py`에 태그 테이블을 추가해서 하나의 메모에 여러 태그를 달 수 있게 합니다.

```python
# memo_app.py (태그 기능 추가)

def setup_database():
    conn = get_connection()
    cursor = conn.cursor()

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS memos (
            id         INTEGER PRIMARY KEY AUTOINCREMENT,
            title      TEXT NOT NULL,
            content    TEXT,
            created_at TEXT DEFAULT (datetime('now'))
        )
    """)

    # 태그 테이블: memo_id(FK) — memos(id)와 1:N 관계
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS tags (
            id      INTEGER PRIMARY KEY AUTOINCREMENT,
            memo_id INTEGER NOT NULL,
            name    TEXT    NOT NULL,
            FOREIGN KEY (memo_id) REFERENCES memos(id)
        )
    """)

    conn.commit()
    conn.close()
    print("[완료] 데이터베이스 준비가 끝났습니다.")

def add_tag(memo_id: int, tag_name: str):
    conn = get_connection()
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO tags (memo_id, name) VALUES (?, ?)",
        (memo_id, tag_name)
    )
    conn.commit()
    conn.close()
    print(f"[태그] 메모 {memo_id}에 '#{tag_name}' 추가됨")

def list_memos_with_tags():
    conn = get_connection()
    cursor = conn.cursor()
    cursor.execute("""
        SELECT memos.id, memos.title, GROUP_CONCAT(tags.name, ', ') AS tag_list
        FROM memos
        LEFT JOIN tags ON tags.memo_id = memos.id
        GROUP BY memos.id
        ORDER BY memos.id DESC
    """)
    for row in cursor.fetchall():
        tags = row[2] if row[2] else "태그 없음"
        print(f"  [{row[0]}] {row[1]}  태그: {tags}")
    conn.close()

if __name__ == "__main__":
    setup_database()
    memo_id = add_memo("파이썬 공부 계획", "DB 챕터부터 시작")
    add_tag(memo_id, "공부")
    add_tag(memo_id, "파이썬")
    print("\n--- 태그 포함 메모 목록 ---")
    list_memos_with_tags()
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-----------------|----------|
| 테이블 이름 오타 | `sqlite3.OperationalError: no such table: user` | `users`처럼 정확한 테이블 이름 확인. `CREATE TABLE IF NOT EXISTS` 코드가 먼저 실행됐는지 확인 |
| `conn.commit()` 누락 | 프로그램 종료 후 데이터가 사라짐 | INSERT/UPDATE/DELETE 후 반드시 `conn.commit()` 호출 |
| f-string으로 SQL 변수 삽입 | 보안 취약점 (에러 없이 작동하지만 위험) | `?` 플레이스홀더와 튜플 사용: `cursor.execute("... WHERE id = ?", (user_id,))` |
| 하나의 값도 튜플로 전달 안 함 | `sqlite3.ProgrammingError: Binding ... is not supported` | `(value,)` — 값이 하나여도 쉼표 필수: `(user_id,)` |
| 테이블을 지우고 다시 만들려고 함 | `sqlite3.OperationalError: table users already exists` | `CREATE TABLE IF NOT EXISTS` 사용하거나, 스키마 변경 시 `ALTER TABLE` 사용 |
| 외래 키 없는 user_id로 주문 삽입 | SQLite 기본 설정에서는 에러 없이 저장됨 | `cursor.execute("PRAGMA foreign_keys = ON")` 을 연결 직후 실행해 외래 키 강제 적용 |
| `fetchall()` 대신 `fetchone()` 혼동 | 결과가 하나만 나오거나 `None` 반환 | 여러 행이 올 수 있으면 `fetchall()`, 정확히 한 행만 기대하면 `fetchone()` |

---

## 확인 체크리스트

- [ ] `sqlite3.connect("파일명.db")`로 데이터베이스 파일을 만들 수 있다
- [ ] `CREATE TABLE`에서 `INTEGER PRIMARY KEY AUTOINCREMENT`의 역할을 설명할 수 있다
- [ ] `NOT NULL`과 `DEFAULT` 제약 조건이 무엇인지 안다
- [ ] `INSERT`, `SELECT`, `UPDATE`, `DELETE` 쿼리를 직접 작성할 수 있다
- [ ] `?` 플레이스홀더를 사용해 SQL 인젝션 없이 변수를 넣을 수 있다
- [ ] 외래 키로 두 테이블을 연결하고 `JOIN`으로 합쳐 조회할 수 있다
- [ ] `conn.commit()`과 `conn.close()`를 빠트리지 않는다
- [ ] 인덱스가 무엇이고 언제 추가하면 좋은지 말할 수 있다
- [ ] 메모 앱 실습 3개를 오류 없이 완료했다

---

## 한 번 더 생각해 보기

1. **분리 기준이 헷갈릴 때**: 사용자 프로필 사진 URL을 `users` 테이블에 컬럼으로 저장할지, 별도 `user_images` 테이블로 분리할지 어떻게 결정하면 좋을까요? 사용자가 사진을 여러 장 올릴 수 있다면 설계가 달라질까요?

2. **데이터가 삭제될 때**: `users` 테이블에서 사용자를 삭제하면 그 사람의 `orders` 데이터는 어떻게 해야 할까요? 같이 삭제해야 할까요, 아니면 남겨 두어야 할까요? 실제 쇼핑몰이라면 어떻게 하는 게 맞을까요?

3. **성능이 느려진다면**: 메모가 100만 개가 되었을 때 `SELECT * FROM memos WHERE title = '공부'`가 느려졌습니다. 지금 배운 내용에서 어떤 방법으로 이 문제를 해결할 수 있을까요?

---

## 다음 장

다음 장에서는 지금까지 만든 데이터베이스를 FastAPI와 연결해서 실제 HTTP API로 데이터를 주고받는 방법을 배웁니다.