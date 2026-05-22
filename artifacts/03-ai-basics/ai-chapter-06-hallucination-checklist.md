# Chapter 06: AI 환각(Hallucination) 감지 체크리스트

## 이 장에서 배우는 것

- AI 환각이 정확히 무엇인지 (있지도 않은 함수, 구버전 API 등)
- Python 코드 기준 환각 유형 5가지와 실제 예시
- 환각을 탐지하는 구체적인 방법 (함수명 검색, 공식 문서 대조, 실제 실행)
- assert로 코드를 검증하는 방법
- AI가 준 코드를 검증하는 3단계 체크리스트
- 실습: 의도적으로 환각 코드 만들어보기 → 탐지 → 수정

---

## 왜 필요한가 — AI는 모르는 것도 자신있게 말한다

AI에게 코드를 받았는데 실행하니 `AttributeError`가 난 경험이 있는가?

```
AttributeError: 'DataFrame' object has no attribute 'remove_duplicates'
```

AI가 "remove_duplicates()를 사용하면 됩니다"라고 확신에 찬 어조로 말했는데, 실제로 그런 메서드는 존재하지 않는다. pandas에서 중복을 제거하는 메서드는 `drop_duplicates()`다.

이것이 **환각(hallucination)**이다.

AI는 없는 것도 있다고, 틀린 것도 맞다고 말한다. 그것도 매우 자신있게. 어조가 자신있다고 해서 맞는 게 아니다.

### 환각이 위험한 이유

**위험 1: 코드가 실행되지만 결과가 틀린 경우**

이게 가장 위험하다. 오류가 나면 적어도 "뭔가 잘못됐다"는 걸 안다. 하지만 코드가 실행되면서 틀린 결과를 낸다면 모른다.

예: AI가 만든 할인율 계산 함수가 10% 할인을 적용하는 대신 10% 증가를 적용하고 있는데, 직접 계산해서 확인하지 않으면 모른다.

**위험 2: 보안 취약점을 포함한 코드**

AI가 "이 방법으로 구현하면 됩니다"라고 준 코드에 SQL injection 취약점이 있을 수 있다.

**위험 3: 오래된 API 사용**

AI의 학습 데이터에는 최신 라이브러리 버전이 없을 수 있다. 현재는 deprecated된 방식을 사용하는 코드를 줄 수 있다.

### 정리

AI가 준 코드를 무조건 믿으면 안 된다. 항상 검증해야 한다.

---

## 1. 환각 유형 5가지 (Python 코드 기준)

### 유형 1. 존재하지 않는 메서드/함수

AI가 실제로 없는 메서드를 만들어서 쓴다.

```python
# AI가 준 코드
import pandas as pd

df = pd.read_csv("data.csv")
df.remove_duplicates()   # 이 메서드는 존재하지 않는다

# 실제 올바른 코드
df.drop_duplicates()     # 정확한 메서드 이름
```

```python
# AI가 준 코드
my_list = [3, 1, 4, 1, 5]
my_list.sort_descending()   # 이 메서드는 존재하지 않는다

# 실제 올바른 코드
my_list.sort(reverse=True)  # 정확한 사용 방법
```

**왜 생기는가:** AI가 메서드 이름을 "그럴 것 같다"는 방향으로 만들어낸다. `remove_duplicates`는 "중복 제거"라는 의미로 직관적이지만 실제 pandas API는 다르다.

**탐지 방법:** 터미널에서 `python -c "help(pd.DataFrame)"` 또는 공식 문서 검색.

---

### 유형 2. 잘못된 인자 형태

실제 함수는 존재하지만 인자 순서나 이름이 틀린 경우.

```python
# AI가 준 코드
import datetime

# 잘못된 키워드 인자 순서 가정
d = datetime.date(day=15, month=5, year=2026)

# 실제 올바른 코드
d = datetime.date(2026, 5, 15)  # year, month, day 순서
# 또는
d = datetime.date(year=2026, month=5, day=15)  # 키워드 인자도 가능
```

```python
# AI가 준 코드
import re

# 잘못된 flags 위치
pattern = re.compile("hello", "world", re.IGNORECASE)  # 두 번째 인자는 없음

# 실제 올바른 코드
pattern = re.compile("hello", re.IGNORECASE)  # 첫 번째: 패턴, 두 번째: flags
```

