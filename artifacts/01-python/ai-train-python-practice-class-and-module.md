# ai-train Python 실습: class + module 집중 연습

## 대상 챕터

Chapter 14 (class와 object), Chapter 15 (module과 import)

---

## Part 1. class 기초 연습

### 문제 1-1. 도서관 시스템

책을 관리하는 `Book` class와 `Library` class를 만든다.

```python
class Book:
    def __init__(self, title, author, year):
        self.title = title
        self.author = author
        self.year = year
        self.is_available = True   # 대출 가능 여부

    def checkout(self):
        """대출 처리. 이미 대출 중이면 False 반환."""
        # 여기에 코드 작성
        pass

    def return_book(self):
        """반납 처리. 이미 반납된 상태면 False 반환."""
        # 여기에 코드 작성
        pass

    def info(self):
        """책 정보를 문자열로 반환."""
        status = "대출 가능" if self.is_available else "대출 중"
        return f"『{self.title}』 ({self.author}, {self.year}) - {status}"


class Library:
    def __init__(self, name):
        self.name = name
        self.books = []

    def add_book(self, book):
        """도서관에 책 추가."""
        # 여기에 코드 작성
        pass

    def find_by_author(self, author):
        """저자 이름으로 책 검색. 리스트 반환."""
        # 여기에 코드 작성
        pass

    def available_books(self):
        """대출 가능한 책 목록 반환."""
        # 여기에 코드 작성
        pass

    def show_all(self):
        """전체 도서 목록 출력."""
        print(f"\n=== {self.name} 도서 목록 ===")
        for book in self.books:
            print(f"  {book.info()}")


# 테스트
lib = Library("우리 도서관")
lib.add_book(Book("파이썬 입문", "김파이", 2023))
lib.add_book(Book("클린 코드", "로버트", 2020))
lib.add_book(Book("파이썬 심화", "김파이", 2024))

lib.show_all()

# 대출 테스트
book1 = lib.books[0]
print(book1.checkout())    # True
print(book1.checkout())    # False (이미 대출 중)

# 검색 테스트
results = lib.find_by_author("김파이")
for r in results:
    print(r.info())

# 대출 가능 목록
print(f"\n대출 가능: {len(lib.available_books())}권")
```

---

### 문제 1-2. 은행 계좌 고급

이전에 만든 `BankAccount`를 확장해서 거래 내역을 기록하는 기능을 추가한다.

```python
class BankAccount:
    def __init__(self, owner, balance=0):
        self.owner = owner
        self.balance = balance
        self.history = []   # 거래 내역 리스트

    def _record(self, action, amount, after):
        """거래 내역 기록 (내부 메서드)."""
        import datetime
        self.history.append({
            "action": action,
            "amount": amount,
            "after": after,
            "time": datetime.datetime.now().strftime("%H:%M:%S")
        })

    def deposit(self, amount):
        """입금. 금액이 양수여야 한다."""
        # 여기에 코드 작성 (검증 포함, _record 호출)
        pass

    def withdraw(self, amount):
        """출금. 잔액이 충분해야 한다."""
        # 여기에 코드 작성 (검증 포함, _record 호출)
        pass

    def show_history(self):
        """거래 내역 출력."""
        print(f"\n=== {self.owner} 거래 내역 ===")
        for h in self.history:
            print(f"  [{h['time']}] {h['action']} {h['amount']:,}원 → 잔액 {h['after']:,}원")

# 테스트
account = BankAccount("Mina", 100000)
account.deposit(50000)
account.withdraw(30000)
account.deposit(-1000)     # 오류: 음수 입금 불가
account.withdraw(200000)   # 오류: 잔액 부족
account.show_history()
print(f"최종 잔액: {account.balance:,}원")
```

---

## Part 2. module 분리 연습

### 문제 2-1. 도서관 시스템 모듈화

문제 1-1의 도서관 시스템을 아래 구조로 분리한다.

```
library_app/
├── main.py
├── book.py        ← Book class
└── library.py     ← Library class
```

**book.py**:
```python
class Book:
    # 위 Book class 코드
```

**library.py**:
```python
from book import Book   # Book class import

class Library:
    # 위 Library class 코드
```

**main.py**:
```python
from library import Library
from book import Book

# 도서관 운영 코드
if __name__ == "__main__":
    lib = Library("우리 도서관")
    # ... 테스트 코드
```

파일을 분리한 뒤 `python3 main.py`로 실행해서 동일하게 동작하는지 확인한다.

---

### 문제 2-2. 유틸리티 모듈 만들기

여러 프로젝트에서 공통으로 쓸 수 있는 유틸리티 함수를 별도 모듈로 만든다.

**utils.py**:
```python
"""
여러 프로젝트에서 공통으로 쓰는 유틸리티 함수 모음.
"""

def truncate(text, max_length, suffix="..."):
    """긴 텍스트를 max_length로 자르고 suffix를 붙인다."""
    # 여기에 코드 작성
    pass

def format_currency(amount, currency="원"):
    """숫자를 통화 형식으로 반환. 예: 1234567 → '1,234,567원'"""
    # 여기에 코드 작성
    pass

def safe_divide(a, b, default=0):
    """b가 0이면 default를 반환한다."""
    # 여기에 코드 작성
    pass

def flatten(nested_list):
    """중첩 리스트를 1차원으로 펼친다. [[1,2],[3,[4,5]]] → [1,2,3,4,5]"""
    # 여기에 코드 작성
    pass


if __name__ == "__main__":
    # 테스트
    print(truncate("Python은 정말 재미있는 프로그래밍 언어입니다", 20))
    # "Python은 정말 재미있는..."

    print(format_currency(1234567))
    # "1,234,567원"

    print(safe_divide(10, 0))
    # 0

    print(flatten([[1, 2], [3, [4, 5]]]))
    # [1, 2, 3, 4, 5]
```

---

## Part 3. class + module 결합

### 문제 3-1. 학생 관리 시스템 (모듈화 포함)

```
student_app/
├── main.py
├── models/
│   ├── __init__.py
│   └── student.py
└── utils/
    ├── __init__.py
    └── grade_utils.py
```

**grade_utils.py**:
```python
def get_grade(score):
    """점수 → 등급"""
    pass

def calculate_average(scores):
    """점수 리스트 → 평균"""
    pass
```

**student.py**:
```python
from utils.grade_utils import get_grade, calculate_average

class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    def average(self):
        return calculate_average(self.scores)

    def grade(self):
        return get_grade(self.average())

    def report(self):
        return f"{self.name}: {self.average():.1f}점 ({self.grade()})"
```

**main.py**:
```python
from models.student import Student

students = [
    Student("Mina", [90, 85, 92]),
    Student("Tom", [70, 65, 72]),
]

if __name__ == "__main__":
    for s in students:
        print(s.report())
```

---

## 도전 과제

1. `Library.find_by_author` 에 대소문자 무관 검색 기능을 추가한다.
2. `BankAccount`에 월별 입/출금 합계를 계산하는 `monthly_summary()` 메서드를 추가한다.
3. `utils.py`의 `flatten`에서 3단계 이상 중첩된 리스트도 처리하도록 만든다.
