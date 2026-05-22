# Lab 01: AI 코드 리뷰 실습

## 이 실습에서 배우는 것

- AI 도구(Claude, GitHub Copilot 등)를 활용해서 실제 Pull Request 코드를 리뷰하는 흐름을 익힌다
- 의도적으로 문제 있는 코드를 작성하고, AI가 어떤 피드백을 주는지 직접 경험한다
- AI 리뷰 결과를 비판적으로 읽고, 반영할 것과 무시할 것을 스스로 판단한다
- 수정 코드를 커밋하고 PR을 업데이트하는 전체 흐름을 완성한다

---

## 먼저 쉬운 설명

코드 리뷰(Code Review)는 내가 만든 코드를 다른 사람이 읽고 피드백을 주는 과정이에요.
팀에서는 동료 개발자가 리뷰어가 되지만, 혼자 작업할 때는 AI가 좋은 리뷰어가 됩니다.

AI 리뷰의 장점:
- 24시간 언제든지 요청 가능
- 오타, 잠재적 버그, 가독성 문제 등을 빠르게 잡아냄
- 부끄럽지 않게 솔직한 피드백을 받을 수 있음

단, AI 리뷰가 항상 정답은 아니에요. AI의 제안을 **그대로 받아들이는 것**이 아니라,
**왜 그렇게 말하는지 이해한 다음 스스로 판단**하는 것이 핵심입니다.

---

## 실습 준비

이 실습을 시작하기 전에 다음이 준비되어 있어야 합니다:

- GitHub 계정
- Python 3.x 설치
- 텍스트 에디터 (VS Code 권장)
- Claude (claude.ai) 또는 GitHub Copilot 중 하나 이상

---

## 실습 1: 의도적으로 문제 있는 Python 코드 작성하기

좋은 코드 리뷰를 경험하려면, **문제가 있는 코드**가 필요합니다.
아래 코드는 기능은 동작하지만 여러 가지 나쁜 습관이 포함되어 있습니다.

### 파일 만들기

`bad_code.py` 파일을 만들고 아래 코드를 그대로 복사하세요.

```python
import json
import requests

# 사용자 데이터 처리 함수
def f(d):
    r = []
    for i in range(len(d)):
        x = d[i]
        if x['age'] > 18:
            r.append(x['name'])
    return r

# 파일에서 데이터 읽기
def read(path):
    f = open(path)
    data = json.load(f)
    return data

# API 호출
def get_data(url):
    response = requests.get(url)
    data = response.json()
    return data

# 두 리스트 합치기
def combine(list1, list2):
    result = []
    for item in list1:
        result.append(item)
    for item in list2:
        result.append(item)
    return result

# 점수 계산
def calc(scores):
    total = 0
    for s in scores:
        total = total + s
    avg = total / len(scores)
    return avg

# 메인 실행
def main():
    data1 = [
        {"name": "김철수", "age": 20},
        {"name": "이영희", "age": 17},
        {"name": "박민준", "age": 25},
    ]
    data2 = [
        {"name": "최지원", "age": 16},
        {"name": "정수연", "age": 30},
    ]
    
    all_data = combine(data1, data2)
    adults = f(all_data)
    
    scores = [85, 92, 78, 90, 88]
    average = calc(scores)
    
    print(adults)
    print(average)

if __name__ == "__main__":
    main()
```

### 이 코드에 숨어있는 문제들

코드를 실행하면 동작은 합니다. 하지만 다음과 같은 문제가 있습니다:

| 문제 유형 | 위치 | 설명 |
|-----------|------|------|
| 나쁜 함수명 | `def f(d)` | 함수가 무엇을 하는지 이름에서 알 수 없음 |
| 나쁜 변수명 | `r`, `x`, `i` | 의미 없는 한 글자 변수명 |
| 파일 미닫음 | `read()` 함수 | `open()` 후 `close()` 없음 — 리소스 누수 |
| 에러 처리 없음 | `get_data()` | API 실패 시 예외 처리 없음 |
| 에러 처리 없음 | `calc()` | 빈 리스트 전달 시 ZeroDivisionError 발생 |
| 불필요한 중복 | `combine()` | `list1 + list2`로 한 줄에 해결 가능 |
| 비효율적인 반복 | `f()` 함수 내부 | `for i in range(len(d))`는 Pythonic하지 않음 |