**왜 생기는가:** AI가 여러 함수의 인터페이스를 혼동하거나, 실제 API보다 "자연스러운" 형태로 재구성한다.

**탐지 방법:** 직접 실행, `help(함수명)`, 공식 문서의 함수 시그니처 확인.

---

### 유형 3. 존재하지 않는 라이브러리 기능

라이브러리 자체는 존재하지만 AI가 만들어낸 기능.

```python
# AI가 준 코드
import requests

# 이런 모듈이나 함수는 requests에 없다
from requests import smart_retry
response = smart_retry.get("https://api.example.com", max_retries=3)

# 실제로 재시도를 구현하려면
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retry = Retry(total=3)
adapter = HTTPAdapter(max_retries=retry)
session.mount("https://", adapter)
response = session.get("https://api.example.com")
```

```python
# AI가 준 코드
import json

# json 모듈에는 이런 함수가 없다
data = json.parse('{"name": "Mina"}')   # parse는 JavaScript, Python은 loads

# 실제 올바른 코드
data = json.loads('{"name": "Mina"}')  # loads (load string)
```

**왜 생기는가:** 다른 언어(JavaScript, Java 등)의 API와 혼동하거나, 학습 데이터에서 비공식 코드를 학습했을 가능성.

**탐지 방법:** `pip show 라이브러리명`으로 설치 확인, 공식 문서 검색.

---

### 유형 4. 동작하지만 결과가 틀린 코드

실행은 되지만 의도한 결과가 아닌 경우. 가장 위험하다.

```python
# AI가 준 코드 (설명: "문자열이 회문인지 확인한다")
def is_palindrome(text):
    return text == text[::-1]

# 테스트
print(is_palindrome("racecar"))             # True  ← 맞음
print(is_palindrome("A man a plan a canal Panama"))  # False ← 틀림 (실제로는 회문)
```

위 함수는 공백과 대소문자를 처리하지 않는다. "A man a plan a canal Panama"는 공백과 대소문자를 무시하면 회문인데 False를 반환한다.

```python
# 올바른 코드
def is_palindrome(text):
    # 소문자 변환, 알파벳/숫자만 남기기
    cleaned = "".join(c.lower() for c in text if c.isalnum())
    return cleaned == cleaned[::-1]

print(is_palindrome("A man a plan a canal Panama"))  # True
```

```python
# AI가 준 코드 (설명: "리스트의 두 번째 항목부터 반환")
def skip_first(items):
    return items[1:]

# 정상 케이스는 동작
skip_first([1, 2, 3])  # [2, 3] ← 맞음

# 빈 리스트는?
skip_first([])   # [] 반환 (오류 없이 조용히 처리 - 의도한 것인지 확인 필요)
# 한 개짜리 리스트는?
skip_first([1])  # [] 반환 - 의도한 것인지?
```

**왜 생기는가:** AI가 명확히 지정하지 않은 엣지 케이스를 다루지 않거나, 요구사항을 일부만 이해한 경우.

**탐지 방법:** 다양한 입력으로 직접 테스트. 특히 빈 값, 경계값, 예상치 못한 타입으로 테스트.

---

### 유형 5. deprecated된 오래된 API 사용

AI가 학습한 시점 이후에 deprecated된 방식을 여전히 쓰는 경우.

```python
# AI가 준 코드 (구버전 방식)
import boto3

# 오래된 방식 (일부 버전에서 동작 안 할 수 있음)
s3 = boto3.resource('s3')
s3.Object('my-bucket', 'file.txt').put(Body='content')

# 현재 권장 방식
s3_client = boto3.client('s3')
s3_client.put_object(Bucket='my-bucket', Key='file.txt', Body='content')
```

```python
# AI가 준 코드 (구버전 Python 방식)
# Python 2 스타일 print 문
print "Hello, World!"   # Python 3에서 SyntaxError

# 올바른 Python 3 코드
print("Hello, World!")
```

```python
# AI가 준 코드 (오래된 requests 방식)
import requests

# 과거 일부 버전에서 쓰던 방식
response = requests.get(url, verify=False)  # SSL 검증 비활성화 → 보안 취약점

# 올바른 방식
response = requests.get(url)  # 기본적으로 SSL 검증 활성화
```

