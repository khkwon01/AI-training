# Chapter 04. 파일 읽기/쓰기와 JSON

## 이 장에서 배우는 것

- 파일을 읽고 쓰는 것이 왜 필요한지
- `with open()` 패턴을 왜 써야 하는지
- 인코딩이 무엇이고 왜 문제가 생기는지
- JSON이 무엇이고 언제 쓰는지
- 실제 메모 저장 프로그램을 만들면서 전체를 연결하는 법
- `FileNotFoundError`, `UnicodeDecodeError` 같은 실제 에러 대처법

---

## 왜 파일과 JSON이 필요한가

프로그램을 실행하면 변수에 값을 저장할 수 있다.  
하지만 프로그램을 종료하는 순간 변수에 저장된 값은 모두 사라진다.

```python
memo = "오늘 Python 배움"
# 프로그램 종료 → memo 사라짐
```

이것이 가장 큰 문제다.  
로그인 기록, 메모, 점수 등 "다음에도 쓰고 싶은 데이터"는 파일이나 데이터베이스에 저장해야 한다.

**파일**은 그 중 가장 단순한 저장 방법이다.  
**JSON**은 파일에 데이터를 구조적으로 저장할 때 사용하는 형식이다.

---

## 1. 파일이란 무엇인가

파일은 컴퓨터 디스크에 데이터를 저장하는 공간이다.  
Python에서는 파일을 "열고 → 읽거나 쓰고 → 닫는" 3단계로 다룬다.

파일에는 크게 두 가지 종류가 있다.

| 종류 | 설명 | 예시 |
|------|------|------|
| 텍스트 파일 | 사람이 읽을 수 있는 글자 | `.txt`, `.md`, `.csv` |
| 바이너리 파일 | 컴퓨터가 읽는 이진 데이터 | `.jpg`, `.exe`, `.pdf` |

이 장에서는 텍스트 파일만 다룬다.

---

## 2. 파일 열기 모드

`open()` 함수를 사용할 때는 파일을 어떤 목적으로 여는지 모드를 지정한다.

| 모드 | 의미 | 파일이 없으면? |
|------|------|----------------|
| `"r"` | 읽기 (기본값) | FileNotFoundError 발생 |
| `"w"` | 쓰기 (처음부터 새로 씀) | 새 파일 생성 |
| `"a"` | 이어쓰기 (기존 내용 뒤에 추가) | 새 파일 생성 |
| `"x"` | 쓰기 (파일이 이미 있으면 에러) | 새 파일 생성 |

**자주 하는 실수**: `"r"` 모드로 존재하지 않는 파일을 열려고 하면 에러가 난다.  
`"w"` 모드로 기존 파일을 열면 기존 내용이 전부 지워진다.

---

## 3. `with open()` 패턴 — 왜 이 방식을 써야 하는가

파일을 열면 반드시 닫아야 한다. 닫지 않으면 파일이 잠겨서 다른 프로그램이 읽지 못할 수 있다.

**나쁜 방법 (직접 open/close)**:
```python
file = open("memo.txt", "w", encoding="utf-8")
file.write("오늘 배운 것")
file.close()  # 중간에 에러가 나면 close가 실행되지 않는다!
```

**좋은 방법 (`with` 사용)**:
```python
with open("memo.txt", "w", encoding="utf-8") as file:
    file.write("오늘 배운 것")
# with 블록을 벗어나는 순간 자동으로 close() 된다
```

`with` 문은 블록이 끝나면 에러가 나더라도 파일을 자동으로 닫아준다.  
Python에서 파일을 다룰 때는 항상 `with open()`을 사용하는 것이 원칙이다.

---

## 4. 인코딩이란 무엇인가

**인코딩(encoding)**은 글자를 컴퓨터가 저장하는 숫자로 변환하는 규칙이다.

- 영문만 쓰면 보통 문제가 없다
- 한글, 일본어, 중국어 등 비영어권 문자는 `utf-8` 인코딩을 사용해야 안전하다

**반드시 `encoding="utf-8"`을 쓰는 습관을 들일 것.**

```python
# 인코딩 없이 쓰면 운영체제마다 다르게 동작한다
# Windows에서는 "cp949", macOS/Linux에서는 "utf-8"이 기본값
# 이 때문에 Windows에서 만든 파일이 macOS에서 깨지는 경우가 생긴다
with open("memo.txt", "w", encoding="utf-8") as file:
    file.write("한글 메모")
```

---

## 5. 파일 쓰기

```python
with open("memo.txt", "w", encoding="utf-8") as file:
    file.write("오늘은 Python 파일 입출력을 배웠다.\n")
    file.write("with open() 패턴이 중요하다.\n")
```

