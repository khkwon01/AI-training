# Python 누적 실습 01 정답 가이드

## 이 문서의 목적

누적 실습 01을 풀다가 막혔을 때 활용하는 가이드다. 정답을 바로 보기 전에 단계별 힌트를 먼저 읽어보자. 힌트만으로 해결되면 그게 가장 좋은 학습이다.

---

## 실습 문제 요약

아래 요구사항을 만족하는 학생 성적 관리 프로그램을 작성한다.

1. `Student` 클래스를 만들고 이름, 나이, 좋아하는 프로그래밍 언어 속성을 가진다
2. `introduce()` 메서드로 자기소개 문장을 반환한다
3. 학생 객체를 JSON 파일로 저장한다
4. JSON 파일을 다시 읽어서 데이터를 검증한다

---

## 단계별 힌트 (막혔을 때 순서대로 읽기)

### 힌트 1: 클래스부터 차근차근 만들기

`class` 키워드로 시작하고, `__init__` 메서드에서 속성을 정의한다.

```python
class Student:
    def __init__(self, name, age, favorite_language):
        self.name = ???          # 채울 것
        self.age = ???           # 채울 것
        self.favorite_language = ???  # 채울 것
```

`self.name = name` 처럼, 파라미터를 `self.` 속성에 담는 패턴이다.

클래스가 완성되면 객체를 하나 만들어 보자:
```python
student1 = Student("Mina", 14, "Python")
```

---

### 힌트 2: 객체를 만든 뒤 값 확인하기

객체를 만들었으면 값이 제대로 들어갔는지 `print()`로 확인한다. 이 확인 단계를 건너뛰면 나중에 오류가 났을 때 원인을 찾기 어렵다.

```python
print(student1.name)             # → Mina
print(student1.age)              # → 14
print(student1.favorite_language)  # → Python
```

값이 기대한 대로 출력되면 다음 단계로 넘어간다.

---

### 힌트 3: introduce() 메서드 만들기

메서드는 클래스 안에서 들여쓰기를 맞춰야 한다. 자기소개 문장을 `return`으로 반환한다.

```python
def introduce(self):
    return f"저는 {self.???}이고, {self.???}살이며, {self.???}를 좋아합니다."
```

`f"..."` 는 f-string 이라고 부른다. `{}` 안에 변수를 넣으면 값이 삽입된다.

메서드 확인:
```python
print(student1.introduce())
# → 저는 Mina이고, 14살이며, Python을 좋아합니다.
```

---

### 힌트 4: JSON으로 저장하기 전에 딕셔너리로 바꾸기

Python 클래스 객체를 그대로 `json.dump()`에 넣으면 오류가 난다. 먼저 딕셔너리로 변환해야 한다.

```python
student_data = {
    "name": student1.name,
    "age": student1.age,
    "favorite_language": student1.favorite_language
}
```

이 딕셔너리를 파일에 저장:

```python
import json

with open("student1.json", "w", encoding="utf-8") as file:
    json.dump(student_data, file, ensure_ascii=False, indent=2)
```

파라미터 설명:
- `"w"` — 쓰기 모드 (파일이 없으면 새로 만들고, 있으면 덮어씀)
- `encoding="utf-8"` — 한글이 깨지지 않도록
- `ensure_ascii=False` — 한글을 그대로 저장 (False 안 하면 `에끼` 같은 코드로 저장됨)
- `indent=2` — 읽기 좋게 2칸 들여쓰기

---

### 힌트 5: 저장 후 다시 읽어서 확인하기 + assert로 검증

파일을 저장했으면 다시 읽어서 값이 올바른지 확인한다.

```python
with open("student1.json", "r", encoding="utf-8") as file:
    loaded_data = json.load(file)

print(loaded_data)
# → {'name': 'Mina', 'age': 14, 'favorite_language': 'Python'}
```

`assert`로 자동 검증:

```python
assert loaded_data["name"] == "Mina", "이름이 다릅니다!"
assert loaded_data["age"] == 14, "나이가 다릅니다!"
```

`assert` 는 조건이 True면 아무 일도 없고, False면 AssertionError를 발생시킨다. 코드가 올바르게 동작하는지 확인하는 간단한 방법이다.

---

## 완전한 정답 코드

아래는 모든 힌트를 합친 완전한 정답이다. 각 라인에 주석으로 설명을 달았다.

