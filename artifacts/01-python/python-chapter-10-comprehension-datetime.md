# Chapter 10: 컴프리헨션과 날짜/시간 (Comprehension & datetime)

## 이 장에서 배우는 것

- 리스트 컴프리헨션이 왜 필요하고 어떻게 만드는지 (5단계 변환)
- 조건부 컴프리헨션으로 필터링하기
- 딕셔너리 컴프리헨션과 셋 컴프리헨션
- `datetime`으로 날짜 생성, 차이 계산, 포맷 변환, 타임존 기초 다루기
- 실습 3개: 성적 필터링, D-day 계산, 날짜 데이터 가공

---

## 왜 컴프리헨션이 필요한가

Python에서 리스트를 만들 때 가장 기본적인 방법은 `for` 반복문과 `append()`를 쓰는 것이다. 그런데 이 패턴을 아주 자주 쓰다 보니 Python은 이것을 한 줄로 쓸 수 있는 문법을 만들었다. 이것이 **컴프리헨션(Comprehension)**이다.

컴프리헨션을 쓰면:
- 코드가 짧아진다
- 읽고 나면 의도가 명확하다
- 실행 속도도 약간 빠르다

단, 너무 복잡하게 쓰면 오히려 읽기 어려워진다. 간단한 경우에만 쓰는 것이 원칙이다.

---

## 1. 리스트 컴프리헨션: 5단계 변환

### 1단계: 기본 for 반복문

```python
numbers = [1, 2, 3, 4, 5]
result = []

for number in numbers:
    result.append(number * 2)

print(result)  # [2, 4, 6, 8, 10]
```

### 2단계: 패턴 파악

`append(number * 2)` 부분이 핵심 변환이다. 이 패턴을 컴프리헨션으로 옮긴다.

### 3단계: 기본 컴프리헨션 구조

```
[표현식 for 변수 in 반복대상]
```

### 4단계: 코드 변환

```python
numbers = [1, 2, 3, 4, 5]
result = [number * 2 for number in numbers]

print(result)  # [2, 4, 6, 8, 10]
```

### 5단계: 다양한 예시 비교

```python
# 예시 1: 문자열 리스트를 대문자로
names = ["alice", "bob", "charlie"]

# for 반복문
upper_names = []
for name in names:
    upper_names.append(name.upper())

# 컴프리헨션
upper_names = [name.upper() for name in names]
# 결과: ['ALICE', 'BOB', 'CHARLIE']

# 예시 2: 숫자 제곱
squares = [x ** 2 for x in range(1, 6)]
# 결과: [1, 4, 9, 16, 25]

# 예시 3: 문자열 길이
words = ["apple", "hi", "banana", "ok"]
lengths = [len(word) for word in words]
# 결과: [5, 2, 6, 2]
```

---

## 2. 조건부 컴프리헨션: 필터링

`if` 조건을 추가하면 특정 조건을 만족하는 요소만 결과에 포함시킬 수 있다.

### 기본 구조

```
[표현식 for 변수 in 반복대상 if 조건]
```

### 예시

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 짝수만 가져오기
even_numbers = [n for n in numbers if n % 2 == 0]
print(even_numbers)  # [2, 4, 6, 8, 10]

# 5보다 큰 수만 가져오기
big_numbers = [n for n in numbers if n > 5]
print(big_numbers)  # [6, 7, 8, 9, 10]

# 문자열에서 특정 조건 필터링
words = ["apple", "hi", "banana", "ok", "cherry"]
long_words = [w for w in words if len(w) >= 5]
print(long_words)  # ['apple', 'banana', 'cherry']
```

### 조건부 변환 (삼항 표현식과 함께)

변환 결과를 조건에 따라 다르게 만들 수도 있다.

```
[참일_때_값 if 조건 else 거짓일_때_값 for 변수 in 반복대상]
```

```python
scores = [85, 42, 91, 60, 73]

