# Chapter 03: AI 리뷰 워크플로우 — PR에서 AI의 도움받기

## 이 장에서 배우는 것

- AI 코드 리뷰가 무엇인지, 왜 혼자 만든 코드에 꼭 필요한지
- GitHub Copilot PR 리뷰와 Claude/ChatGPT 붙여넣기 방식의 차이
- Pull Request를 만들고 AI에게 리뷰를 요청하는 전체 흐름
- AI 리뷰 결과를 읽고 반영할 것과 무시할 것을 구분하는 방법
- AI가 자주 지적하는 6가지 패턴
- AI 리뷰를 받아 코드를 수정하고 re-push하는 실습

---

## 왜 필요한가 — AI 코드 리뷰가 있어야 하는 이유

### 혼자 만든 코드의 사각지대

코드를 혼자 오래 들여다보면 이런 일이 생긴다.

- 오타가 있어도 눈에 들어오지 않는다
- "이 정도면 이해하겠지"라고 생각하고 넘어간 부분이 나중에 문제가 된다
- 변수 이름을 `data`, `temp`, `x`처럼 지어도 지금은 맥락을 알기 때문에 괜찮아 보인다
- 예외 처리를 빠뜨려도 "이 상황에서 오류가 날 일은 없겠지"라고 생각한다

이것이 **사각지대**다. 작성자는 코드 의도를 이미 알고 있기 때문에 코드만 보고도 맥락을 채워 읽는다. 하지만 처음 보는 사람이나 6개월 후의 자신은 그 맥락을 모른다.

사람 리뷰어가 있으면 이 사각지대를 잡을 수 있지만, 항상 즉시 리뷰해 줄 사람이 있는 것은 아니다. 혼자 공부할 때, 사이드 프로젝트를 할 때, 빠르게 검토가 필요할 때 — AI 리뷰는 이 공백을 채워준다.

### AI 리뷰가 잡아주는 것

AI는 아래를 빠르게 잡는다.

- 명확하지 않은 변수명이나 함수명
- 예외 처리가 빠진 부분
- 중복된 로직
- 잠재적인 보안 취약점 (하드코딩된 비밀번호, SQL 인젝션 등)
- 명백한 성능 문제 (루프 안에서 반복 쿼리 등)
- 테스트가 없는 주요 함수

AI 리뷰는 사람 리뷰를 대체하지 않는다. 하지만 사람이 리뷰하기 전에 기본적인 품질 문제를 먼저 걸러주는 **1차 필터** 역할을 한다.

---

## AI 리뷰 방법 2가지 비교

### 방법 1: GitHub Copilot PR 리뷰

GitHub Copilot이 Pull Request에 직접 통합된 리뷰 기능이다.

| 항목 | 내용 |
|---|---|
| 사용 방법 | PR 페이지 → Summary 탭 → "Copilot summary" 자동 생성 또는 리뷰 요청 |
| 장점 | GitHub와 완전히 통합, PR diff를 자동으로 분석, 코드 라인 단위로 코멘트 |
| 단점 | GitHub Copilot 유료 구독 필요 (월 $10~), 리뷰 깊이가 제한적일 수 있음 |
| 적합한 상황 | 이미 Copilot을 사용 중인 팀, CI/CD에 자동 리뷰를 통합하고 싶을 때 |

### 방법 2: Claude / ChatGPT에 코드 붙여넣기

별도 AI 서비스에 코드를 복사해 리뷰를 요청하는 방식이다.

| 항목 | 내용 |
|---|---|
| 사용 방법 | 코드를 복사 → Claude.ai 또는 ChatGPT 열기 → 리뷰 프롬프트와 함께 붙여넣기 |
| 장점 | 무료(기본 플랜), 리뷰 요청 방식을 커스터마이징 가능, 설명을 대화 형태로 이어갈 수 있음 |
| 단점 | GitHub와 직접 통합 안 됨, 수동으로 코드를 복사해야 함 |
| 적합한 상황 | 혼자 공부할 때, 특정 부분에 대해 깊이 있는 설명을 원할 때 |

