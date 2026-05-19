## 이 장에서 배우는 것

- JSON 파일에서 데이터를 다시 읽는 방법
- `json.load()`와 `json.loads()`의 차이
- 파일이 없을 때 발생하는 오류를 처리하는 방법
- 이전 장에서 저장한 데이터를 불러와서 활용하는 방법

---

## 먼저 쉬운 설명

지난 장에서 우리는 사용자 데이터를 JSON 파일로 저장했어요.  
그런데 저장만 하면 의미가 없겠죠? 프로그램을 다시 실행했을 때 **"아, 이 사람이 전에 가입했었구나"** 라고 기억할 수 있어야 해요.

그게 바로 **다시 읽기(read back)** 예요.

마치 일기장에 오늘 있었던 일을 쓰고, 다음 날 다시 펼쳐보는 것과 같아요.  
파일에서 데이터를 읽어오면 프로그램이 "기억"을 갖게 됩니다.

---

## 1. json.load() — 파일에서 읽기

`json.load()`는 파일 객체를 받아서 파이썬 딕셔너리로 변환해줍니다.

```python
# read_user.py
import json

def load_user(filename):
    with open(filename, "r", encoding="utf-8") as f:
        user = json.load(f)  # 파일 → 파이썬 딕셔너리
    return user

user = load_user("user_data.json")
print(user["name"])   # 홍길동
print(user["email"])  # hong@example.com
```

> **핵심:** `json.load(f)`는 파일 객체 `f`를 받아요. 문자열이 아닙니다.

---

## 2. json.loads() — 문자열에서 읽기

`json.loads()`는 JSON **문자열**을 파이썬 딕셔너리로 변환해요.  
API 응답이나 변수에 저장된 JSON 문자열을 다룰 때 씁니다.

```python
# parse_json_string.py
import json

# 예: 외부 API에서 이런 문자열이 왔다고 가정
raw = '{"name": "김철수", "age": 25, "city": "서울"}'

user = json.loads(raw)  # 문자열 → 파이썬 딕셔너리
print(user["name"])   # 김철수
print(user["city"])   # 서울
```

> **헷갈리지 마세요:**
> - 파일 → `json.load(파일객체)`
> - 문자열 → `json.loads(문자열)` (`s`는 string의 s)

---

## 3. 이전 tiny service 코드와 연결하기

지난 장에서 `save_user()`를 만들었죠? 이제 `load_user()`를 붙여봅시다.

```python
# tiny_service.py
import json
import os

DATA_FILE = "user_data.json"

def save_user(name, email):
    user = {"name": name, "email": email}
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(user, f, ensure_ascii=False, indent=2)
    print(f"저장 완료: {name}")

def load_user():
    if not os.path.exists(DATA_FILE):
        print("저장된 사용자가 없습니다.")
        return None
    with open(DATA_FILE, "r", encoding="utf-8") as f:
        user = json.load(f)
    return user

# --- 실행 흐름 ---
save_user("박지수", "jisu@example.com")

user = load_user()
if user:
    print(f"불러온 사용자: {user['name']} / {user['email']}")
```

**실행 결과:**
```
저장 완료: 박지수
불러온 사용자: 박지수 / jisu@example.com
```

---

## 4. 파일이 없을 때 안전하게 처리하기

파일이 없는데 읽으려 하면 오류가 납니다. 두 가지 방법으로 막을 수 있어요.

**방법 1 — os.path.exists() 로 미리 확인:**

```python
# safe_load_v1.py
import json
import os

def load_user_safe(filename):
    if not os.path.exists(filename):
        return None  # 파일 없으면 None 반환
    with open(filename, "r", encoding="utf-8") as f:
        return json.load(f)
```

**방법 2 — try/except 로 오류 잡기:**

```python
# safe_load_v2.py
import json

def load_user_safe(filename):
    try:
        with open(filename, "r", encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"파일을 찾을 수 없어요: {filename}")
        return None
    except json.JSONDecodeError:
        print("JSON 형식이 올바르지 않아요.")
        return None
```