**왜 생기는가:** AI 학습 데이터의 컷오프(cutoff) 이후에 API가 변경됐거나, 인터넷에 deprecated 예제가 많이 남아있기 때문.

**탐지 방법:** 공식 문서의 "Migration Guide" 또는 "Deprecated" 섹션 확인. 라이브러리 버전 확인(`pip show 패키지명`).

---

## 2. 환각을 탐지하는 3가지 방법

### 방법 1. 함수명/메서드명 검색

AI가 준 코드에서 처음 보는 함수나 메서드가 있으면 바로 확인한다.

**Python REPL에서 확인:**

```python
# 해당 라이브러리를 import하고
import pandas as pd

# dir()로 사용 가능한 메서드 목록 확인
print([m for m in dir(pd.DataFrame) if 'dupl' in m.lower()])
# 출력: ['drop_duplicates']  ← remove_duplicates는 없음
```

**help()로 함수 시그니처 확인:**

```python
import pandas as pd
df = pd.DataFrame()
help(df.drop_duplicates)  # 실제 메서드의 인자와 설명 확인
```

**터미널에서 확인:**

```bash
# requests 패키지에 smart_retry가 있는지 확인
python -c "import requests; print(dir(requests))"
# smart_retry가 목록에 없으면 존재하지 않는 것
```

### 방법 2. 공식 문서 대조

공식 문서가 유일한 정답이다. AI는 틀릴 수 있지만 공식 문서는 해당 버전의 정확한 정보를 담고 있다.

주요 공식 문서:
- Python 표준 라이브러리: https://docs.python.org/3/library/
- pandas: https://pandas.pydata.org/docs/reference/
- requests: https://requests.readthedocs.io/
- boto3 (AWS SDK): https://boto3.amazonaws.com/v1/documentation/api/latest/

문서 확인 방법:
1. 함수/메서드 이름을 공식 문서에서 검색
2. 파라미터 이름과 순서 확인
3. 반환값 타입 확인
4. 예제 코드와 AI가 준 코드 비교

### 방법 3. 실제 실행으로 확인

코드를 받으면 반드시 실행해서 결과를 확인한다.

```python
# AI가 준 코드를 그대로 복붙해서 실행
# 정상 케이스로 먼저 테스트
result = some_function([1, 2, 3])
print(result)  # 예상한 결과인가?

# 엣지 케이스 테스트
result_empty = some_function([])
print(result_empty)  # 빈 리스트는?

result_none = some_function(None)
print(result_none)  # None은?
```

실행 결과와 AI의 설명이 일치하는지 비교한다.

---

## 3. assert로 코드 검증하기

`assert`는 조건이 참인지 확인하는 Python 내장 기능이다. 조건이 거짓이면 즉시 `AssertionError`를 발생시킨다.

```python
assert 조건, "실패 시 출력할 메시지"
```

### 기본 사용법

```python
def calculate_discount(price, rate):
    """rate%만큼 할인된 가격을 반환한다."""
    return price * (1 - rate / 100)

# 정상 케이스 검증
assert calculate_discount(1000, 10) == 900.0, "10% 할인 실패"
assert calculate_discount(1000, 0) == 1000.0, "0% 할인 실패"
assert calculate_discount(1000, 100) == 0.0, "100% 할인 실패"

# 소수점 결과 검증 (부동소수점 비교 주의)
result = calculate_discount(1000, 33)
assert abs(result - 670.0) < 0.01, f"33% 할인 실패: {result}"

print("모든 테스트 통과!")
```

### 타입 검증

```python
def get_even_numbers(numbers):
    return [n for n in numbers if n % 2 == 0]

result = get_even_numbers([1, 2, 3, 4, 5, 6])

# 반환 타입 검증
assert isinstance(result, list), f"리스트를 반환해야 함, 실제: {type(result)}"

# 값 검증
assert result == [2, 4, 6], f"예상: [2, 4, 6], 실제: {result}"

# 길이 검증
assert len(result) == 3, f"3개를 반환해야 함, 실제: {len(result)}개"
```

### 예외 발생 검증