**초보자 권장:** GitHub Copilot 구독이 없다면 Claude.ai나 ChatGPT에 붙여넣는 방식으로 시작한다. 비용 없이 바로 시작할 수 있다.

---

## PR에서 AI 리뷰를 받는 전체 흐름

아래 5단계가 이 장의 핵심 흐름이다.

```
1. 코드 작성 (의도적으로 문제 포함)
   ↓
2. 브랜치 생성 + 커밋 + Push
   ↓
3. GitHub에서 Pull Request 생성
   ↓
4. AI에게 PR 코드 붙여넣기 + 리뷰 요청
   ↓
5. 리뷰 결과 읽기 → 반영할 것 수정 → re-push
```

---

## AI가 자주 지적하는 6가지 패턴

AI 리뷰를 받을 때 어떤 것들이 지적될지 미리 알면 코드를 쓸 때도 더 주의하게 된다.

### 패턴 1: 변수명·함수명이 의미를 담지 않음

**나쁜 예:**
```python
def calc(a, b):
    x = a * b * 0.1
    return x
```

**AI가 지적하는 이유:** `calc`, `a`, `b`, `x`가 무엇을 의미하는지 코드만 보고 알 수 없다.

**좋은 예:**
```python
def calculate_discount(price, quantity):
    discount_amount = price * quantity * 0.1
    return discount_amount
```

### 패턴 2: 예외 처리 누락

**나쁜 예:**
```python
def get_user(user_id):
    user = db.query(f"SELECT * FROM users WHERE id={user_id}")
    return user[0]  # user가 없으면 IndexError 발생
```

**AI가 지적하는 이유:** 결과가 없을 때 `IndexError`가 발생하고, 호출하는 쪽에서 이를 처리할 방법이 없다.

**좋은 예:**
```python
def get_user(user_id):
    users = db.query(f"SELECT * FROM users WHERE id={user_id}")
    if not users:
        return None
    return users[0]
```

### 패턴 3: 중복 코드

**나쁜 예:**
```python
def send_welcome_email(email):
    subject = "환영합니다"
    body = f"안녕하세요! {email}님, 가입을 환영합니다."
    smtp.send(email, subject, body)

def send_password_reset_email(email):
    subject = "비밀번호 재설정"
    body = f"안녕하세요! {email}님, 비밀번호를 재설정하려면 클릭하세요."
    smtp.send(email, subject, body)
```

**AI가 지적하는 이유:** `smtp.send` 호출 패턴이 중복된다. 이메일 전송 로직을 함수로 분리해야 한다.

### 패턴 4: 보안 취약점

**나쁜 예:**
```python
password = "admin1234"  # 코드에 비밀번호 하드코딩
db_url = "mysql://root:password@localhost/mydb"
```

**AI가 지적하는 이유:** 코드가 GitHub에 올라가면 비밀번호가 공개된다. 환경변수로 분리해야 한다.

**좋은 예:**
```python
import os
password = os.environ.get("DB_PASSWORD")
db_url = os.environ.get("DATABASE_URL")
```

### 패턴 5: 성능 문제

**나쁜 예:**
```python
# 루프 안에서 매번 데이터베이스 쿼리 (N+1 문제)
for user_id in user_ids:
    user = db.query(f"SELECT * FROM users WHERE id={user_id}")
    print(user.name)
```

**AI가 지적하는 이유:** 사용자가 100명이면 데이터베이스 쿼리가 100번 실행된다. 한 번에 모두 조회하는 것이 훨씬 효율적이다.

### 패턴 6: 테스트 누락

**AI가 지적하는 이유:** 핵심 비즈니스 로직에 테스트 코드가 없으면 나중에 수정했을 때 무언가 깨졌는지 알 수 없다.

AI는 "이 함수에 대한 단위 테스트가 있어야 할 것 같습니다"라고 제안한다.

---

## 실습 1: 의도적으로 문제 있는 코드로 PR 만들고 AI 리뷰 요청하기

### 준비

- GitHub 계정
- git이 설치된 컴퓨터 또는 GitHub 웹 편집기