# 60점 이상이면 "합격", 아니면 "불합격"
results = ["합격" if score >= 60 else "불합격" for score in scores]
print(results)  # ['합격', '불합격', '합격', '합격', '합격']
```

---

## 3. 딕셔너리 컴프리헨션

리스트 대신 딕셔너리를 만드는 컴프리헨션이다. `{}` 와 `키: 값` 형식을 쓴다.

```
{키_표현식: 값_표현식 for 변수 in 반복대상}
```

```python
# 예시 1: 리스트에서 딕셔너리 만들기
names = ["김철수", "이영희", "박민준"]
scores = [85, 92, 78]

# zip()으로 두 리스트를 묶어서 딕셔너리 생성
score_dict = {name: score for name, score in zip(names, scores)}
print(score_dict)
# {'김철수': 85, '이영희': 92, '박민준': 78}

# 예시 2: 키를 가공해서 만들기
words = ["Hello", "World", "Python"]
word_lengths = {word: len(word) for word in words}
print(word_lengths)
# {'Hello': 5, 'World': 5, 'Python': 6}

# 예시 3: 조건 포함
students = {"김철수": 85, "이영희": 92, "박민준": 55, "최수아": 70}
# 60점 이상만 포함
passing = {name: score for name, score in students.items() if score >= 60}
print(passing)
# {'김철수': 85, '이영희': 92, '최수아': 70}
```

---

## 4. 셋 컴프리헨션

중복을 자동으로 제거하는 `set`을 컴프리헨션으로 만들 수 있다.

```python
numbers = [1, 2, 2, 3, 3, 3, 4]

# 중복 제거한 셋
unique = {n for n in numbers}
print(unique)  # {1, 2, 3, 4} (순서는 보장되지 않음)

# 예시: 이메일 도메인만 추출 (중복 없이)
emails = ["user1@gmail.com", "user2@naver.com", "user3@gmail.com", "user4@kakao.com"]
domains = {email.split("@")[1] for email in emails}
print(domains)  # {'gmail.com', 'naver.com', 'kakao.com'}
```

---

## 5. 컴프리헨션을 쓸 때 주의할 점

```python
# 이렇게 복잡하게 쓰면 안 된다 - 읽기 어렵다
result = [x * y for x in range(1, 4) for y in range(1, 4) if x != y]

# 이 경우에는 그냥 for 반복문이 낫다
result = []
for x in range(1, 4):
    for y in range(1, 4):
        if x != y:
            result.append(x * y)
```

기준: 한 줄로 읽었을 때 바로 이해되면 컴프리헨션을 쓴다. 그렇지 않으면 반복문을 쓴다.

---

## 6. datetime: 날짜와 시간 다루기

### 왜 datetime이 필요한가

날짜와 시간은 생각보다 복잡하다.

- "2024-02-29"는 윤년에만 존재하는 날짜다
- "2024-01-15"에서 30일 후는 "2024-02-14"다 (2월이 28일이나 29일이라서)
- 한국 시간과 미국 시간은 다르다

이런 계산을 직접 하면 실수가 많다. `datetime` 모듈이 이런 복잡한 계산을 대신 해준다.

### import 방법

```python
from datetime import datetime, date, timedelta, timezone
```

---

## 7. 날짜 생성하기

```python
from datetime import datetime, date

# 현재 날짜와 시간
now = datetime.now()
print(now)  # 2026-05-19 14:30:25.123456

# 현재 날짜만 (시간 없음)
today = date.today()
print(today)  # 2026-05-19

# 특정 날짜 만들기
birthday = datetime(1990, 3, 15)
print(birthday)  # 1990-03-15 00:00:00

# 특정 날짜와 시간
event = datetime(2026, 12, 25, 9, 0, 0)
print(event)  # 2026-12-25 09:00:00

# 속성 접근
print(now.year)    # 2026
print(now.month)   # 5
print(now.day)     # 19
print(now.hour)    # 14
print(now.weekday())  # 0=월, 1=화, ..., 6=일
```

---

## 8. 날짜 차이 계산하기 (timedelta)

`timedelta`는 날짜 간의 간격을 나타낸다. 날짜를 더하거나 빼는 데 사용한다.

```python
from datetime import datetime, timedelta

now = datetime.now()

# 특정 기간 후 날짜
tomorrow = now + timedelta(days=1)
next_week = now + timedelta(weeks=1)
three_hours_later = now + timedelta(hours=3)