```python
import pytest  # pytest를 사용하는 경우

def divide(a, b):
    return a / b

# ZeroDivisionError가 발생하는지 검증
with pytest.raises(ZeroDivisionError):
    divide(10, 0)

# pytest 없이 직접 검증
try:
    divide(10, 0)
    assert False, "ZeroDivisionError가 발생해야 했는데 발생하지 않음"
except ZeroDivisionError:
    pass  # 예상한 대로 예외 발생
print("예외 테스트 통과!")
```

---

## 4. AI가 준 코드를 검증하는 3단계 체크리스트

AI로부터 코드를 받으면 이 3단계를 순서대로 실행한다.

### 1단계: 코드 읽기 전 확인 (받자마자)

```
□ import 문: 사용한 라이브러리가 실제로 설치 가능한가?
  pip show 라이브러리이름

□ 함수/메서드 이름: 공식 문서에 실제로 존재하는가?
  Python REPL에서 dir() 또는 help() 확인

□ 문법: 현재 Python 버전에서 지원하는 문법인가?
  특이한 문법은 Python 버전 명시 요청
```

### 2단계: 직접 실행 확인

```
□ 정상 케이스: 일반적인 입력으로 예상한 결과가 나오는가?
  실제로 실행하고 print()로 결과 확인

□ 빈 값 케이스: 빈 리스트, 빈 문자열, 0을 입력해도 오류가 없는가?

□ None 케이스: None을 입력하면 어떻게 동작하는가?

□ 경계값: 최대값, 최솟값에서 어떻게 동작하는가?
```

### 3단계: AI 설명 vs 코드 대조

```
□ AI가 설명한 동작과 실제 코드의 동작이 일치하는가?
  "10% 할인" → 실제로 10% 감소하는가?

□ 주석이 코드의 실제 동작을 올바르게 설명하는가?
  # 중복 제거 → 실제로 중복이 제거되는가?

□ 함수 이름과 실제 동작이 일치하는가?
  is_valid() → 실제로 유효성 검사를 하는가?
```

---

## 5. 환각이 의심될 때 AI와 대화하기

### 방법 1. 공식 문서 링크 요청

```
이 코드에서 사용한 df.remove_duplicates() 메서드가 실제로 pandas에 있는지 모르겠어.
공식 pandas 문서 링크와 함께 확인해줄 수 있어?
```

주의: AI가 "있습니다"라고 답해도 직접 확인해야 한다. AI는 존재하지 않는 링크를 만들어낼 수도 있다.

### 방법 2. 실행 결과 공유

```
이 코드를 실행했더니 아래 오류가 났어:
AttributeError: 'DataFrame' object has no attribute 'remove_duplicates'

pandas DataFrame에서 중복 행을 제거하는 올바른 메서드 이름이 뭐야?
```

오류 메시지를 그대로 붙여넣으면 AI가 정확한 원인을 파악할 수 있다.

### 방법 3. 검증 코드 요청

```
이 함수가 제대로 동작하는지 확인할 수 있도록
다양한 케이스(정상, 빈 리스트, None 입력)를 테스트하는 assert 코드를 작성해줘.
```

### 방법 4. 대안 제시 요청

```
내가 원하는 건 pandas DataFrame에서 중복 행을 제거하는 거야.
가장 표준적이고 pandas 공식 문서에 나오는 방법을 알려줘.
```

---

## 6. 환각 빈도가 높은 영역

경험적으로 AI 환각이 자주 나타나는 영역이다.

| 영역 | 주의해야 할 내용 | 확인 방법 |
|------|---------------|---------|
| **외부 라이브러리 API** | 버전마다 다름, 없는 메서드 생성 | 공식 문서, `dir()`, `help()` |
| **날짜/시간 처리** | 시간대, 형식 지정자, 메서드 이름 오류 | datetime 공식 문서 |
| **파일 경로 처리** | OS별 차이, 상대/절대 경로 혼동 | 실제 실행, `pathlib` 사용 권장 |
| **정규표현식** | 패턴 오류, 플래그 누락 | regex101.com 에서 직접 테스트 |
| **AWS/클라우드 API** | 버전 업데이트로 deprecated된 방식 | AWS 공식 문서, boto3 최신 문서 |
| **최신 Python 기능** | 학습 데이터 컷오프 이후 추가된 문법 | Python 버전별 릴리즈 노트 |
| **수식 계산** | 사칙연산은 맞지만 복잡한 수식에서 오류 | 직접 계산으로 검증 |