### 단계 1: 저장소에 브랜치 생성

GitHub 저장소 페이지에서:
1. 브랜치 선택 드롭다운(기본: `main`) 클릭
2. 새 브랜치 이름 입력: `feat/user-calculator`
3. **Create branch: feat/user-calculator** 클릭

또는 터미널에서:
```bash
git checkout -b feat/user-calculator
```

### 단계 2: 의도적으로 문제 있는 파일 작성

저장소에 `calculator.py` 파일을 아래 내용으로 만든다. 이 코드에는 AI가 지적할 여러 문제가 포함되어 있다.

```python
# calculator.py
# 의도적으로 나쁜 코드 예시 (AI 리뷰 실습용)

secret_key = "mysecretkey123"  # 하드코딩된 비밀

def calc(a, b, t):
    if t == "add":
        return a + b
    if t == "sub":
        return a - b
    if t == "mul":
        return a * b
    if t == "div":
        return a / b  # b가 0이면 ZeroDivisionError 발생

def process(nums):
    result = []
    for x in nums:
        r = calc(x, 2, "mul")  # 의미 불분명
        result.append(r)
    return result

def process2(nums):  # process와 거의 동일한 중복 코드
    result = []
    for x in nums:
        r = calc(x, 3, "mul")
        result.append(r)
    return result
```

### 단계 3: 커밋하고 Push

터미널에서:
```bash
git add calculator.py
git commit -m "Add calculator module"
git push origin feat/user-calculator
```

GitHub 웹 편집기를 사용하는 경우: 파일 편집 후 "Commit changes" 클릭.

### 단계 4: Pull Request 생성

1. GitHub 저장소 페이지로 이동
2. "feat/user-calculator had recent pushes" 알림 옆 **Compare & pull request** 버튼 클릭
3. PR 제목: `feat: 계산기 모듈 추가`
4. PR 설명:

```text
## 변경 내용
- 기본 사칙연산 계산기 모듈 추가
- 숫자 리스트에 일괄 연산을 적용하는 process 함수 추가

## 리뷰 요청
- 코드 품질 및 잠재적 문제 확인 부탁드립니다
```

5. **Create pull request** 클릭

### 단계 5: AI에게 리뷰 요청하기

PR 페이지에서 **Files changed** 탭을 클릭하면 추가된 코드 전체가 보인다.

이 코드를 복사한 뒤, Claude.ai 또는 ChatGPT에서 아래 프롬프트와 함께 붙여넣는다.

```text
아래 Python 코드를 코드 리뷰 관점에서 검토해 주세요.
초보 개발자가 작성한 코드입니다.

검토 항목:
1. 변수명/함수명의 명확성
2. 예외 처리 누락 여부
3. 중복 코드
4. 보안 취약점
5. 성능 문제
6. 테스트 필요 부분

각 문제에 대해 (1) 문제 설명, (2) 개선 코드를 함께 작성해 주세요.

[코드 붙여넣기]
```

> **막히는 지점:** 코드를 그냥 붙여넣으면 AI가 어떤 관점에서 리뷰해야 할지 명확하지 않을 수 있다. 위처럼 검토 항목을 미리 나열하면 더 구체적인 피드백을 받을 수 있다.

---

## AI 리뷰 결과 읽는 방법

### AI 리뷰 결과 예시

위 코드에 대해 AI는 보통 이런 피드백을 준다.

```
1. [보안] secret_key가 코드에 하드코딩되어 있습니다.
   → os.environ.get("SECRET_KEY")로 환경변수에서 읽어야 합니다.

2. [예외 처리] calc 함수에서 t="div"일 때 b=0이면 ZeroDivisionError가 발생합니다.
   → if b == 0: raise ValueError("0으로 나눌 수 없습니다") 처리를 추가하세요.

3. [변수명] calc, a, b, t, x, r 등 의미를 알 수 없는 변수명이 많습니다.
   → calculate_operation, first_number, second_number, operation_type 처럼
      의미가 담긴 이름을 사용하세요.

4. [중복 코드] process와 process2가 거의 동일합니다.
   → multiplier 파라미터를 받는 함수 하나로 통합하세요.

5. [테스트 누락] 핵심 계산 로직에 단위 테스트가 없습니다.
   → pytest를 사용한 기본 테스트를 추가하는 것을 권장합니다.
```

