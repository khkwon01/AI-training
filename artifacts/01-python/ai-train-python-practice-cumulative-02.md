# ai-train Python 누적 실습 02: 조건문 + 반복문 + 함수 + class 연결

## 대상 챕터

- Chapter 11: 조건문
- Chapter 12: 반복문
- Chapter 13: 함수
- Chapter 14: class와 object
- Chapter 15: module과 import

---

## 이 실습에서 만드는 것

**학생 성적 관리 프로그램** — 학생 목록을 관리하고, 성적을 계산하고, 결과를 출력하는 간단한 프로그램이다.

완성된 프로그램은 아래 기능을 한다:
1. 학생 추가 (이름 + 점수 목록)
2. 전체 학생 목록 출력
3. 평균 점수와 등급 계산
4. 특정 등급 이상 학생만 필터링
5. 결과를 파일에 저장

---

## Step 1. 함수로 기초 기능 만들기

먼저 학생 없이 성적 계산 함수만 만든다.

```python
# grade_manager.py

def calculate_average(scores):
    """점수 리스트의 평균을 반환한다. 빈 리스트면 0 반환."""
    if not scores:
        return 0
    return sum(scores) / len(scores)


def get_grade(average):
    """평균 점수에 따라 등급을 반환한다."""
    if average >= 90:
        return "A"
    elif average >= 80:
        return "B"
    elif average >= 70:
        return "C"
    elif average >= 60:
        return "D"
    else:
        return "F"


# 테스트
print(calculate_average([90, 85, 92]))   # 89.0
print(get_grade(89))                      # B
print(calculate_average([]))              # 0
```

실행해서 각 함수가 예상대로 동작하는지 확인한다.

---

## Step 2. class로 학생 데이터 묶기

각 학생의 이름과 점수를 class로 관리한다.

```python
class Student:
    def __init__(self, name, scores):
        """
        name: 학생 이름 (문자열)
        scores: 점수 목록 (리스트)
        """
        self.name = name
        self.scores = scores

    def average(self):
        """평균 점수를 반환한다."""
        return calculate_average(self.scores)

    def grade(self):
        """등급을 반환한다."""
        return get_grade(self.average())

    def summary(self):
        """학생 정보를 요약해서 반환한다."""
        avg = self.average()
        return f"{self.name}: 평균 {avg:.1f}점 ({self.grade()})"


# 테스트
s1 = Student("Mina", [90, 85, 92, 88])
s2 = Student("Tom", [70, 65, 72, 68])

print(s1.summary())   # Mina: 평균 88.8점 (B)
print(s2.summary())   # Tom: 평균 68.8점 (D)
```

---

## Step 3. 반복문으로 여러 학생 처리

학생 목록을 만들고 반복문으로 처리한다.

```python
students = [
    Student("Mina", [90, 85, 92, 88]),
    Student("Tom", [70, 65, 72, 68]),
    Student("Jisoo", [95, 98, 91, 97]),
    Student("Alex", [55, 60, 58, 52]),
    Student("Yuna", [82, 79, 85, 88]),
]


def print_all(students):
    """전체 학생 목록을 출력한다."""
    print("\n=== 전체 학생 목록 ===")
    for i, s in enumerate(students, 1):
        print(f"{i}. {s.summary()}")
    print(f"총 {len(students)}명\n")


print_all(students)
```

---

## Step 4. 조건문으로 필터링

특정 등급 이상 학생만 골라낸다.

```python
def filter_by_grade(students, min_grade):
    """
    최소 등급 이상인 학생만 반환한다.
    등급 순서: A > B > C > D > F
    """
    grade_order = ["A", "B", "C", "D", "F"]

    if min_grade not in grade_order:
        print(f"잘못된 등급: {min_grade}")
        return []

    min_index = grade_order.index(min_grade)
    result = []

    for s in students:
        student_index = grade_order.index(s.grade())
        if student_index <= min_index:   # 더 좋은 등급일수록 index가 작음
            result.append(s)

    return result


# B 등급 이상 학생 필터링
passing = filter_by_grade(students, "B")
print("=== B 이상 학생 ===")
for s in passing:
    print(f"  {s.summary()}")
```

---

## Step 5. 파일에 저장하기

결과를 텍스트 파일로 저장한다.

```python
import datetime


def save_report(students, filename="grade_report.txt"):
    """성적 보고서를 파일에 저장한다."""
    today = datetime.date.today()

    with open(filename, "w", encoding="utf-8") as f:
        f.write(f"성적 보고서 ({today})\n")
        f.write("=" * 30 + "\n")

        for s in students:
            f.write(f"{s.summary()}\n")

        f.write("=" * 30 + "\n")
        averages = [s.average() for s in students]
        f.write(f"전체 평균: {sum(averages)/len(averages):.1f}점\n")

    print(f"보고서가 '{filename}' 에 저장됐습니다.")


save_report(students)
```

---

## Step 6. module로 분리하기

코드가 길어졌으니 역할별로 파일을 나눈다.

```
grade_app/
├── main.py
├── student.py      ← Student class
└── grade_utils.py  ← calculate_average, get_grade, filter_by_grade
```

**grade_utils.py**:
```python
def calculate_average(scores):
    if not scores:
        return 0
    return sum(scores) / len(scores)

def get_grade(average):
    if average >= 90: return "A"
    elif average >= 80: return "B"
    elif average >= 70: return "C"
    elif average >= 60: return "D"
    else: return "F"

def filter_by_grade(students, min_grade):
    grade_order = ["A", "B", "C", "D", "F"]
    if min_grade not in grade_order:
        return []
    min_idx = grade_order.index(min_grade)
    return [s for s in students if grade_order.index(s.grade()) <= min_idx]
```

**student.py**:
```python
from grade_utils import calculate_average, get_grade

class Student:
    def __init__(self, name, scores):
        self.name = name
        self.scores = scores

    def average(self):
        return calculate_average(self.scores)

    def grade(self):
        return get_grade(self.average())

    def summary(self):
        return f"{self.name}: 평균 {self.average():.1f}점 ({self.grade()})"
```

**main.py**:
```python
from student import Student
from grade_utils import filter_by_grade
import datetime

students = [
    Student("Mina", [90, 85, 92, 88]),
    Student("Tom", [70, 65, 72, 68]),
    Student("Jisoo", [95, 98, 91, 97]),
    Student("Alex", [55, 60, 58, 52]),
    Student("Yuna", [82, 79, 85, 88]),
]

if __name__ == "__main__":
    print("=== 전체 학생 ===")
    for i, s in enumerate(students, 1):
        print(f"{i}. {s.summary()}")

    print("\n=== B 이상 학생 ===")
    for s in filter_by_grade(students, "B"):
        print(f"  {s.summary()}")
```

---

## 확인 질문

1. `Student` class에서 `self`가 없으면 `average()` 메서드는 어떻게 달라질까?
2. `filter_by_grade`에서 `grade_order.index(s.grade()) <= min_idx` 조건이 의미하는 것은?
3. `main.py`에 `if __name__ == "__main__":` 가 없으면 `student.py`에서 import할 때 어떤 일이 생길까?
4. `calculate_average([])` 가 `0`을 반환하는 대신 예외를 발생시키면 어떤 장단점이 있을까?

---

## 도전 과제

1. 학생을 평균 점수 내림차순으로 정렬해서 출력하는 함수 `rank_students(students)` 를 추가하자.
2. 점수가 0~100 범위를 벗어나면 경고를 출력하는 검증 로직을 `Student.__init__` 에 추가하자.
3. 보고서 파일을 JSON 형식으로도 저장하는 `save_json_report(students)` 함수를 만들자.