> 실제 서비스에서는 **방법 2(try/except)** 가 더 안전해요.  
> 파일이 있어도 내용이 깨져 있을 수 있거든요.

---

## 따라 하기 실습

### 실습 1 — 기본 읽기

`user_data.json` 파일을 직접 만들어 보세요.

```json
{
  "name": "이민준",
  "email": "minjun@example.com",
  "age": 30
}
```

그 다음 아래 코드를 `read_practice.py`로 저장하고 실행하세요.

```python
# read_practice.py
import json

with open("user_data.json", "r", encoding="utf-8") as f:
    user = json.load(f)

print("이름:", user["name"])
print("이메일:", user["email"])
print("나이:", user["age"])
```

---

### 실습 2 — 없는 파일 처리

이번엔 **없는 파일**을 읽어보고 오류를 직접 확인해요.

```python
# read_missing.py
import json

def load_or_default(filename):
    try:
        with open(filename, "r", encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        print(f"[알림] {filename} 파일이 없어서 기본값을 사용합니다.")
        return {"name": "게스트", "email": "guest@example.com"}

user = load_or_default("없는파일.json")
print("사용자:", user["name"])
```

기대 출력:
```
[알림] 없는파일.json 파일이 없어서 기본값을 사용합니다.
사용자: 게스트
```

---

### 실습 3 — 저장하고 바로 읽기 (전체 흐름)

`tiny_service_v4.py` 파일을 만들어 저장 → 읽기 전체 흐름을 완성하세요.

```python
# tiny_service_v4.py
import json
import os

DATA_FILE = "service_user.json"

def save_user(name, email, age):
    data = {"name": name, "email": email, "age": age}
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)

def load_user():
    try:
        with open(DATA_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        return None

# 1. 저장
save_user("최수연", "suyeon@example.com", 27)
print("저장했습니다.")

# 2. 읽기
user = load_user()
if user:
    print(f"안녕하세요, {user['name']}님! (나이: {user['age']})")
else:
    print("사용자 정보가 없습니다.")
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 해결 방법 |
|------|------------|-----------|
| 파일이 없는데 `open()` 호출 | `FileNotFoundError: [Errno 2] No such file or directory` | `os.path.exists()` 또는 `try/except` 사용 |
| `json.load(문자열)` 사용 | `AttributeError: 'str' object has no attribute 'read'` | 파일 객체가 필요함. 문자열이면 `json.loads()` 사용 |
| `json.loads(파일객체)` 사용 | `TypeError: the JSON object must be str, not TextIOWrapper` | 파일에서 읽으려면 `json.load(f)` 사용 |
| UTF-8 인코딩 미지정 | `UnicodeDecodeError` (한글 깨짐) | `open(..., encoding="utf-8")` 추가 |
| JSON 파일 내용이 깨진 경우 | `json.JSONDecodeError: Expecting value: line 1 column 1` | `json.JSONDecodeError` 예외 처리 추가 |

---

## 확인 체크리스트

- [ ] `json.load()`는 파일 객체를 받는다는 것을 안다
- [ ] `json.loads()`는 문자열을 받는다는 것을 안다
- [ ] `FileNotFoundError`를 `try/except`로 처리할 수 있다
- [ ] `open()` 에 `encoding="utf-8"` 을 붙이는 이유를 안다
- [ ] `save_user()` + `load_user()` 흐름을 순서대로 설명할 수 있다
- [ ] 실습 3의 `tiny_service_v4.py` 를 오류 없이 실행했다

---

## 한 번 더 생각해 보기

1. `json.load()`와 `json.loads()` 중 어느 것을 어떤 상황에서 써야 할까요? 각자 한 가지 예를 직접 생각해 보세요.

2. 파일이 없을 때 `None`을 반환하는 것과 기본값 딕셔너리를 반환하는 것 중 어느 방식이 더 좋을까요? 상황에 따라 달라질까요?

3. 사용자가 여러 명이라면 딕셔너리 하나 대신 어떤 자료구조를 써야 할까요?

---

## 다음 장

다음 장에서는 여러 사용자를 **리스트**로 관리하고 JSON 파일에 저장하는 방법을 배웁니다.