### 반영할 것과 무시할 것 구분하기

AI 리뷰가 모두 옳은 것은 아니다. 아래 기준으로 판단한다.

**반드시 반영해야 하는 것:**
- 보안 취약점 (하드코딩된 비밀번호, API 키)
- 런타임 오류 가능성 (ZeroDivisionError, IndexError 등 예외 미처리)
- 코드가 동작하지 않는 버그

**반영하는 것이 좋은 것:**
- 변수명 개선 (팀 컨벤션이 없는 경우)
- 중복 코드 제거
- 명확성이 낮은 주석이나 설명

**상황에 따라 무시해도 되는 것:**
- 스타일 취향 문제 (AI가 선호하는 방식이 팀 규칙과 다를 때)
- "이렇게 하면 더 좋다"는 수준의 제안 (지금 당장 리팩토링이 필요 없을 때)
- 프로토타입 코드에 테스트를 추가하라는 제안 (아직 설계가 확정되지 않았을 때)

---

## 실습 2: AI 리뷰 결과 반영해서 코드 수정 후 re-push

### 단계 1: 수정된 코드 작성

AI 리뷰를 반영해 `calculator.py`를 아래처럼 수정한다.

```python
# calculator.py
# AI 리뷰 반영 버전

import os

# 비밀 정보는 환경변수에서 읽음
secret_key = os.environ.get("SECRET_KEY")

def calculate_operation(first_number, second_number, operation_type):
    """두 숫자에 대해 사칙연산을 수행한다."""
    if operation_type == "add":
        return first_number + second_number
    if operation_type == "subtract":
        return first_number - second_number
    if operation_type == "multiply":
        return first_number * second_number
    if operation_type == "divide":
        if second_number == 0:
            raise ValueError("0으로 나눌 수 없습니다")
        return first_number / second_number
    raise ValueError(f"알 수 없는 연산 유형입니다: {operation_type}")

def apply_multiplier_to_list(numbers, multiplier):
    """숫자 리스트의 각 항목에 multiplier를 곱한 결과를 반환한다."""
    return [calculate_operation(n, multiplier, "multiply") for n in numbers]
```

### 단계 2: 커밋 메시지에 AI 리뷰 반영 내용 명시

```bash
git add calculator.py
git commit -m "refactor: AI 리뷰 반영 — 변수명 개선, 예외 처리 추가, 중복 코드 제거"
git push origin feat/user-calculator
```

### 단계 3: PR에서 변경 확인

1. GitHub의 PR 페이지로 이동
2. **Files changed** 탭에서 before/after diff 확인
3. PR 댓글에 변경 내용 요약을 남긴다

PR 댓글 예시:
```text
AI 리뷰 반영 완료.

반영한 항목:
- secret_key 환경변수로 이동
- divide 연산에 ZeroDivisionError 예외 처리 추가
- 의미 없는 변수명 전부 개선 (calc → calculate_operation 등)
- process/process2 중복 → apply_multiplier_to_list 하나로 통합

반영하지 않은 항목:
- 테스트 추가 제안: 현재 프로토타입 단계라 우선순위가 낮음. 다음 PR에서 추가 예정.
```

> **막히는 지점:** re-push 후 PR 페이지가 자동으로 업데이트되지 않는 것처럼 보일 수 있다. 브라우저를 새로고침하면 새 커밋이 반영된 것을 확인할 수 있다.

---

## 실습 3: AI 리뷰를 무시해야 할 때 판단 기준

AI가 지적했다고 해서 모두 반영해야 하는 것은 아니다. 이 실습에서는 AI 리뷰를 무시하는 판단을 연습한다.

### 상황 1: AI가 스타일을 강제할 때

AI 리뷰:
```
list comprehension 대신 for 루프를 사용하는 것이 가독성이 높습니다.
```

