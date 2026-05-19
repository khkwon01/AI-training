# Chapter 01: AI와 함께 코딩하는 워크플로우와 검증

## 이 장에서 배우는 것

- AI와 코딩할 때 실제로 어떤 흐름으로 작업하는지
- AI가 자주 틀리는 패턴 5가지
- 코드를 검증하는 체크리스트
- 실습: AI에게 코드 요청 → 검토 → 수정하는 전체 과정

---

## 왜 AI 워크플로우가 필요한가

AI는 코딩을 빠르게 시작할 수 있게 도와준다. 문법을 잊었을 때 물어볼 수 있고, 구조를 잡을 때 아이디어를 얻을 수 있다.

그런데 AI를 처음 쓰는 사람들이 흔히 하는 실수가 있다. AI가 준 코드를 그대로 복사해서 실행하는 것이다.

AI는 다음과 같은 오류를 만들어낸다.

- 존재하지 않는 함수를 사용하는 코드
- 이미 변경된 구버전 API를 쓰는 코드
- 실행은 되지만 결과가 틀린 코드
- 보안 취약점이 있는 코드
- 불필요하게 복잡한 코드

이것이 잘못이 아니다. AI는 확률적으로 그럴듯한 코드를 생성하는 도구다. 그러므로 AI를 쓸 때는 "내가 검증하는 역할을 한다"고 생각해야 한다.

올바른 태도: AI는 초안을 빠르게 만들어 주는 도우미다. 최종 판단은 사람이 한다.

---

## 1. AI와 코딩하는 단계별 워크플로우

### 1단계: 요청 — 무엇을 원하는지 명확하게 전달한다

AI에게 막연하게 질문하면 막연한 답이 나온다. 원하는 것을 구체적으로 설명해야 좋은 결과를 얻는다.

나쁜 요청과 좋은 요청을 비교해 보자.

```
나쁜 요청:
"파이썬으로 파일 읽는 코드 만들어줘"

좋은 요청:
"Python으로 텍스트 파일을 읽는 함수를 만들어줘.
조건:
- 파일 이름을 인자로 받는다
- 파일이 없으면 None을 반환한다
- 인코딩은 UTF-8을 사용한다
- 초보자도 이해할 수 있도록 주석을 달아줘"
```

좋은 요청의 요소:
- 무엇을 만드는가 (함수, 클래스, 스크립트)
- 입력과 출력이 무엇인가
- 어떤 오류 상황을 처리해야 하는가
- 대상 독자가 누구인가 (초보자, 전문가)
- 어떤 제약 조건이 있는가

### 2단계: 검토 — 코드를 실행하기 전에 읽는다

AI가 코드를 주면 바로 실행하지 않는다. 먼저 읽으면서 이해하려고 한다.

읽을 때 확인하는 것들:

- 함수 이름이 기능을 잘 표현하고 있는가
- 입력과 출력이 내가 원한 것과 맞는가
- 변수 이름이 이해되는가
- 논리 흐름이 말이 되는가
- 이해가 안 되는 함수나 메서드가 있는가

이해가 안 되는 부분이 있으면 AI에게 바로 물어본다.

```
"이 코드에서 `with open() as f:` 부분이 왜 with를 써야 하는지 설명해줘"
```

### 3단계: 수정 — 직접 고친다

검토하다가 문제를 발견하거나, 내 상황에 맞게 바꿔야 할 때 직접 수정한다. AI에게 수정을 요청해도 되지만, 작은 수정은 직접 해보는 것이 실력을 키우는 데 도움이 된다.

수정할 때:
- 한 번에 여러 곳을 바꾸지 않는다
- 수정 후 바로 실행해서 결과를 확인한다
- 왜 수정했는지 주석이나 메모로 남긴다

### 4단계: 검증 — 다양한 입력으로 테스트한다

코드가 "일단 동작"한다고 완성된 것이 아니다. 다양한 상황을 테스트해야 한다.