- `\n`은 줄 바꿈 문자다. 없으면 두 문장이 한 줄에 이어서 저장된다.
- `"w"` 모드는 파일이 없으면 새로 만들고, 이미 있으면 기존 내용을 지우고 덮어쓴다.

**여러 줄 한 번에 쓰기**:
```python
lines = ["첫 번째 메모\n", "두 번째 메모\n", "세 번째 메모\n"]

with open("memo.txt", "w", encoding="utf-8") as file:
    file.writelines(lines)
```

**이어서 쓰기**:
```python
with open("memo.txt", "a", encoding="utf-8") as file:
    file.write("나중에 추가한 메모\n")
```

---

## 6. 파일 읽기

**전체 읽기**:
```python
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

**한 줄씩 읽기**:
```python
with open("memo.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line.strip())  # strip()은 줄 끝의 \n을 제거한다
```

**줄 목록으로 읽기**:
```python
with open("memo.txt", "r", encoding="utf-8") as file:
    lines = file.readlines()

print(lines)
# ['첫 번째 메모\n', '두 번째 메모\n', '세 번째 메모\n']
```

---

## 7. 실제 에러 예시: FileNotFoundError

파일 읽기에서 가장 자주 만나는 에러다.

```python
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

```
FileNotFoundError: [Errno 2] No such file or directory: 'memo.txt'
```

**이 에러가 나는 이유**:
1. 파일 이름을 잘못 적었다 (`memo.txt` vs `Memo.txt`)
2. 파일이 다른 폴더에 있다
3. 파일을 아직 만들지 않았다

**해결 방법 1 — 파일이 있는지 먼저 확인하기**:
```python
import os

if os.path.exists("memo.txt"):
    with open("memo.txt", "r", encoding="utf-8") as file:
        content = file.read()
    print(content)
else:
    print("파일이 없습니다.")
```

**해결 방법 2 — try/except 사용하기**:
```python
try:
    with open("memo.txt", "r", encoding="utf-8") as file:
        content = file.read()
    print(content)
except FileNotFoundError:
    print("파일을 찾을 수 없습니다. 파일 이름을 확인하세요.")
```

---

## 8. 실제 에러 예시: UnicodeDecodeError

인코딩이 맞지 않을 때 발생하는 에러다.

```python
with open("memo.txt", "r") as file:  # encoding을 지정하지 않음
    content = file.read()
```

```
UnicodeDecodeError: 'cp949' codec can't decode byte 0xec in position 0
```

**이 에러가 나는 이유**:
- 파일은 `utf-8`로 저장되어 있는데, 읽을 때 다른 인코딩(`cp949` 등)으로 읽으려 했다

**해결 방법**:
```python
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()
```

항상 `encoding="utf-8"`을 명시하면 이 문제를 거의 피할 수 있다.

---

## 9. JSON이란 무엇인가

JSON(JavaScript Object Notation)은 데이터를 구조적으로 저장하는 텍스트 형식이다.

**왜 JSON을 쓰는가**:
- 숫자, 문자열, 리스트, 딕셔너리를 하나의 파일에 구조 그대로 저장할 수 있다
- 텍스트 형식이라 사람이 읽을 수 있다
- Python, JavaScript, Java 등 거의 모든 언어에서 읽을 수 있어 데이터를 주고받기 편하다

```json
{
  "name": "Mina",
  "age": 14,
  "favorite_subjects": ["수학", "영어", "Python"],
  "passed": true
}
```

**Python 자료형과 JSON 대응표**:

| Python | JSON |
|--------|------|
| dict | object `{}` |
| list | array `[]` |
| str | string `""` |
| int, float | number |
| True / False | true / false |
| None | null |

---

## 10. Python에서 JSON 저장하기

```python
import json

student = {
    "name": "Mina",
    "age": 14,
    "favorite_subjects": ["수학", "영어", "Python"],
    "passed": True
}

with open("student.json", "w", encoding="utf-8") as file:
    json.dump(student, file, ensure_ascii=False, indent=2)
```

- `json.dump()`: Python 객체 → JSON 파일로 저장
- `ensure_ascii=False`: 한글을 그대로 저장 (False가 아니면 `한글` 같이 저장됨)
- `indent=2`: 보기 좋게 들여쓰기 (없으면 한 줄로 저장됨)

저장 결과:
```json
{
  "name": "Mina",
  "age": 14,
  "favorite_subjects": [
    "수학",
    "영어",
    "Python"
  ],
  "passed": true
}
```

---

## 11. Python에서 JSON 읽기

```python
import json

with open("student.json", "r", encoding="utf-8") as file:
    student = json.load(file)

print(student["name"])           # Mina
print(student["favorite_subjects"][0])  # 수학
```

- `json.load()`: JSON 파일 → Python 객체로 불러오기

**json.dump vs json.load 기억법**:
- `dump` = 파일에 "dump(투하)한다" → 저장
- `load` = 파일에서 "load(불러온다)" → 읽기

---

## 12. json.dumps / json.loads — 문자열과 JSON 변환

파일이 아니라 문자열과 변환할 때 쓴다.

```python
import json

# Python 딕셔너리 → JSON 문자열
student = {"name": "Mina", "age": 14}
json_str = json.dumps(student, ensure_ascii=False)
print(json_str)   # {"name": "Mina", "age": 14}
print(type(json_str))  # <class 'str'>

# JSON 문자열 → Python 딕셔너리
data = json.loads(json_str)
print(data["name"])  # Mina
print(type(data))    # <class 'dict'>
```

---

## 실습 1 (따라 하기). 텍스트 메모 저장하고 읽기

**목표**: 메모 파일을 만들고, 내용을 추가하고, 전체를 출력한다.

```python
# 1단계: 메모 파일 생성
with open("my_memo.txt", "w", encoding="utf-8") as file:
    file.write("오늘 배운 것: with open() 패턴\n")
    file.write("오늘 배운 것: encoding='utf-8'\n")

print("메모 저장 완료")

# 2단계: 메모 추가
with open("my_memo.txt", "a", encoding="utf-8") as file:
    file.write("오늘 배운 것: json.dump / json.load\n")

print("메모 추가 완료")

# 3단계: 전체 메모 읽기
with open("my_memo.txt", "r", encoding="utf-8") as file:
    content = file.read()

print("=== 저장된 메모 ===")
print(content)
```

**직접 해보기**: 위 코드를 실행한 뒤, `"a"` 모드를 `"w"` 모드로 바꾸면 어떻게 되는지 확인해보자. 앞에 쓴 내용은 어떻게 될까?

---

## 실습 2 (따라 하기). JSON으로 프로필 저장하고 읽기

**목표**: 사용자 프로필을 JSON 파일로 저장하고, 다시 불러와 특정 값을 출력한다.

```python
import json

# 1단계: 프로필 데이터 만들기
profile = {
    "name": "Jisoo",
    "age": 15,
    "city": "Seoul",
    "hobbies": ["독서", "코딩", "그림"]
}

# 2단계: JSON 파일로 저장
with open("profile.json", "w", encoding="utf-8") as file:
    json.dump(profile, file, ensure_ascii=False, indent=2)

print("프로필 저장 완료")

# 3단계: JSON 파일에서 읽기
with open("profile.json", "r", encoding="utf-8") as file:
    loaded_profile = json.load(file)

print(f"이름: {loaded_profile['name']}")
print(f"사는 곳: {loaded_profile['city']}")
print(f"취미 첫 번째: {loaded_profile['hobbies'][0]}")
```

**직접 해보기**: `hobbies` 리스트에 새 취미를 추가하고 파일을 다시 저장해보자.  
힌트: `loaded_profile["hobbies"].append("음악")` 처럼 리스트에 추가한 뒤 `json.dump`를 다시 실행한다.

---

## 실습 3 (따라 하기). 실제 메모 저장 프로그램 만들기

**목표**: 메모를 JSON 파일에 목록으로 쌓아두는 간단한 프로그램을 만든다.

```python
import json
import os

MEMO_FILE = "memos.json"

def load_memos():
    """파일에서 메모 목록을 불러온다. 파일이 없으면 빈 리스트를 반환한다."""
    if not os.path.exists(MEMO_FILE):
        return []
    try:
        with open(MEMO_FILE, "r", encoding="utf-8") as file:
            return json.load(file)
    except json.JSONDecodeError:
        print("메모 파일이 손상되었습니다. 새로 시작합니다.")
        return []

def save_memos(memos):
    """메모 목록을 파일에 저장한다."""
    with open(MEMO_FILE, "w", encoding="utf-8") as file:
        json.dump(memos, file, ensure_ascii=False, indent=2)

def add_memo(text):
    """새 메모를 추가하고 저장한다."""
    memos = load_memos()
    memos.append(text)
    save_memos(memos)
    print(f"메모 추가됨: {text}")

def show_memos():
    """저장된 메모를 모두 출력한다."""
    memos = load_memos()
    if not memos:
        print("저장된 메모가 없습니다.")
        return
    print("=== 저장된 메모 목록 ===")
    for i, memo in enumerate(memos, start=1):
        print(f"{i}. {memo}")

# 실행
add_memo("with open() 패턴을 배웠다")
add_memo("JSON은 구조적으로 데이터를 저장한다")
add_memo("encode='utf-8'은 항상 써야 한다")
show_memos()
```

**직접 해보기**: 
1. 프로그램을 두 번 실행해보자. 두 번째 실행 때 메모가 이어서 쌓이는가?
2. `delete_memo(index)` 함수를 직접 만들어보자. 특정 번호의 메모를 지우고 다시 저장하는 함수다.

---

## 자주 막히는 지점 (Common Pitfalls)

### Pitfall 1. `r`과 `w` 모드를 헷갈린다

```python
# 잘못된 예: 파일이 없는데 "r"로 읽으려 한다
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()
# → FileNotFoundError 발생
```

해결: 파일을 먼저 `"w"` 또는 `"a"` 모드로 만든 다음에 `"r"` 모드로 읽는다.

---

### Pitfall 2. `"w"` 모드가 기존 파일을 덮어쓴다

```python
with open("memo.txt", "w", encoding="utf-8") as file:
    file.write("첫 번째 메모")

with open("memo.txt", "w", encoding="utf-8") as file:
    file.write("두 번째 메모")  # 첫 번째 메모는 사라진다!
```

해결: 이어서 추가할 때는 `"a"` 모드를 사용한다.

---

### Pitfall 3. `encoding="utf-8"` 을 빠뜨린다

```python
# Windows 환경에서 한글이 깨질 수 있다
with open("memo.txt", "w") as file:
    file.write("한글 메모")
```

해결: 항상 `encoding="utf-8"` 을 명시한다.

---

### Pitfall 4. `json.dump`와 `json.load` 방향을 헷갈린다

- `json.dump(data, file)` → 데이터를 파일에 **넣는다** (저장)
- `json.load(file)` → 파일에서 데이터를 **꺼낸다** (읽기)

---

### Pitfall 5. `ensure_ascii=False`를 빠뜨린다

```python
import json

data = {"message": "안녕하세요"}
json_str = json.dumps(data)
print(json_str)
# {"message": "안녕하세요"}  ← 한글이 유니코드 코드로 변환됨

json_str = json.dumps(data, ensure_ascii=False)
print(json_str)
# {"message": "안녕하세요"}  ← 한글이 그대로 유지됨
```

---

### Pitfall 6. 파일 내용이 JSON인지 확인하지 않는다

파일이 비어있거나 JSON 형식이 아닐 때 `json.load()`는 에러가 난다.

```python
import json

try:
    with open("data.json", "r", encoding="utf-8") as file:
        data = json.load(file)
except json.JSONDecodeError as e:
    print(f"JSON 파싱 에러: {e}")
```

---

## 확인 체크리스트

- `with open()` 패턴을 왜 써야 하는지 설명할 수 있는가
- `"r"`, `"w"`, `"a"` 모드의 차이를 말할 수 있는가
- `encoding="utf-8"` 을 왜 써야 하는지 설명할 수 있는가
- `json.dump()`와 `json.load()`의 역할을 말할 수 있는가
- `FileNotFoundError`가 나면 무엇을 확인해야 하는지 알고 있는가
- `UnicodeDecodeError`가 나면 어떻게 해결하는지 알고 있는가
- 메모 저장 프로그램을 스스로 수정할 수 있는가

---

## 한 번 더 생각해 보기

1. 프로그램을 종료해도 데이터가 남아있게 하려면 무엇이 필요한가?
2. `json.dump`와 `json.dumps`의 차이는 무엇인가? (힌트: `s`가 붙으면 string)
3. 메모 저장 프로그램에서 `load_memos()`가 없는 파일을 빈 리스트로 처리하는 이유는 무엇인가?
4. `ensure_ascii=False`를 넣지 않으면 JSON 파일을 열었을 때 어떤 문제가 생길까?

---

## 교사용 메모

- 강조: 파일은 저장 공간, 변수는 실행 중 임시 공간임을 그림으로 설명하면 효과적이다.
- 막힘 포인트: `r/w` 모드 혼동, `json.dump/load` 방향, 저장된 파일 위치 확인에서 자주 멈춘다.
- 실습 3의 메모 프로그램을 먼저 실행하게 하고, 거기서 역으로 각 함수의 역할을 분석하게 하는 방식이 효과적이다.
- 질문 1: `json.dump()`는 넣는 동작일까, 꺼내는 동작일까?
- 질문 2: 같은 파일을 `"w"` 모드로 두 번 열면 첫 번째 내용은 어떻게 될까?