---

## 7. 따라 하기 실습

### 실습 1. 환각 코드 찾고 수정하기

**아래 코드에서 환각(실제로 동작하지 않는 부분)을 찾아보자.**

```python
import os

# 현재 폴더의 .py 파일만 가져오기
files = os.listdir(".")
py_files = files.filter(lambda f: f.endswith(".py"))
print(f"Python 파일: {py_files}")
```

**Step 1. 실행해서 오류 확인**

```bash
python test_hallucination.py
```

예상 오류:
```
AttributeError: 'list' object has no attribute 'filter'
```

**Step 2. 오류 원인 파악**

`list`에는 `.filter()` 메서드가 없다. `.filter()`는 JavaScript의 배열 메서드다.

Python에서 확인:
```python
python -c "print([m for m in dir([]) if 'filter' in m.lower()])"
# 출력: []  ← filter 관련 메서드 없음
```

**Step 3. 올바른 코드로 수정**

```python
import os

# Python에서 리스트 필터링 방법 1: 리스트 컴프리헨션
files = os.listdir(".")
py_files = [f for f in files if f.endswith(".py")]
print(f"Python 파일: {py_files}")

# 방법 2: filter() 내장 함수 사용
py_files = list(filter(lambda f: f.endswith(".py"), files))
print(f"Python 파일: {py_files}")
```

수정 후 실행해서 정상 동작 확인.

---

### 실습 2. assert로 AI 코드 검증하기

**Step 1. AI에게 다음을 요청한다:**

```
리스트에서 중복을 제거하고 원래 순서를 유지하는
remove_duplicates(items) 함수를 Python으로 만들어줘.
```

AI가 다음과 같은 코드를 줄 수 있다:

```python
def remove_duplicates(items):
    return list(set(items))
```

**Step 2. assert로 검증하기**

```python
def remove_duplicates(items):
    return list(set(items))

# 테스트 1: 중복 제거 확인
result = remove_duplicates([1, 2, 1, 3, 2])
assert set(result) == {1, 2, 3}, f"중복 제거 실패: {result}"
print(f"중복 제거 결과: {result}")

# 테스트 2: 순서 유지 확인 ← 이 테스트에서 실패할 것이다
result_order = remove_duplicates([3, 1, 2, 1, 3])
print(f"순서 결과: {result_order}")
assert result_order == [3, 1, 2], f"순서 유지 실패: {result_order}"
# set()은 순서를 보장하지 않기 때문에 이 테스트는 실패할 수 있다
```

`set()`은 순서를 보장하지 않는다. AI는 "원래 순서 유지" 조건을 제대로 구현하지 않은 것이다.

**Step 3. 올바른 코드 요청 및 재검증**

```python
# 순서를 유지하는 올바른 구현
def remove_duplicates(items):
    seen = set()
    result = []
    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)
    return result

# 재검증
assert remove_duplicates([1, 2, 1, 3, 2]) == [1, 2, 3], "중복 제거 + 순서 유지 실패"
assert remove_duplicates([]) == [], "빈 리스트 실패"
assert remove_duplicates([1, 1, 1]) == [1], "전체 중복 실패"
assert remove_duplicates([3, 1, 2, 1, 3]) == [3, 1, 2], "순서 유지 실패"
print("모든 테스트 통과!")
```

---

### 실습 3. 공식 문서 대조 연습

**목표**: AI가 준 코드를 공식 문서와 직접 대조하는 연습.

**Step 1. AI에게 요청**

```
Python datetime 모듈을 사용해서 "2026-05-21"이라는 문자열을 날짜 객체로 변환하는 코드를 줘.
그리고 그 날짜 객체에서 요일(월=0, 일=6)을 숫자로 가져오는 방법도 알려줘.
```

AI가 다음과 같은 코드를 줄 수 있다:

```python
from datetime import datetime

# 문자열 → 날짜 객체 변환
date_str = "2026-05-21"
date_obj = datetime.strptime(date_str, "%Y-%m-%d")

# 요일 가져오기 (0=월요일, 6=일요일)
weekday = date_obj.weekday()
print(f"요일: {weekday}")
```