테스트해야 할 상황들:
- 정상적인 입력 (기대한 대로 동작하는가)
- 경계값 (최솟값, 최댓값)
- 비정상적인 입력 (빈 문자열, 0, 음수, None)
- 극단적인 경우 (매우 큰 숫자, 아주 긴 문자열)

---

## 2. AI가 자주 틀리는 패턴 5가지

AI가 내놓는 코드가 완벽하지 않은 이유와 구체적인 패턴을 알면, 검토할 때 어디를 집중적으로 봐야 할지 알 수 있다.

### 패턴 1: 존재하지 않는 함수나 메서드 사용

AI는 실제로 없는 함수를 "있는 것처럼" 만들어낼 때가 있다. 특히 라이브러리를 쓸 때 이런 일이 잦다.

```python
# AI가 생성한 코드 (잘못된 예)
import pandas as pd

df = pd.read_csv("data.csv")
df.filter_rows(df["score"] > 80)  # filter_rows()는 존재하지 않는다!

# 실제 올바른 코드
df_filtered = df[df["score"] > 80]
```

대응 방법: 처음 보는 함수나 메서드는 공식 문서에서 검색해서 존재하는지 확인한다.

### 패턴 2: 구버전 API 사용

AI의 학습 데이터에는 오래된 코드도 포함되어 있다. 최신 라이브러리 버전에서는 이미 바뀐 API를 쓰는 코드가 나올 수 있다.

```python
# AI가 생성한 코드 (구버전 방식)
from openai import OpenAI
response = openai.ChatCompletion.create(  # 구버전 API
    model="gpt-4",
    messages=[{"role": "user", "content": "안녕"}]
)

# 현재 올바른 방식
client = OpenAI()
response = client.chat.completions.create(  # 현재 API
    model="gpt-4",
    messages=[{"role": "user", "content": "안녕"}]
)
```

대응 방법: `DeprecationWarning` 메시지가 나오면 공식 문서에서 최신 방법을 찾는다.

### 패턴 3: 논리 오류 (실행은 되지만 결과가 틀림)

코드가 오류 없이 실행되지만, 결과가 기대와 다른 경우다. 가장 발견하기 어려운 패턴이다.

```python
# AI가 생성한 코드 (논리 오류)
def calculate_average(scores):
    total = 0
    for score in scores:
        total = score  # 오류: += 대신 = 를 써서 마지막 값만 남는다
    return total / len(scores)

# 테스트
print(calculate_average([80, 90, 70]))  # 기대: 80.0, 실제: 23.3...

# 올바른 코드
def calculate_average(scores):
    total = 0
    for score in scores:
        total += score  # += 로 누적해야 한다
    return total / len(scores)
```

대응 방법: 예상 결과를 먼저 계산해보고, 코드 출력과 비교한다.

### 패턴 4: 보안 취약점

간단한 예시 코드에서는 보안을 고려하지 않는 경우가 많다. 실제 서비스에 그대로 쓰면 위험하다.

```python
# AI가 생성한 코드 (보안 문제 있음)
import sqlite3

def get_user(username):
    conn = sqlite3.connect("db.sqlite")
    cursor = conn.cursor()
    # SQL Injection 취약점: 사용자 입력을 직접 SQL에 삽입
    query = f"SELECT * FROM users WHERE name = '{username}'"
    cursor.execute(query)
    return cursor.fetchone()

# 만약 username = "' OR 1=1 --" 이면 모든 사용자 정보가 노출된다

# 올바른 코드 (파라미터화된 쿼리 사용)
def get_user(username):
    conn = sqlite3.connect("db.sqlite")
    cursor = conn.cursor()
    query = "SELECT * FROM users WHERE name = ?"  # ? 플레이스홀더 사용
    cursor.execute(query, (username,))
    return cursor.fetchone()
```

대응 방법: 사용자 입력을 그대로 SQL이나 시스템 명령에 넣는 코드는 항상 의심한다.

### 패턴 5: 불필요한 복잡성

