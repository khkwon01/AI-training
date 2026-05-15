# ai-train AI Chapter 06: AI 환각(Hallucination) 감지 체크리스트

## 이 장에서 배우는 것

- AI 환각(hallucination)이 무엇인지
- 코드에서 환각이 어떤 형태로 나타나는지
- 환각을 감지하는 구체적인 방법
- assert와 테스트로 코드 정확성을 검증하는 방법
- 환각이 의심될 때 AI와 대화하는 방법

---

## 먼저 쉬운 설명

AI는 틀린 정보를 매우 자신있게 말할 때가 있다. 이것을 **환각(hallucination)**이라고 한다.

코드에서는 이런 형태로 나타난다:

- 존재하지 않는 함수나 라이브러리를 사용
- 실제 동작과 다른 설명
- 오래된 문법이나 삭제된 기능 사용
- 잘못된 API 응답 형태 가정

환각은 코드가 **실행은 되지만 틀린 결과**를 낼 때 특히 위험하다.

---

## 1. 코드 환각의 주요 유형

### 유형 1. 존재하지 않는 함수/메서드

```python
# AI가 준 코드 (환각)
import pandas as pd
df = pd.read_csv("data.csv")
df.remove_duplicates()   # ← 실제 메서드는 drop_duplicates()
```

**감지법**: 공식 문서나 `help(df)` 로 메서드 존재 여부 확인

---

### 유형 2. 잘못된 인자 순서 또는 타입

```python
# AI가 준 코드 (환각)
import datetime
d = datetime.date(day=15, month=5, year=2026)   # 실제는 (year, month, day)
```

**감지법**: 직접 실행해서 오류 확인, 공식 문서 대조

---

### 유형 3. 존재하지 않는 라이브러리 버전의 기능

```python
# AI가 준 코드 (환각 - 가상의 기능)
from requests import smart_retry
response = smart_retry.get("https://api.example.com")
```

**감지법**: `pip show requests` 로 버전 확인, 공식 문서 검색

---

### 유형 4. 동작하지만 결과가 틀린 코드

```python
# AI가 준 코드 (환각)
def is_palindrome(text):
    """문자열이 회문인지 확인한다."""
    return text == text[::-1]   # 공백과 대소문자를 고려하지 않음

# "A man a plan a canal Panama" → False (실제로는 회문)
```

**감지법**: 다양한 입력으로 직접 테스트

---

## 2. 환각 감지 체크리스트

AI가 준 코드를 받았을 때 아래를 확인한다.

### 즉시 확인

- [ ] **import 문**: 사용한 라이브러리가 실제로 설치 가능한가?
  ```bash
  pip show 라이브러리이름
  ```
- [ ] **함수/메서드 이름**: 공식 문서에 실제로 존재하는가?
- [ ] **문법**: 현재 Python 버전에서 지원하는 문법인가?

### 실행 후 확인

- [ ] **정상 케이스**: 일반적인 입력으로 예상한 결과가 나오는가?
- [ ] **엣지 케이스**: 빈 값, 경계값, 잘못된 타입으로도 테스트했는가?
- [ ] **출력 형식**: 반환값의 타입이 설명과 일치하는가?

### 설명 vs 코드 대조

- [ ] AI가 설명한 내용과 실제 코드가 일치하는가?
- [ ] 주석이 코드의 실제 동작을 올바르게 설명하는가?

---

## 3. assert로 검증하기

환각을 빠르게 잡는 가장 쉬운 방법은 `assert`로 예상 결과를 직접 확인하는 것이다.

```python
def calculate_discount(price, rate):
    """rate%만큼 할인된 가격을 반환한다."""
    return price * (1 - rate / 100)

# AI 설명: "10% 할인 시 1000원 → 900원"
assert calculate_discount(1000, 10) == 900.0, "10% 할인 실패"

# 추가 테스트
assert calculate_discount(1000, 0) == 1000.0, "0% 할인 실패"
assert calculate_discount(1000, 100) == 0.0, "100% 할인 실패"

print("모든 테스트 통과!")
```

