## 이 장에서 배우는 것

- 변수와 자료형을 실제 상황에 맞게 사용하는 방법
- `class`로 데이터를 묶어서 표현하는 방법
- `JSON`으로 데이터를 저장하고 불러오는 방법
- `assert`로 코드가 올바르게 동작하는지 확인하는 방법
- 오류 메시지를 읽고 스스로 디버깅하는 방법
- 위 개념들을 하나의 흐름으로 연결해서 쓰는 방법

---

## 먼저 쉬운 설명

지금까지 변수, 클래스, JSON, assert를 따로따로 배웠습니다. 그런데 실제 코드를 짤 때는 이것들이 모두 함께 쓰입니다.

예를 들어 학생 성적 관리 프로그램을 만든다고 생각해 보세요.

- 학생 정보를 **클래스**로 묶고
- 성적이 올바른지 **assert**로 확인하고
- 데이터를 **JSON** 파일에 저장하고
- 문제가 생기면 오류 메시지를 읽어서 **디버깅**합니다

이 장에서는 그 흐름을 처음부터 끝까지 함께 만들어 봅니다. 작은 프로그램이지만, 실무에서 쓰는 방식과 완전히 같습니다.

---

## 1. 변수로 학생 정보 표현하기

가장 먼저 변수로 학생 한 명의 정보를 표현해 봅니다.

```python
# student_basic.py

student_name = "김민준"
student_age = 17
student_score = 85

print(f"이름: {student_name}")
print(f"나이: {student_age}살")
print(f"점수: {student_score}점")
```

실행 결과:

```
이름: 김민준
나이: 17살
점수: 85점
```

변수만으로도 정보를 담을 수 있지만, 학생이 10명이 되면 변수가 30개로 늘어납니다. 이럴 때 `class`를 씁니다.

---

## 2. class로 학생 정보 묶기

`class`는 관련된 정보를 하나의 덩어리로 묶어 주는 도구입니다. 서랍에 이름표를 붙이는 것과 같습니다.

```python
# student_class.py

class Student:
    def __init__(self, name, age, score):
        self.name = name
        self.age = age
        self.score = score

    def introduce(self):
        print(f"안녕하세요, 저는 {self.name}이고 점수는 {self.score}점입니다.")


# 학생 객체 만들기
student1 = Student("김민준", 17, 85)
student2 = Student("이서연", 16, 92)

student1.introduce()
student2.introduce()
```

실행 결과:

```
안녕하세요, 저는 김민준이고 점수는 85점입니다.
안녕하세요, 저는 이서연이고 점수는 92점입니다.
```

> **핵심 포인트**: `self`는 "이 객체 자신"을 가리킵니다. `self.name`은 "이 학생의 이름"이라는 뜻입니다.

---

## 3. assert로 데이터 검증하기

점수는 0점에서 100점 사이여야 합니다. 실수로 200점을 입력하면 어떻게 될까요? `assert`를 쓰면 잘못된 데이터가 들어오는 순간 바로 알 수 있습니다.

```python
# student_validate.py

class Student:
    def __init__(self, name, age, score):
        assert isinstance(name, str), "이름은 문자열이어야 합니다"
        assert 0 <= score <= 100, f"점수는 0~100 사이여야 합니다. 입력값: {score}"

        self.name = name
        self.age = age
        self.score = score


# 정상적인 경우
good_student = Student("김민준", 17, 85)
print(f"{good_student.name} 등록 완료")

# 잘못된 점수 — 아래 줄의 주석을 해제하면 오류가 발생합니다
# bad_student = Student("오류학생", 17, 200)
```

주석을 해제하면 이런 오류가 납니다:

```
AssertionError: 점수는 0~100 사이여야 합니다. 입력값: 200
```

`assert` 뒤에 쉼표로 메시지를 붙이면 오류가 났을 때 원인을 바로 알 수 있어서 디버깅이 쉬워집니다.

---

## 4. JSON으로 데이터 저장하고 불러오기

프로그램을 종료하면 변수에 저장된 데이터는 사라집니다. `JSON` 파일에 저장하면 다음에 다시 불러올 수 있습니다.