print(f"내일: {tomorrow.strftime('%Y-%m-%d')}")
print(f"다음 주: {next_week.strftime('%Y-%m-%d')}")

# 두 날짜 사이의 차이
birthday = datetime(1990, 3, 15)
days_lived = now - birthday
print(f"태어난 지 {days_lived.days}일이 지났습니다.")

# D-day 계산
target = datetime(2026, 12, 31)
diff = target - now
print(f"올해 마지막 날까지 {diff.days}일 남았습니다.")
```

---

## 9. 날짜 포맷 변환

### datetime → 문자열 (strftime)

`strftime()`은 날짜를 원하는 형식의 문자열로 변환한다.

```python
from datetime import datetime

now = datetime.now()

# 주요 포맷 코드
print(now.strftime("%Y-%m-%d"))        # 2026-05-19
print(now.strftime("%Y년 %m월 %d일"))  # 2026년 05월 19일
print(now.strftime("%H:%M:%S"))        # 14:30:25
print(now.strftime("%Y/%m/%d %H:%M")) # 2026/05/19 14:30
print(now.strftime("%A"))              # Tuesday (영어 요일)
```

| 코드 | 의미 | 예시 |
|------|------|------|
| `%Y` | 4자리 연도 | 2026 |
| `%m` | 2자리 월 | 05 |
| `%d` | 2자리 일 | 19 |
| `%H` | 24시간 | 14 |
| `%M` | 분 | 30 |
| `%S` | 초 | 25 |
| `%A` | 영어 요일 이름 | Tuesday |

### 문자열 → datetime (strptime)

문자열로 된 날짜를 `datetime` 객체로 변환할 때 사용한다.

```python
from datetime import datetime

# 문자열 → datetime
date_str = "2026-05-19"
parsed_date = datetime.strptime(date_str, "%Y-%m-%d")
print(type(parsed_date))  # <class 'datetime.datetime'>
print(parsed_date.year)   # 2026

# 다양한 형식
date_str2 = "19/05/2026 14:30"
parsed_date2 = datetime.strptime(date_str2, "%d/%m/%Y %H:%M")
```

---

## 10. 타임존 기초

### 왜 타임존이 필요한가

`datetime.now()`가 반환하는 시간은 컴퓨터가 설정된 지역 시간이다. 한국에서 실행하면 KST(UTC+9)다. 서버가 미국에 있거나 국제 서비스를 만들면 타임존을 명시해야 한다.

```python
from datetime import datetime, timezone, timedelta

# UTC 시간 가져오기
utc_now = datetime.now(timezone.utc)
print(f"UTC 시간: {utc_now.strftime('%Y-%m-%d %H:%M:%S %Z')}")

# 한국 시간 (UTC+9)
KST = timezone(timedelta(hours=9))
kst_now = datetime.now(KST)
print(f"한국 시간: {kst_now.strftime('%Y-%m-%d %H:%M:%S %Z')}")

# UTC 시간을 한국 시간으로 변환
korea_time = utc_now.astimezone(KST)
print(f"UTC→KST 변환: {korea_time.strftime('%Y-%m-%d %H:%M:%S')}")
```

---

## 실습 1. 성적 필터링 (컴프리헨션)

### 따라 하기

학생 성적 데이터를 컴프리헨션으로 다양하게 처리해 보자.

```python
students = [
    {"name": "김철수", "score": 85, "subject": "수학"},
    {"name": "이영희", "score": 92, "subject": "영어"},
    {"name": "박민준", "score": 55, "subject": "수학"},
    {"name": "최수아", "score": 70, "subject": "영어"},
    {"name": "정태양", "score": 88, "subject": "수학"},
    {"name": "강지수", "score": 45, "subject": "영어"},
]

# 1. 이름만 추출
names = [s["name"] for s in students]
print("전체 학생:", names)

# 2. 60점 이상인 학생 이름만
passing_names = [s["name"] for s in students if s["score"] >= 60]
print("합격자:", passing_names)