**Step 2. 공식 문서에서 확인**

Python 공식 문서에서 `strptime` 검색: https://docs.python.org/3/library/datetime.html#datetime.datetime.strptime

확인할 내용:
- `%Y`, `%m`, `%d` 형식 지정자가 맞는가?
- `weekday()` 반환값이 0=월요일, 6=일요일인가?

**Step 3. 직접 실행으로 검증**

```python
from datetime import datetime

date_str = "2026-05-21"
date_obj = datetime.strptime(date_str, "%Y-%m-%d")
weekday = date_obj.weekday()

# 2026-05-21이 무슨 요일인지 확인
days = ["월", "화", "수", "목", "금", "토", "일"]
print(f"날짜: {date_obj.date()}")
print(f"요일 번호: {weekday}")
print(f"요일 이름: {days[weekday]}")

# assert로 검증
assert 0 <= weekday <= 6, f"요일 번호가 0-6 범위를 벗어남: {weekday}"
```

실행 결과와 실제 달력의 요일을 비교한다.

---

## 자주 막히는 지점

### 막히는 지점 1. "assert가 실패했는데 어떻게 읽는지 모르겠어요"

```
AssertionError: 순서 유지 실패: [2, 1, 3]
```

읽는 방법:
- `AssertionError`: assert 조건이 False였다
- `순서 유지 실패`: assert 뒤에 쓴 메시지
- `[2, 1, 3]`: 실제로 나온 값

예상값 `[1, 2, 3]`이어야 하는데 실제 `[2, 1, 3]`이 나왔다는 뜻이다.

### 막히는 지점 2. "공식 문서를 봐도 뭐가 맞는지 모르겠어요"

가장 빠른 방법: Python REPL에서 `help(함수명)` 실행.

```python
import pandas as pd
df = pd.DataFrame()
help(df.drop_duplicates)
# 파라미터, 설명, 예제가 나온다
```

### 막히는 지점 3. "AI가 틀렸다고 확신하는데, 어떻게 수정할지 모르겠어요"

AI에게 오류 메시지를 그대로 붙여넣어서 다시 요청한다.

```
이 코드를 실행했더니 다음 오류가 났어:
AttributeError: 'list' object has no attribute 'filter'

list에서 조건으로 필터링하는 올바른 Python 방법을 알려줘.
```

---

## 확인 체크리스트

- [ ] AI가 준 코드를 무조건 믿지 않고 항상 실행해서 확인하는가
- [ ] 처음 보는 함수/메서드는 `dir()`, `help()`, 공식 문서로 확인하는가
- [ ] `assert`로 예상 결과를 코드로 검증할 수 있는가
- [ ] 빈 리스트, None, 경계값 등 엣지 케이스로 테스트하는가
- [ ] 환각이 의심될 때 AI에게 오류 메시지를 붙여넣어 수정을 요청하는가

---

## 한 번 더 생각해 보기

1. AI가 자신있게 틀린 코드를 줄 때, 어조만으로는 환각을 판별할 수 없다. 그렇다면 무엇으로 판별해야 하는가?
2. 모든 코드를 공식 문서로 검증하는 것이 현실적인가? 어떤 코드는 검증하고 어떤 코드는 믿어도 되는가?
3. AI에게 "이 코드가 맞는지 검증해줘"라고 요청하면 환각을 잡을 수 있을까? 왜 그럴 수도, 그렇지 않을 수도 있는가?
4. 환각이 가장 위험한 상황은 언제인가? (오류가 나는 경우 vs 오류 없이 틀린 결과를 내는 경우)

---

## 다음 장

AI 환각에 대처하는 능력을 키웠다. 다음 단계에서는 이 모든 기술을 종합해서 실제 Lambda 서비스에 AI 도움을 활용하면서도 안전하게 코드를 작성하고 배포하는 워크플로를 배운다.

---

## 참고 자료

- Python 표준 라이브러리 공식 문서 — https://docs.python.org/3/library/index.html
- Python Built-in Functions — https://docs.python.org/3/library/functions.html
- pandas 공식 문서 — https://pandas.pydata.org/docs/reference/
- boto3 (AWS SDK for Python) — https://boto3.amazonaws.com/v1/documentation/api/latest/