```python
# student_json.py
import json

class Student:
    def __init__(self, name, age, score):
        assert 0 <= score <= 100, f"점수는 0~100 사이여야 합니다. 입력값: {score}"
        self.name = name
        self.age = age
        self.score = score

    def to_dict(self):
        """객체를 딕셔너리로 변환합니다."""
        return {
            "name": self.name,
            "age": self.age,
            "score": self.score
        }


def save_students(students, filename):
    """학생 목록을 JSON 파일에 저장합니다."""
    data = [s.to_dict() for s in students]
    with open(filename, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f"{filename}에 저장했습니다.")


def load_students(filename):
    """JSON 파일에서 학생 목록을 불러옵니다."""
    with open(filename, "r", encoding="utf-8") as f:
        data = json.load(f)
    students = [Student(d["name"], d["age"], d["score"]) for d in data]
    print(f"{filename}에서 {len(students)}명을 불러왔습니다.")
    return students


# 학생 목록 만들기
students = [
    Student("김민준", 17, 85),
    Student("이서연", 16, 92),
    Student("박지호", 17, 78),
]

# 저장
save_students(students, "students.json")

# 다시 불러오기
loaded = load_students("students.json")
for s in loaded:
    print(f"  {s.name}: {s.score}점")
```

실행 결과:

```
students.json에 저장했습니다.
students.json에서 3명을 불러왔습니다.
  김민준: 85점
  이서연: 92점
  박지호: 78점
```

저장된 `students.json` 파일은 이렇게 생겼습니다:

```json
[
  {
    "name": "김민준",
    "age": 17,
    "score": 85
  },
  {
    "name": "이서연",
    "age": 16,
    "score": 92
  },
  {
    "name": "박지호",
    "age": 17,
    "score": 78
  }
]
```

---

## 5. 오류 메시지 읽고 디버깅하기

오류가 나도 당황하지 않는 것이 중요합니다. Python 오류 메시지는 항상 **어디서**, **왜** 문제가 생겼는지 알려줍니다.

```python
# student_debug.py
import json

# 의도적으로 오류가 있는 코드입니다

def load_students_broken(filename):
    with open(filename, "r", encoding="utf-8") as f:
        data = json.load(f)
    # 버그: score 대신 잘못된 키 'grade'를 참조하고 있습니다
    students = [{"name": d["name"], "score": d["grade"]} for d in data]
    return students

try:
    result = load_students_broken("students.json")
except KeyError as e:
    print(f"[오류] 딕셔너리에 키가 없습니다: {e}")
    print("힌트: JSON 파일의 키 이름을 확인해 보세요.")
```

실행 결과:

```
[오류] 딕셔너리에 키가 없습니다: 'grade'
힌트: JSON 파일의 키 이름을 확인해 보세요.
```

**디버깅 3단계 습관**:

1. 오류 메시지 마지막 줄을 먼저 읽는다 → 오류 종류와 원인이 있다
2. `Traceback`에서 내 파일 이름이 나오는 줄 번호를 찾는다
3. 그 줄 근처에서 변수나 키 이름이 맞는지 확인한다

---

## 따라 하기 실습

### 실습 1 — 기본 클래스 만들기

`book_manager.py` 파일을 새로 만들고 아래를 따라 입력합니다.

```python
# book_manager.py

class Book:
    def __init__(self, title, author, pages):
        assert isinstance(title, str), "제목은 문자열이어야 합니다"
        assert pages > 0, "페이지 수는 0보다 커야 합니다"
        self.title = title
        self.author = author
        self.pages = pages

    def summary(self):
        print(f"『{self.title}』 - {self.author} 지음 ({self.pages}쪽)")


book1 = Book("파이썬 한 입", "홍길동", 320)
book2 = Book("데이터 입문", "김철수", 250)

book1.summary()
book2.summary()
```

터미널에서 실행합니다:

```bash
python book_manager.py
```

기대 출력:

```
『파이썬 한 입』 - 홍길동 지음 (320쪽)
『데이터 입문』 - 김철수 지음 (250쪽)
```

---

### 실습 2 — JSON 저장 기능 추가하기

`book_manager.py`에 아래 함수들을 추가합니다. 기존 코드 아래에 이어서 붙여 넣으세요.

```python
import json  # 파일 맨 위로 옮겨 주세요

def save_books(books, filename):
    data = [{"title": b.title, "author": b.author, "pages": b.pages} for b in books]
    with open(filename, "w", encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f"'{filename}'에 {len(books)}권 저장 완료")


def load_books(filename):
    with open(filename, "r", encoding="utf-8") as f:
        data = json.load(f)
    return [Book(d["title"], d["author"], d["pages"]) for d in data]


# 기존 book1, book2 코드 아래에 추가
books = [book1, book2]
save_books(books, "books.json")

loaded_books = load_books("books.json")
print("--- 불러온 책 목록 ---")
for b in loaded_books:
    b.summary()
```