간단하게 해결할 수 있는 문제를 복잡하게 만드는 경우다. 코드가 길고 어려워 보이지만 실제로는 더 간단하게 쓸 수 있다.

```python
# AI가 생성한 코드 (불필요하게 복잡)
def is_even(number):
    if number % 2 == 0:
        if True:
            return True
        else:
            return False
    else:
        return False

# 훨씬 간단한 코드
def is_even(number):
    return number % 2 == 0
```

```python
# AI가 생성한 코드 (불필요하게 복잡)
numbers = [1, 2, 3, 4, 5]
result = []
for i in range(len(numbers)):
    result.append(numbers[i] * 2)

# 더 간단한 코드
result = [n * 2 for n in numbers]
```

대응 방법: "이것보다 더 간단하게 쓸 수 있지 않을까?" 라고 먼저 물어본다.

---

## 3. 검증 체크리스트

AI가 준 코드를 검토할 때 다음 항목을 순서대로 확인한다.

### 실행 가능성 확인

- 코드를 실행했을 때 오류 없이 돌아가는가
- import 문이 모두 있는가
- 필요한 패키지가 설치되어 있는가

### 결과 정확성 확인

- 간단한 예시 입력으로 기대하는 결과가 나오는가
- 계산이 필요한 경우, 손으로 계산한 결과와 일치하는가
- 함수의 반환값 타입이 내가 원하는 것인가

### 엣지케이스 확인

엣지케이스란 "보통은 잘 안 생기지만 생기면 문제가 되는 상황"이다.

| 상황 | 확인 방법 |
|------|-----------|
| 빈 리스트/문자열/딕셔너리 | `[]`, `""`, `{}` 입력으로 테스트 |
| None 값 | `None` 입력으로 테스트 |
| 최댓값/최솟값 | 허용 범위의 끝값으로 테스트 |
| 음수/0 | 음수나 0 입력으로 테스트 |
| 매우 큰 값 | 비정상적으로 큰 숫자로 테스트 |

```python
# 엣지케이스 테스트 예시
def calculate_average(scores):
    if not scores:  # 빈 리스트 처리 여부 확인
        return 0
    return sum(scores) / len(scores)

# 정상 케이스
print(calculate_average([80, 90, 70]))   # 기대: 80.0

# 엣지케이스
print(calculate_average([]))             # 기대: 0 (오류 없이 처리)
print(calculate_average([100]))          # 기대: 100.0 (하나짜리)
print(calculate_average([0, 0, 0]))      # 기대: 0.0
```

### 코드 품질 확인

- 변수 이름이 의미를 담고 있는가 (`x` 대신 `score`, `a` 대신 `filename`)
- 같은 코드가 여러 번 반복되지는 않는가
- 이해하기 어려운 부분에 주석이 있는가

---

## 실습: AI에게 코드 요청 → 검토 → 수정하는 전체 과정

### 따라 하기

실제 AI에게 코드를 요청하고 검토하고 수정하는 과정을 단계별로 따라가 보자.

**시나리오**: 학생 점수 목록을 받아서 통계를 계산하는 함수가 필요하다.

---

**1단계: 요청 작성**

다음과 같이 구체적으로 요청한다.

```
Python 함수를 만들어줘.

기능: 학생 점수 리스트를 받아서 통계를 계산한다.
입력: 정수 점수의 리스트 (예: [85, 92, 78, 65])
출력: 딕셔너리 (평균, 최고점, 최저점, 합격자 수 포함)
조건:
- 합격 기준은 60점 이상
- 빈 리스트가 들어오면 None을 반환한다
- 주석을 달아줘
```

---

**2단계: AI가 생성한 코드 (예시)**

AI가 이런 코드를 줬다고 가정하자.