# 3. 점수를 "합격"/"불합격"으로 변환
results = [
    {"name": s["name"], "result": "합격" if s["score"] >= 60 else "불합격"}
    for s in students
]
for r in results:
    print(f"  {r['name']}: {r['result']}")

# 4. 수학 과목 학생만 딕셔너리로
math_scores = {s["name"]: s["score"] for s in students if s["subject"] == "수학"}
print("\n수학 성적:", math_scores)

# 5. 80점 이상 학생의 점수 목록
high_scores = [s["score"] for s in students if s["score"] >= 80]
print(f"\n80점 이상 점수: {high_scores}")
print(f"평균: {sum(high_scores) / len(high_scores):.1f}")
```

### 직접 해보기

1. 영어 과목에서 70점 이상인 학생의 이름과 점수를 딕셔너리로 만들어 보자
2. 전체 학생의 점수를 10점씩 올려서 새 리스트를 만들되, 100점을 넘으면 100으로 고정해 보자

---

## 실습 2. D-day 계산기 (datetime)

### 따라 하기

오늘 날짜를 기준으로 다양한 날짜를 계산하는 프로그램을 만들어 보자.

```python
from datetime import datetime, timedelta

def calculate_dday(target_date_str, event_name):
    """D-day를 계산하는 함수"""
    target = datetime.strptime(target_date_str, "%Y-%m-%d")
    today = datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)
    diff = (target - today).days

    if diff > 0:
        return f"{event_name}: D-{diff}"
    elif diff == 0:
        return f"{event_name}: D-Day! (오늘!)"
    else:
        return f"{event_name}: D+{abs(diff)} (이미 {abs(diff)}일 지남)"

# 오늘 날짜 출력
today = datetime.now()
print(f"오늘: {today.strftime('%Y년 %m월 %d일 (%A)')}\n")

# D-day 목록
events = [
    ("2026-12-25", "크리스마스"),
    ("2026-12-31", "새해 전날"),
    ("2027-01-01", "새해"),
]

for date_str, name in events:
    print(calculate_dday(date_str, name))

# 며칠 후/전 날짜 계산
print("\n--- 날짜 계산 ---")
print(f"100일 후: {(today + timedelta(days=100)).strftime('%Y-%m-%d')}")
print(f"30일 전: {(today - timedelta(days=30)).strftime('%Y-%m-%d')}")

# 생일까지 남은 날 계산
def days_until_birthday(month, day):
    today = datetime.now()
    birthday_this_year = datetime(today.year, month, day)
    if birthday_this_year < today:
        birthday_this_year = datetime(today.year + 1, month, day)
    return (birthday_this_year - today).days

days_left = days_until_birthday(3, 15)
print(f"\n3월 15일 생일까지: {days_left}일 남음")
```

### 직접 해보기

1. 사용자에게 날짜를 입력받아 (`input()`) D-day를 계산하는 프로그램으로 수정해 보자
2. 태어난 날짜를 입력받아 살아온 날 수, 시간 수를 계산해 보자

---

## 실습 3. 날짜 데이터 가공하기 (컴프리헨션 + datetime 통합)

### 따라 하기

문자열 형태의 날짜 데이터를 컴프리헨션으로 가공해 보자.

```python
from datetime import datetime, timedelta

# 주문 데이터 (문자열 날짜)
orders = [
    {"id": 1, "date": "2026-05-01", "amount": 15000},
    {"id": 2, "date": "2026-05-10", "amount": 32000},
    {"id": 3, "date": "2026-05-15", "amount": 8000},
    {"id": 4, "date": "2026-04-28", "amount": 45000},
    {"id": 5, "date": "2026-05-18", "amount": 12000},
]

today = datetime(2026, 5, 19)
thirty_days_ago = today - timedelta(days=30)

# 문자열 날짜를 datetime으로 변환
parsed_orders = [
    {**order, "date": datetime.strptime(order["date"], "%Y-%m-%d")}
    for order in orders
]

# 30일 이내 주문만 필터링
recent_orders = [o for o in parsed_orders if o["date"] >= thirty_days_ago]
print(f"최근 30일 주문 수: {len(recent_orders)}")

