# ai-train Python Chapter 14: class와 object

## 이 장에서 배우는 것

- class가 무엇인지, 왜 필요한지
- class를 만들고 object를 생성하는 방법
- `__init__`으로 초기 데이터를 설정하는 방법
- 메서드(method)를 만들고 사용하는 방법
- `self`가 무엇인지 이해하기

---

## 먼저 쉬운 설명

학생 정보를 저장한다고 생각해보자.

```python
# 변수로 관리하면 학생이 늘수록 복잡해진다
name1 = "Mina"
age1 = 14
name2 = "Tom"
age2 = 15
```

class를 쓰면 학생 정보를 하나로 묶을 수 있다.

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age

mina = Student("Mina", 14)
tom = Student("Tom", 15)
```

**class = 설계도**, **object = 설계도로 만든 실제 물건**

Student class는 "학생은 이름과 나이를 갖는다"는 설계도다. `mina`와 `tom`은 그 설계도로 만든 실제 학생 object다.

---

## 1. class 만들기

```python
class Student:
    pass   # 내용이 없는 class
```

`pass`는 "아무것도 하지 않는다"는 의미다. 나중에 내용을 채울 자리를 잡아두는 용도로 쓴다.

---

## 2. `__init__`: 초기화 메서드

object를 만들 때 자동으로 실행되는 특별한 메서드다.

```python
class Student:
    def __init__(self, name, age):
        self.name = name    # name 속성 저장
        self.age = age      # age 속성 저장

mina = Student("Mina", 14)
print(mina.name)   # Mina
print(mina.age)    # 14
```

- `self`는 "이 object 자신"을 가리킨다
- `self.name`은 "이 object의 name 속성"이다

---

## 3. 메서드 만들기

class 안에 정의한 함수를 **메서드(method)**라고 한다.

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def introduce(self):
        print(f"안녕하세요! 저는 {self.name}이고 {self.age}살입니다.")

    def is_adult(self):
        return self.age >= 18

mina = Student("Mina", 14)
mina.introduce()           # 안녕하세요! 저는 Mina이고 14살입니다.
print(mina.is_adult())     # False
```

메서드의 첫 번째 매개변수는 반드시 `self`여야 한다.

---

## 4. 여러 object 만들기

같은 class로 여러 object를 만들 수 있다. 각각은 독립적인 데이터를 가진다.

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score

    def grade(self):
        if self.score >= 90:
            return "A"
        elif self.score >= 80:
            return "B"
        else:
            return "C"

students = [
    Student("Mina", 95),
    Student("Tom", 82),
    Student("Jisoo", 71),
]

for s in students:
    print(f"{s.name}: {s.grade()}")
```

출력:
```
Mina: A
Tom: B
Jisoo: C
```

---

## 5. 속성 변경하기

object의 속성은 언제든 변경할 수 있다.

```python
mina = Student("Mina", 14)
print(mina.age)   # 14

mina.age = 15
print(mina.age)   # 15
```

---

## 6. 앞 장 메모 서비스를 class로 리팩토링

```python
class MemoService:
    def __init__(self):
        self.memos = []

    def add(self, text):
        if text.strip():
            self.memos.append(text.strip())
            print(f"✓ 추가됨: {text}")

    def show(self):
        if not self.memos:
            print("메모가 없습니다.")
            return
        for i, memo in enumerate(self.memos, 1):
            print(f"{i}. {memo}")

    def delete(self, number):
        if 1 <= number <= len(self.memos):
            removed = self.memos.pop(number - 1)
            print(f"✓ 삭제됨: {removed}")
        else:
            print("잘못된 번호입니다.")

service = MemoService()
service.add("Python 공부하기")
service.add("GitHub 연습하기")
service.show()
service.delete(1)
service.show()
```

---

## 7. 따라 하기 실습

### 실습 1. 도서 class 만들기

`library.py` 파일을 만들고:

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages

    def info(self):
        print(f"제목: {self.title}")
        print(f"저자: {self.author}")
        print(f"페이지: {self.pages}쪽")

    def is_long(self):
        return self.pages > 300

book1 = Book("파이썬 입문", "김파이", 250)
book2 = Book("클린 코드", "로버트", 464)

book1.info()
print(f"긴 책인가: {book1.is_long()}")
```

### 실습 2. 은행 계좌 class

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount
        print(f"{amount}원 입금. 잔액: {self.balance}원")

    def withdraw(self, amount):
        if amount > self.balance:
            print("잔액 부족")
        else:
            self.balance -= amount
            print(f"{amount}원 출금. 잔액: {self.balance}원")

account = BankAccount("Mina", 10000)
account.deposit(5000)
account.withdraw(3000)
account.withdraw(20000)
```

---

## 자주 하는 실수

| 실수 | 증상 | 해결 방법 |
|------|------|----------|
| 메서드에서 `self` 빠짐 | `TypeError: takes 0 positional arguments but 1 was given` | 모든 메서드 첫 번째 매개변수에 `self` 추가 |
| class 만들고 object 생성 안 함 | 아무것도 실행 안 됨 | `mina = Student(...)` 처럼 object 생성 |
| `self.name` 대신 `name` 사용 | 다른 메서드에서 해당 값 접근 불가 | 인스턴스 속성은 반드시 `self.` 접두어 사용 |
| `__init__` 철자 오류 | 초기화가 실행 안 됨 | 앞뒤 언더스코어 2개씩 확인 (`__init__`) |

---

## 확인 체크리스트

- [ ] class와 object의 차이를 설명할 수 있는가
- [ ] `__init__`이 언제 실행되는지 말할 수 있는가
- [ ] `self`가 무엇을 가리키는지 설명할 수 있는가
- [ ] 간단한 class를 만들고 object를 생성해 메서드를 호출할 수 있는가

---

## 한 번 더 생각해 보기

1. 함수와 메서드는 어떻게 다른가?
2. `self`가 없으면 메서드들이 서로의 데이터를 어떻게 공유할까?
3. 같은 class로 만든 두 object가 독립적이라는 것이 왜 중요한가?

---

## 다음 장

다음 장에서는 module과 import를 배운다. 코드가 길어지면 여러 파일로 나눠야 하는데, 다른 파일의 코드를 가져다 쓰는 방법을 익힌다.

---

## 참고 자료

- Python Tutorial: Classes — https://docs.python.org/3/tutorial/classes.html