```python
# student_manager.py
# 학생 정보를 관리하는 간단한 프로그램

import json  # JSON 파일 저장/읽기에 필요한 모듈


# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Student 클래스 정의
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

class Student:
    """학생 정보를 담는 클래스"""

    def __init__(self, name, age, favorite_language):
        """
        Student 객체를 만들 때 자동으로 호출되는 초기화 메서드
        
        파라미터:
            name (str): 학생 이름
            age (int): 학생 나이
            favorite_language (str): 좋아하는 프로그래밍 언어
        """
        self.name = name                      # 이름을 속성으로 저장
        self.age = age                        # 나이를 속성으로 저장
        self.favorite_language = favorite_language  # 좋아하는 언어를 속성으로 저장

    def introduce(self):
        """자기소개 문장을 반환하는 메서드"""
        return f"저는 {self.name}이고, {self.age}살이며, {self.favorite_language}를 좋아합니다."

    def to_dict(self):
        """
        객체를 딕셔너리로 변환하는 메서드
        JSON 저장 전에 호출한다
        """
        return {
            "name": self.name,
            "age": self.age,
            "favorite_language": self.favorite_language
        }


# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# 메인 실행 코드
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# 1. 학생 객체 만들기
student1 = Student("Mina", 14, "Python")

# 2. 값 확인
print("=== 학생 정보 확인 ===")
print("이름:", student1.name)
print("나이:", student1.age)
print("좋아하는 언어:", student1.favorite_language)
print("자기소개:", student1.introduce())
print()

# 3. 딕셔너리로 변환
student_data = student1.to_dict()
print("=== 딕셔너리 변환 결과 ===")
print(student_data)
print()

# 4. JSON 파일로 저장
file_path = "student1.json"

with open(file_path, "w", encoding="utf-8") as file:
    json.dump(student_data, file, ensure_ascii=False, indent=2)

print(f"=== {file_path} 저장 완료 ===")

# 5. JSON 파일 다시 읽기
with open(file_path, "r", encoding="utf-8") as file:
    loaded_data = json.load(file)

print("=== 파일에서 읽은 데이터 ===")
print(loaded_data)
print()

# 6. 검증 (assert)
assert loaded_data["name"] == "Mina", "이름이 저장된 값과 다릅니다!"
assert loaded_data["age"] == 14, "나이가 저장된 값과 다릅니다!"
assert loaded_data["favorite_language"] == "Python", "좋아하는 언어가 저장된 값과 다릅니다!"

print("=== 모든 검증 통과! ===")
```

### 예상 출력

```
=== 학생 정보 확인 ===
이름: Mina
나이: 14
좋아하는 언어: Python
자기소개: 저는 Mina이고, 14살이며, Python을 좋아합니다.

=== 딕셔너리 변환 결과 ===
{'name': 'Mina', 'age': 14, 'favorite_language': 'Python'}

=== student1.json 저장 완료 ===
=== 파일에서 읽은 데이터 ===
{'name': 'Mina', 'age': 14, 'favorite_language': 'Python'}

=== 모든 검증 통과! ===
```

### 저장된 student1.json 파일 내용

```json
{
  "name": "Mina",
  "age": 14,
  "favorite_language": "Python"
}
```

---

## 코드 각 부분 상세 설명

### class 정의 부분

```python
class Student:
```

`class`는 속성(데이터)과 메서드(동작)를 하나로 묶는 틀이다. `Student`라는 이름의 틀을 정의한다. 클래스 이름은 관례상 대문자로 시작한다.

---

```python
def __init__(self, name, age, favorite_language):
    self.name = name
```

`__init__`은 **생성자(constructor)**라고 부른다. `Student("Mina", 14, "Python")` 처럼 객체를 만들 때 자동으로 실행된다.

`self`는 "이 객체 자신"을 가리키는 특별한 파라미터다. `self.name`은 "이 객체의 name 속성"이라는 뜻이다.

---

### to_dict() 메서드

```python
def to_dict(self):
    return {
        "name": self.name,
        ...
    }
```

이 메서드를 따로 만든 이유:

1. `json.dump()`는 Python 딕셔너리는 처리할 수 있지만, 직접 만든 클래스 객체는 처리하지 못한다
2. 매번 딕셔너리를 직접 만들지 않고 메서드를 호출하면 코드가 더 깔끔하다

---

### with open() 구문

```python
with open("student1.json", "w", encoding="utf-8") as file:
    json.dump(student_data, file, ensure_ascii=False, indent=2)
```

`with` 구문은 파일을 열고 작업이 끝나면 자동으로 닫아준다. `with` 없이 `open()`만 쓰면 `file.close()`를 직접 호출해야 한다. `with`를 쓰는 것이 훨씬 안전하다.

---

### assert 검증

```python
assert loaded_data["name"] == "Mina", "이름이 저장된 값과 다릅니다!"
```

`assert 조건, 메시지` 형태다.
- 조건이 `True` → 아무 일도 없음, 계속 진행
- 조건이 `False` → `AssertionError: 이름이 저장된 값과 다릅니다!` 발생

실제 프로젝트에서는 `unittest`나 `pytest` 같은 테스트 프레임워크를 쓰지만, 간단한 검증에는 `assert`가 충분하다.

---

## 확장 문제: 도전 과제 3개

기본 문제를 다 풀었다면 아래 도전 과제를 시도해 보자.

### 도전 1: 학생 여러 명 처리하기

학생이 3명이고, 이들을 리스트에 담아 모두 JSON 파일 하나에 저장해 보자.

힌트:
```python
students = [
    Student("Mina", 14, "Python"),
    Student("Junho", 15, "JavaScript"),
    Student("Sora", 13, "Java")
]

# students 리스트의 각 Student 객체를 딕셔너리로 변환해서
# all_students.json 파일에 저장해 보자
```