실행하면 `books.json` 파일이 생기고 불러온 목록이 출력됩니다.

---

### 실습 3 — 검증 오류 일부러 만들어 보기

`book_manager.py` 맨 아래에 아래 코드를 추가하고 실행해서 오류 메시지를 직접 확인합니다.

```python
print("\n--- 오류 테스트 ---")

try:
    bad_book = Book("잘못된 책", "작가", -10)  # pages가 음수
except AssertionError as e:
    print(f"[AssertionError 발생] {e}")

try:
    bad_book2 = Book(12345, "작가", 100)  # title이 숫자
except AssertionError as e:
    print(f"[AssertionError 발생] {e}")

print("오류를 잘 잡았습니다!")
```

기대 출력:

```
--- 오류 테스트 ---
[AssertionError 발생] 페이지 수는 0보다 커야 합니다
[AssertionError 발생] 제목은 문자열이어야 합니다
오류를 잘 잡았습니다!
```

`assert`가 잘못된 데이터를 막아 주는 것을 직접 눈으로 확인했습니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 원인 | 수정 방법 |
|------|------------|------|-----------|
| `__init__` 철자 오타 | `TypeError: object.__init__() takes exactly one argument` | `__init__`을 `_init_`으로 쓴 경우 | 언더바 두 개씩 확인 (`__init__`) |
| `self` 빠뜨리기 | `NameError: name 'name' is not defined` | 메서드 안에서 `self.name` 대신 `name`만 씀 | 클래스 메서드 안에서는 항상 `self.` 붙이기 |
| JSON 파일 없이 `load` 시도 | `FileNotFoundError: [Errno 2] No such file or directory` | 저장하기 전에 불러오려 함 | `save` 먼저 실행 후 `load` 실행 |
| JSON 키 이름 불일치 | `KeyError: 'grade'` | 저장할 때 쓴 키와 불러올 때 쓴 키가 다름 | 양쪽 키 이름이 완전히 같은지 확인 |
| `assert` 메시지를 괄호로 감쌈 | `SyntaxWarning` 또는 항상 통과 | `assert(조건, "메시지")` 형태로 씀 | 괄호 없이 `assert 조건, "메시지"` 로 쓰기 |
| `ensure_ascii=False` 누락 | JSON 파일에 `\uc131` 같은 코드가 저장됨 | 한글이 유니코드 이스케이프로 저장됨 | `json.dump(..., ensure_ascii=False)` 추가 |

---

## 확인 체크리스트

스스로 아래 항목을 하나씩 확인해 보세요.

- [ ] `class`를 만들고 `__init__`으로 속성을 초기화할 수 있다
- [ ] `self`가 무엇을 가리키는지 설명할 수 있다
- [ ] `assert`로 잘못된 입력값을 막을 수 있다
- [ ] `assert` 오류가 났을 때 메시지로 원인을 파악할 수 있다
- [ ] `json.dump`로 파이썬 객체를 JSON 파일에 저장할 수 있다
- [ ] `json.load`로 JSON 파일을 다시 파이썬 객체로 불러올 수 있다
- [ ] `FileNotFoundError`와 `KeyError`가 각각 왜 나는지 설명할 수 있다
- [ ] `try / except`로 오류를 잡아서 프로그램이 멈추지 않게 할 수 있다
- [ ] 실습 3의 오류 테스트 코드를 직접 작성하고 실행했다

---

## 한 번 더 생각해 보기

1. `assert` 대신 `if`문으로 검증해도 됩니다. 그렇다면 `assert`를 쓰는 이유는 무엇일까요? 어떤 상황에서 `if`가 더 낫고, 어떤 상황에서 `assert`가 더 나을까요?

2. `to_dict()` 메서드를 클래스 안에 넣는 것과, 클래스 밖에서 `{"name": student.name, ...}` 처럼 직접 딕셔너리를 만드는 것의 차이는 무엇일까요? 학생이 100명이라면 어느 쪽이 더 편할까요?

3. JSON 파일을 불러올 때 파일이 없으면 `FileNotFoundError`가 납니다. 이 오류를 `try / except`로 잡는 대신, 파일이 없을 때 빈 리스트를 반환하도록 `load_books` 함수를 고쳐 보세요. 어떤 방법이 더 안전한 코드일까요?

---

## 다음 장

다음 장에서는 오늘 만든 `Book` 클래스를 발전시켜, 여러 클래스가 서로 관계를 맺는 **클래스 상속(inheritance)**을 배웁니다.