판단 기준: 팀 코딩 컨벤션이 list comprehension을 허용한다면, 또는 나 혼자 하는 프로젝트라면 무시해도 된다. AI의 스타일 선호는 절대적이지 않다.

**무시하는 방법:** PR 댓글에 "이 부분은 팀 컨벤션에 따라 유지합니다"라고 남긴다. 이유를 기록해 두는 것이 중요하다.

### 상황 2: AI가 과도한 추상화를 제안할 때

AI 리뷰:
```
이 두 함수를 Strategy 패턴으로 리팩토링하면 확장성이 높아집니다.
```

판단 기준: 지금 만드는 것이 5가지 연산을 지원하는 간단한 계산기라면, Strategy 패턴은 과도하다. "지금 필요하지 않은 복잡성"은 무시해도 된다.

### 상황 3: AI가 잘못된 정보를 줄 때

AI 리뷰:
```
Python에서 / 연산자는 정수 나눗셈을 수행하므로 // 로 변경해야 합니다.
```

판단 기준: Python 3에서 `/`는 항상 float 나눗셈을 한다. `//`는 정수 나눗셈이다. AI가 틀렸다. 이 경우 반영하지 않고 AI에게 바로잡아 주는 답변을 남긴다.

**무시하는 방법:** "Python 3 기준에서 `/`는 float 나눗셈입니다. 이 코드는 float 결과가 필요하므로 `/`를 유지합니다"라고 기록한다.

### AI 리뷰 판단 3단계 체크

AI 리뷰를 하나씩 볼 때 아래 순서로 판단한다.

```
1. 이 지적이 버그나 보안 취약점인가?
   → YES: 반드시 반영
   → NO: 2단계로

2. 이 지적이 코드를 더 명확하게 만드는가?
   → YES: 반영 권장
   → NO: 3단계로

3. 이 지적이 현재 상황에 필요한가?
   → YES: 반영 고려
   → NO: 무시하고 이유를 PR에 기록
```

---

## 자주 하는 실수

- AI 리뷰를 보지 않고 그냥 merge 하기
- AI가 지적한 것을 이유 없이 모두 반영하기 (AI가 틀릴 수도 있다)
- AI 리뷰만 받고 사람 리뷰를 생략하기 (초보자 교육 자료, 팀 코드는 사람 리뷰가 필수)
- 무시하기로 한 리뷰를 기록 없이 버리기 (왜 무시했는지 PR에 남겨야 한다)
- 한 번에 너무 많은 코드를 PR로 올려 AI도 리뷰하기 어렵게 만들기

---

## 확인 체크리스트

- [ ] AI 코드 리뷰가 왜 필요한지 한 문장으로 설명할 수 있다
- [ ] GitHub Copilot PR 리뷰와 붙여넣기 방식의 차이를 알고 있다
- [ ] 의도적으로 문제 있는 코드로 PR을 만들었다
- [ ] AI에게 리뷰 프롬프트를 사용해서 피드백을 받았다
- [ ] AI 리뷰 중 반영할 것과 무시할 것을 구분했다
- [ ] 수정된 코드를 커밋하고 같은 브랜치에 re-push했다
- [ ] PR 댓글에 AI 리뷰 반영/무시 이유를 기록했다

---

## 한 번 더 생각해 보기

1. AI 리뷰와 사람 리뷰 중 한 가지만 선택해야 한다면 어떤 것을 선택할 것인가? 각각이 더 잘하는 것이 무엇인지 생각해 보자.

2. AI가 코드 리뷰에서 "이렇게 하면 더 좋다"라고 제안할 때, 그 제안이 현재 상황에 맞는지 어떻게 판단할 수 있을까?

3. 6개월 후 다시 이 코드를 열었을 때 쉽게 이해할 수 있으려면 지금 무엇을 챙겨야 할까?

---

## 다음 장

다음 장(Chapter 04)에서는 AI가 생성한 Python 초안을 직접 읽고 수정하는 연습을 한다. AI가 만든 코드를 그대로 쓰지 않고, 목적에 맞게 다듬는 과정을 배운다.