지금은 이 문제들을 고치지 마세요. AI 리뷰를 받기 위해 일부러 이 상태로 둡니다.

---

## 실습 2: GitHub에 PR 만들기

### 단계별 순서

**1. 새 브랜치 만들기**

```bash
git checkout -b feature/user-data-processor
```

**2. 파일 추가하고 커밋**

```bash
git add bad_code.py
git commit -m "Add user data processor"
```

**3. 브랜치를 원격 저장소에 push**

```bash
git push origin feature/user-data-processor
```

**4. GitHub에서 PR 만들기**

- GitHub 저장소 페이지로 이동
- "Compare & pull request" 버튼 클릭
- PR 제목: `Add user data processor`
- PR 설명:
  ```
  ## 변경 내용
  - 사용자 데이터 필터링 함수 추가
  - 파일 읽기 유틸리티 추가
  - 점수 계산 함수 추가
  
  ## 테스트
  - 로컬 실행 확인 완료
  ```
- "Create pull request" 클릭

이제 AI 리뷰를 받을 준비가 됐습니다.

---

## 실습 3: AI에게 PR 코드 리뷰 요청하기

### 프롬프트 형식이 중요한 이유

AI에게 단순히 "리뷰해줘"라고 하면 너무 일반적인 답이 옵니다.
**역할, 코드, 맥락, 요청 범위**를 명확히 줄수록 좋은 리뷰를 받을 수 있어요.

---

### 프롬프트 템플릿 1 — 전체 코드 리뷰

```
너는 Python 코드 리뷰어야. 주니어 개발자가 작성한 다음 코드를 리뷰해줘.

리뷰 기준:
1. 변수명과 함수명이 명확한가?
2. 에러 처리가 적절한가?
3. 중복 코드나 비효율적인 부분이 있는가?
4. Python 관용 표현(Pythonic)을 잘 활용하고 있는가?

각 문제에 대해 다음 형식으로 답해줘:
- 문제: [어떤 문제인지]
- 위치: [어느 함수/줄인지]
- 제안: [어떻게 고치면 좋은지]
- 심각도: [높음/중간/낮음]

--- 코드 ---
[여기에 bad_code.py 내용 전체 붙여넣기]
```

---

### 프롬프트 템플릿 2 — 특정 함수 집중 리뷰

```
아래 Python 함수를 리뷰해줘.
특히 에러 처리와 리소스 관리 측면에서 문제가 있는지 확인해줘.

def read(path):
    f = open(path)
    data = json.load(f)
    return data

이 코드의 잠재적 문제점과 개선 방법을 알려줘.
수정된 코드 예시도 보여줘.
```

---

### 프롬프트 템플릿 3 — 보안/성능 관점 리뷰

```
Python 코드를 보안과 성능 관점에서만 리뷰해줘.

보안 체크포인트:
- 외부 입력값 검증 여부
- 예외 처리에서 민감 정보 노출 여부

성능 체크포인트:
- 불필요한 루프나 연산
- 메모리 효율

--- 코드 ---
[코드 붙여넣기]
```

---

### Claude에서 실행하는 방법

1. claude.ai 접속
2. 새 대화 시작
3. 위 템플릿 중 하나를 선택하고, `[코드]` 자리에 `bad_code.py` 내용을 붙여넣기
4. 전송

---

## AI 리뷰 결과 예시

아래는 실제로 Claude에게 리뷰를 요청했을 때 받을 수 있는 응답의 예시입니다.

---

### 예시 응답

**문제 1: 함수명과 변수명이 불명확함**
- 문제: `f(d)` 함수는 이름만 봐서는 무엇을 하는지 알 수 없습니다.
- 위치: `def f(d)` 함수 전체
- 제안: `get_adult_names(users)` 처럼 동작을 설명하는 이름을 사용하세요.
- 심각도: 중간

```python
# Before
def f(d):
    r = []
    for i in range(len(d)):
        x = d[i]
        if x['age'] > 18:
            r.append(x['name'])
    return r

# After
def get_adult_names(users):
    return [user['name'] for user in users if user['age'] > 18]
```

---

**문제 2: 파일 핸들이 닫히지 않음**
- 문제: `open()`으로 파일을 열었지만 닫지 않아서 파일 핸들이 누수됩니다.
- 위치: `read()` 함수
- 제안: `with` 구문을 사용해서 자동으로 파일이 닫히도록 하세요.
- 심각도: 높음