# 날짜 포맷을 "5월 1일" 형식으로 변환해서 출력
formatted = [
    f"주문 {o['id']}: {o['date'].strftime('%m월 %d일')} - {o['amount']:,}원"
    for o in recent_orders
]
for line in formatted:
    print(line)

# 최근 30일 총 금액
total = sum(o["amount"] for o in recent_orders)
print(f"\n최근 30일 총 매출: {total:,}원")
```

### 직접 해보기

1. 금액이 10,000원 이상인 주문만 필터링해서 출력해 보자
2. 날짜를 "며칠 전" 형식으로 바꿔서 출력해 보자 (예: "4일 전")

---

## 초보자가 자주 막히는 지점

### 막힘 1: strftime과 strptime 혼동

- `strftime`: datetime **→** 문자열 ("format **to** string")
- `strptime`: 문자열 **→** datetime ("**p**arse from string")

```python
# 날짜를 문자열로 → strftime
now = datetime.now()
text = now.strftime("%Y-%m-%d")   # "2026-05-19"

# 문자열을 날짜로 → strptime
text = "2026-05-19"
date = datetime.strptime(text, "%Y-%m-%d")
```

### 막힘 2: timedelta에서 일수가 음수로 나올 때

두 날짜의 차이를 계산할 때 순서가 바뀌면 음수가 나온다.

```python
past = datetime(2020, 1, 1)
future = datetime(2026, 5, 19)

diff1 = future - past    # 양수 (미래 - 과거)
diff2 = past - future    # 음수 (과거 - 미래)

print(diff1.days)   # 2330 (양수)
print(diff2.days)   # -2330 (음수)

# 절댓값으로 처리
print(abs(diff2.days))  # 2330
```

### 막힘 3: `datetime` import 방법

```python
# 방법 1: 모듈 import (매번 datetime.datetime으로 써야 함)
import datetime
now = datetime.datetime.now()

# 방법 2: 클래스 직접 import (권장)
from datetime import datetime
now = datetime.now()

# 방법 3: 여러 개 import
from datetime import datetime, timedelta, date
```

### 막힘 4: 컴프리헨션에서 조건의 위치

```python
numbers = [1, 2, 3, 4, 5]

# 필터링 조건은 for 뒤에 붙는다
even = [n for n in numbers if n % 2 == 0]

# 변환 조건(삼항)은 앞에 붙는다
labeled = ["짝수" if n % 2 == 0 else "홀수" for n in numbers]
```

---

## 자주 만나는 에러와 해결법

| 에러 메시지 | 원인 | 해결 방법 |
|-------------|------|-----------|
| `NameError: name 'datetime' is not defined` | import 안 함 | `from datetime import datetime` 추가 |
| `ValueError: time data '2026-05-19' does not match format '%d/%m/%Y'` | 날짜 형식이 맞지 않음 | 형식 문자열을 날짜에 맞게 수정 |
| `TypeError: '<' not supported between instances of 'str' and 'datetime.datetime'` | 문자열과 datetime을 비교 | `strptime()`으로 변환 후 비교 |

---

## 확인 체크리스트

- for 반복문을 리스트 컴프리헨션으로 바꿀 수 있는가
- 컴프리헨션에 `if` 조건을 추가해서 필터링할 수 있는가
- 딕셔너리 컴프리헨션으로 두 리스트를 묶어서 딕셔너리를 만들 수 있는가
- `datetime.now()`로 현재 시간을 가져올 수 있는가
- `strftime()`으로 날짜를 원하는 형식의 문자열로 변환할 수 있는가
- `strptime()`으로 문자열을 `datetime`으로 변환할 수 있는가
- `timedelta`로 날짜를 더하거나 빼서 D-day를 계산할 수 있는가

---

## 한 번 더 생각해 보기

1. 컴프리헨션이 더 읽기 어려워지는 경우는 어떤 때일까?
2. `timedelta`가 없다면 날짜 차이를 어떻게 계산해야 할까?
3. 한국 서비스를 만들 때 타임존을 명시해야 하는 경우는 어떤 때일까?
4. 딕셔너리 컴프리헨션과 셋 컴프리헨션은 각각 어떤 상황에서 유용할까?