JSON 파일 예상 구조:
```json
[
  {"name": "Mina", "age": 14, "favorite_language": "Python"},
  {"name": "Junho", "age": 15, "favorite_language": "JavaScript"},
  {"name": "Sora", "age": 13, "favorite_language": "Java"}
]
```

---

### 도전 2: 평균 나이 계산 메서드 추가하기

`StudentManager` 클래스를 만들고, 학생 리스트를 관리하는 기능을 추가해 보자.

힌트:
```python
class StudentManager:
    def __init__(self):
        self.students = []     # 학생을 담을 빈 리스트
    
    def add_student(self, student):
        # student를 self.students에 추가하는 코드
        pass
    
    def average_age(self):
        # 모든 학생의 평균 나이를 계산해서 반환하는 코드
        pass
    
    def find_by_language(self, language):
        # 특정 언어를 좋아하는 학생 목록을 반환하는 코드
        pass
```

---

### 도전 3: 점수 추가하고 등급 계산하기

`Student` 클래스에 `scores` 속성(점수 목록)과 `grade()` 메서드를 추가해 보자.

힌트:
```python
class Student:
    def __init__(self, name, age, favorite_language):
        self.name = name
        self.age = age
        self.favorite_language = favorite_language
        self.scores = []       # 점수를 담을 빈 리스트 추가
    
    def add_score(self, score):
        # scores에 점수를 추가
        pass
    
    def average_score(self):
        # 평균 점수 반환 (점수가 없으면 0 반환)
        pass
    
    def grade(self):
        # 평균 점수에 따라 등급 반환
        # 90 이상: "A"
        # 80 이상: "B"
        # 70 이상: "C"
        # 그 외: "F"
        pass
```

테스트:
```python
student1 = Student("Mina", 14, "Python")
student1.add_score(85)
student1.add_score(92)
student1.add_score(88)

print(student1.average_score())  # → 88.33...
print(student1.grade())          # → "B"
```

---

## AI에게 질문하는 방법

막혔을 때 AI(ChatGPT, Claude 등)에게 질문하면 빠르게 도움을 받을 수 있다. 효과적인 질문 방법을 알아두자.

### 기본 원칙

좋은 질문에는 세 가지가 포함된다:
1. 무엇을 하려고 하는가
2. 어떤 코드를 작성했는가
3. 어떤 오류 또는 문제가 발생했는가

---

### 질문 예시 1: 오류가 났을 때

```
Python 초보자입니다.
Student 클래스를 만들고 JSON 파일에 저장하는 코드를 작성하고 있습니다.

아래 코드를 실행했더니 오류가 났습니다.

--- 코드 ---
import json

class Student:
    def __init__(self, name):
        self.name = name

student1 = Student("Mina")
json.dump(student1, open("test.json", "w"))

--- 오류 메시지 ---
TypeError: Object of type Student is not JSON serializable

--- 질문 ---
이 오류가 왜 발생하는지, 어떻게 수정해야 하는지 알려주세요.
```

---

### 질문 예시 2: 방법을 모를 때

```
Python으로 학생 성적 관리 프로그램을 만들고 있습니다.
현재 Student 클래스에 name, age, scores(점수 리스트) 속성이 있습니다.

scores 리스트의 평균을 계산하는 메서드를 만들고 싶은데,
scores 리스트가 비어 있을 때 0을 나누는 오류(ZeroDivisionError)가 날 것 같습니다.
이 경우를 어떻게 처리하면 좋을까요?

코드 예시도 함께 보여주시면 감사하겠습니다.
```

---

### 질문 예시 3: 개념을 이해하고 싶을 때

```
Python의 with open() 구문이 왜 필요한지 모르겠습니다.
그냥 open()만 사용하면 안 되나요?
초등학생도 이해할 수 있는 쉬운 비유로 설명해 주세요.
```

---

### AI 답변 활용 팁

1. AI가 준 코드를 그냥 복사하지 말고, 한 줄씩 읽어서 이해한다
2. 이해 안 되는 부분은 "이 코드에서 `with`가 하는 역할이 무엇인가요?" 처럼 다시 질문한다
3. AI의 설명을 읽은 후 책 없이 스스로 다시 작성해 본다

---

## 정답 확인 체크리스트

이 문제를 완전히 이해했다면 아래 항목에 답할 수 있어야 한다.

- [ ] `class Student:`와 `def __init__(self, ...):`의 관계를 설명할 수 있는가?
- [ ] `self.name = name` 에서 왜 `self.`를 붙이는지 설명할 수 있는가?
- [ ] 클래스 객체를 JSON에 바로 저장하지 못하는 이유를 설명할 수 있는가?
- [ ] `with open()` 구문이 `open()` 단독 사용보다 좋은 이유를 말할 수 있는가?
- [ ] `ensure_ascii=False`가 없으면 한글이 어떻게 저장되는지 알고 있는가?
- [ ] `assert`가 False일 때 어떤 일이 벌어지는지 알고 있는가?
- [ ] 도전 과제 1에서 학생 여러 명을 리스트로 저장하는 방법을 구현할 수 있는가?