```python
# Before
def read(path):
    f = open(path)
    data = json.load(f)
    return data

# After
def read_json_file(path):
    with open(path, encoding='utf-8') as f:
        return json.load(f)
```

---

**문제 3: 에러 처리 없음 — API 호출**
- 문제: `requests.get()`이 실패(네트워크 오류, 404 등)할 경우 예외 처리가 없어서 프로그램이 강제 종료됩니다.
- 위치: `get_data()` 함수
- 제안: `try/except`로 에러를 잡거나 `response.raise_for_status()`로 HTTP 오류를 처리하세요.
- 심각도: 높음

```python
# Before
def get_data(url):
    response = requests.get(url)
    data = response.json()
    return data

# After
def get_data(url):
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"API 요청 실패: {e}")
        return None
```

---

**문제 4: 빈 리스트 처리 안 됨**
- 문제: `calc([])`처럼 빈 리스트를 전달하면 `ZeroDivisionError`가 발생합니다.
- 위치: `calc()` 함수의 `avg = total / len(scores)` 부분
- 제안: 빈 리스트에 대한 방어 코드를 추가하세요.
- 심각도: 높음

```python
# Before
def calc(scores):
    total = 0
    for s in scores:
        total = total + s
    avg = total / len(scores)
    return avg

# After
def calculate_average(scores):
    if not scores:
        return 0
    return sum(scores) / len(scores)
```

---

**문제 5: 불필요한 함수 — combine()**
- 문제: 두 리스트를 합치는 함수가 있지만, Python의 `+` 연산자로 한 줄에 해결 가능합니다.
- 위치: `combine()` 함수 전체
- 제안: 이 함수를 삭제하고 `all_data = data1 + data2`로 대체하세요.
- 심각도: 낮음

---

## 실습 4: AI 리뷰 결과 읽고 반영 판단하기

AI가 주는 모든 피드백을 다 따를 필요는 없어요. 중요한 건 **스스로 판단**하는 것입니다.

### 판단 기준표

| 피드백 유형 | 반영 여부 | 판단 이유 |
|-------------|-----------|-----------|
| 파일을 닫지 않음 (`open` 미닫음) | 반드시 반영 | 실제 리소스 누수가 발생하는 버그 |
| API 에러 처리 없음 | 반드시 반영 | 네트워크 오류 시 프로그램 중단 |
| 빈 리스트 처리 없음 | 반드시 반영 | 런타임 에러 발생 가능 |
| 함수명/변수명 개선 | 상황에 맞게 | 팀 코딩 컨벤션이 있으면 그것을 따름 |
| combine() 삭제 제안 | 선택적 | 작은 최적화, 당장 필수는 아님 |

### 무시해도 되는 AI 피드백 예시

AI가 때때로 지나치게 세세한 제안을 하기도 합니다:

- "이 변수명을 `x` 대신 `user_item`으로 바꾸세요" — 코드가 내부 스코프에서만 쓰인다면 큰 문제 없음
- "type hint를 추가하세요" — 팀에서 아직 사용하지 않는다면 지금 당장 필수는 아님
- "docstring을 추가하세요" — 실습용 코드라면 생략 가능

**AI 리뷰는 조언이지, 명령이 아닙니다.**

---

## 실습 5: 수정 코드 커밋하고 PR 업데이트하기

### 수정된 코드 작성

AI 리뷰를 바탕으로 `bad_code.py`를 수정합니다. 아래는 수정 후 완성된 코드 예시입니다.

```python
import json
import requests


def get_adult_names(users):
    """18세 초과 사용자의 이름 목록을 반환한다."""
    return [user['name'] for user in users if user['age'] > 18]


def read_json_file(path):
    """JSON 파일을 읽고 데이터를 반환한다."""
    with open(path, encoding='utf-8') as f:
        return json.load(f)


def get_data(url):
    """URL에서 JSON 데이터를 가져온다. 실패 시 None을 반환한다."""
    try:
        response = requests.get(url, timeout=10)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException as e:
        print(f"API 요청 실패: {e}")
        return None


def calculate_average(scores):
    """점수 리스트의 평균을 반환한다. 빈 리스트면 0을 반환한다."""
    if not scores:
        return 0
    return sum(scores) / len(scores)


def main():
    data1 = [
        {"name": "김철수", "age": 20},
        {"name": "이영희", "age": 17},
        {"name": "박민준", "age": 25},
    ]
    data2 = [
        {"name": "최지원", "age": 16},
        {"name": "정수연", "age": 30},
    ]

    all_data = data1 + data2
    adults = get_adult_names(all_data)

    scores = [85, 92, 78, 90, 88]
    average = calculate_average(scores)

    print(f"성인 목록: {adults}")
    print(f"평균 점수: {average:.1f}")


if __name__ == "__main__":
    main()
```