```python
def analyze_scores(scores):
    # 빈 리스트 처리
    if len(scores) == 0:
        return None

    total = 0
    for score in scores:
        total = score  # 여기 문제 있음

    average = total / len(scores)
    highest = max(scores)
    lowest = min(scores)
    passed = len([s for s in scores if s >= 60])

    return {
        "average": average,
        "highest": highest,
        "lowest": lowest,
        "passed": passed,
        "total": len(scores)  # 요청하지 않은 항목 추가됨
    }
```

---

**3단계: 코드 검토**

코드를 읽으면서 아래 사항을 확인한다.

```
확인 결과:
1. total = score 에서 += 가 누락됨 → 마지막 점수만 남는다 (논리 오류)
2. if len(scores) == 0: → if not scores: 가 더 Python답다 (품질)
3. "total" 키는 요청하지 않았지만 있어도 유용하다 (판단: 남겨둔다)
4. 빈 리스트 처리는 있다 (확인 완료)
5. 합격 기준 60점은 요청대로 구현됐다 (확인 완료)
```

---

**4단계: 수정**

발견한 문제를 수정한다.

```python
def analyze_scores(scores):
    """
    학생 점수 리스트를 분석해서 통계를 반환하는 함수.

    Args:
        scores: 정수 점수의 리스트 (예: [85, 92, 78])

    Returns:
        통계 딕셔너리 또는 None (빈 리스트인 경우)
    """
    if not scores:  # 수정 1: if not scores로 변경
        return None

    total = 0
    for score in scores:
        total += score  # 수정 2: = 를 += 로 변경 (핵심 버그 수정)

    average = total / len(scores)
    highest = max(scores)
    lowest = min(scores)
    passed = len([s for s in scores if s >= 60])

    return {
        "average": round(average, 1),  # 수정 3: 소수점 1자리로 반올림
        "highest": highest,
        "lowest": lowest,
        "passed": passed,
        "total_count": len(scores)  # 수정 4: 이름 명확하게 변경
    }
```

---

**5단계: 검증**

다양한 입력으로 테스트한다.

```python
# 정상 케이스
result = analyze_scores([85, 92, 78, 65, 45])
print(result)
# 기대: {'average': 73.0, 'highest': 92, 'lowest': 45, 'passed': 4, 'total_count': 5}

# 하나짜리
result = analyze_scores([100])
print(result)
# 기대: {'average': 100.0, 'highest': 100, 'lowest': 100, 'passed': 1, 'total_count': 1}

# 빈 리스트
result = analyze_scores([])
print(result)
# 기대: None

# 모두 불합격
result = analyze_scores([30, 40, 50])
print(result)
# 기대: passed가 0
```

### 직접 해보기

1. 실제 AI 도구에 위의 요청을 그대로 보내고, 받은 코드를 검토해 보자
2. 위에서 설명한 5가지 패턴 중 어떤 문제가 있는지 찾아보자
3. 문제를 직접 수정하고, 테스트 코드를 작성해서 검증해 보자

---

## 자주 하는 실수

- AI가 준 코드를 검토 없이 그대로 복사하기
- "코드가 실행됐다"를 "코드가 올바르다"와 같다고 생각하기
- 처음 보는 함수를 실제로 존재하는지 확인하지 않기
- 엣지케이스를 테스트하지 않기
- AI에게 너무 짧게 요청하기

---

## 확인 체크리스트

- AI에게 요청할 때 입력, 출력, 조건을 명확하게 썼는가
- AI가 준 코드를 실행하기 전에 먼저 읽었는가
- 처음 보는 함수가 실제로 존재하는지 확인했는가
- 정상 케이스와 엣지케이스 모두 테스트했는가
- 논리 오류를 찾기 위해 손으로 계산해서 비교했는가

---

## 한 번 더 생각해 보기

1. AI가 논리 오류를 만드는 이유는 무엇일까?
2. AI 코드를 검증하는 능력은 어떻게 키울 수 있을까?
3. AI 없이도 코드를 작성하는 능력이 왜 중요할까?
4. AI가 생성한 코드에서 보안 취약점을 발견하려면 무엇을 공부해야 할까?