`assert` 조건이 `False`이면 즉시 오류가 발생한다. 통과하면 다음 줄로 진행한다.

---

## 4. 환각이 의심될 때 AI와 대화하기

### 방법 1: 공식 문서 링크 요청

```
이 코드에서 사용한 df.remove_duplicates() 가 실제로 pandas에 있는지 확인하고,
공식 문서 링크도 같이 알려줘.
```

### 방법 2: 직접 실행 결과 공유

```
이 코드를 실행했더니 아래 오류가 났어.
AttributeError: 'DataFrame' object has no attribute 'remove_duplicates'

혹시 올바른 메서드 이름을 알려줄 수 있어?
```

### 방법 3: 대안 확인 요청

```
이 기능을 구현하는 방법이 여러 가지 있을 것 같아.
가장 표준적이고 널리 쓰이는 방법을 알려줘.
그리고 내가 확인할 수 있는 공식 문서 위치도 알려줘.
```

---

## 5. 환각 빈도가 높은 영역

경험적으로 AI 환각이 자주 나타나는 영역이다.

| 영역 | 주의 사항 |
|------|----------|
| 외부 라이브러리 API | 버전마다 다름, 공식 문서 필수 확인 |
| 날짜/시간 처리 | 시간대, 형식, 메서드 이름 오류 빈번 |
| 파일 경로 처리 | OS별 차이, 상대/절대 경로 혼동 |
| 정규표현식 | 패턴 오류, 플래그 누락 |
| AWS/클라우드 API | 버전 업데이트로 deprecated된 방식 |
| 최신 Python 기능 | 학습 데이터 컷오프 이후 기능 모를 수 있음 |

---

## 6. 따라 하기 실습

### 실습 1. 환각 코드 찾기

아래 코드에서 환각(실제로 동작하지 않는 부분)을 찾아보자.

```python
import os

# 파일 목록을 가져와서 .py 파일만 필터링
files = os.listdir(".")
py_files = files.filter(lambda f: f.endswith(".py"))   # 환각?
print(py_files)
```

실행해서 오류를 확인하고, 올바른 코드로 수정한다.

### 실습 2. assert로 함수 검증

AI에게 아래 함수를 요청하고, 직접 assert로 검증하는 테스트를 3개 이상 작성한다.

```
리스트에서 중복을 제거하고 원래 순서를 유지하는 remove_duplicates(items) 함수를 만들어줘.
```

```python
# 받은 함수를 테스트하는 코드
assert remove_duplicates([1, 2, 1, 3, 2]) == [1, 2, 3]
assert remove_duplicates([]) == []
assert remove_duplicates([1, 1, 1]) == [1]
# 추가 테스트 직접 작성
```

### 실습 3. 공식 문서 대조

AI에게 `datetime.datetime.strptime()` 사용법을 물어보고, 공식 문서에서 실제 파라미터와 일치하는지 확인한다.

---

## 확인 체크리스트

- [ ] AI가 준 코드에서 import 문과 함수 이름을 직접 확인하는 습관이 있는가
- [ ] 다양한 입력으로 테스트해서 결과가 설명과 일치하는지 확인하는가
- [ ] `assert`로 예상 결과를 코드로 검증할 수 있는가
- [ ] 환각이 의심될 때 AI에게 공식 문서 링크를 요청할 수 있는가

---

## 한 번 더 생각해 보기

1. AI가 자신있게 틀린 코드를 줄 때, 어떻게 환각임을 판별할 수 있을까?
2. 모든 코드를 공식 문서로 검증하는 것이 현실적인가? 어디서 선을 그어야 할까?
3. AI에게 "이 코드가 맞는지 검증해줘"라고 요청하면 환각을 잡을 수 있을까?

---

## 참고 자료

- Python 표준 라이브러리 문서 — https://docs.python.org/3/library/index.html
- Python Built-in Functions — https://docs.python.org/3/library/functions.html