### 수정사항 커밋

```bash
git add bad_code.py
git commit -m "Refactor: apply AI code review suggestions

- Rename functions and variables for clarity
- Add with statement to close file handle properly
- Add error handling for API calls
- Handle empty list in calculate_average
- Simplify list merging with + operator"
```

### PR에 커밋 push

```bash
git push origin feature/user-data-processor
```

GitHub의 PR 페이지로 가면 자동으로 새 커밋이 반영됩니다.

---

## PR 설명 업데이트 예시

수정 후 PR 설명을 아래처럼 업데이트하면 리뷰어(또는 미래의 나)가 맥락을 쉽게 파악할 수 있어요.

```
## 변경 내용
- 사용자 데이터 필터링 함수 추가
- 파일 읽기 유틸리티 추가
- 점수 계산 함수 추가

## AI 코드 리뷰 반영 내용
- [수정] 함수명/변수명 명확하게 변경 (f → get_adult_names 등)
- [수정] open() 후 with 구문으로 파일 핸들 자동 닫기
- [수정] requests 에러 처리 추가 (timeout, raise_for_status)
- [수정] 빈 리스트 처리 로직 추가
- [미반영] type hint — 팀 컨벤션 미적용 상태라 보류

## 테스트
- 로컬 실행 확인 완료
- 빈 리스트, 잘못된 URL 시나리오 수동 테스트 완료
```

---

## 자주 하는 실수

| 실수 | 설명 | 올바른 방법 |
|------|------|------------|
| AI 피드백을 100% 따른다 | AI도 틀릴 수 있음 | 피드백 의도를 이해한 뒤 판단하기 |
| 코드 없이 리뷰 요청 | AI가 맥락 없이 일반론만 답함 | 코드 + 맥락 + 질문 범위를 함께 제공 |
| 에러만 고치고 PR 업데이트 안 함 | PR이 원래 상태로 남음 | 수정 후 반드시 push |
| AI 리뷰를 받고 아무것도 안 함 | 리뷰의 의미가 없음 | 반영할 것과 아닌 것을 명시적으로 정리 |
| PR 설명을 업데이트 안 함 | 리뷰어가 변경 맥락을 모름 | AI 리뷰 반영 내용을 PR 설명에 추가 |

---

## 핵심 정리

AI 코드 리뷰 워크플로를 한 줄로 요약하면:

> **코드 작성 → PR 생성 → AI 리뷰 요청 → 판단 → 수정 → 커밋 → PR 업데이트**

이 흐름을 반복할수록 코드 품질이 올라가고, AI와 협업하는 감각도 자연스럽게 생깁니다.

---

## 완료 체크리스트

- [ ] 의도적으로 문제 있는 Python 코드(`bad_code.py`)를 작성했다
- [ ] GitHub에 새 브랜치를 만들고 PR을 생성했다
- [ ] 프롬프트 템플릿을 활용해서 AI에게 코드 리뷰를 요청했다
- [ ] AI 리뷰 결과를 읽고, 반영할 것과 무시할 것을 목록으로 정리했다
- [ ] AI 피드백을 반영해서 코드를 수정했다
- [ ] 수정된 코드를 커밋하고 PR에 push했다
- [ ] PR 설명에 AI 리뷰 반영 내용을 업데이트했다

---

## 한 번 더 생각해 보기

1. AI 리뷰에서 가장 유용했던 피드백은 무엇이었나요? 왜 그 피드백이 도움이 됐나요?
2. AI가 제안했지만 반영하지 않은 피드백이 있었나요? 그 이유는 무엇이었나요?
3. AI 없이 이 코드를 처음 읽었다면, 어떤 문제를 스스로 발견할 수 있었을까요?

---

## 다음 실습

Lab 02에서는 GitHub Actions를 활용해서 PR이 생성될 때 자동으로 코드 품질 검사가 실행되는 CI 파이프라인을 만들어봅니다.
