## 이 장에서 배우는 것

- Claude Code, GitHub Copilot, Cursor 각 도구의 특성과 그에 맞는 프롬프트 전략
- 원하는 결과가 나오지 않았을 때 프롬프트를 단계적으로 개선하는 방법
- 맥락 제공 → 역할 지정 → 제약 추가, 3단계 프롬프트 개선 기법
- AI가 엉뚱한 답을 했을 때 대응하는 패턴
- AI가 생성한 코드를 검증하고 다시 피드백하는 반복 루프

---

## 먼저 쉬운 설명

AI 도구에게 "함수 만들어줘"라고만 하면 어떻게 될까요?

결과가 나오기는 합니다. 하지만 내가 원하는 것과 다를 때가 많습니다. 변수 이름이 영어인데 나는 한국어 주석을 원했다거나, 에러 처리가 없다거나, 라이브러리를 잘못 썼다거나.

이건 AI가 나쁜 게 아닙니다. **내가 충분한 정보를 주지 않았기 때문입니다.**

더 중요한 것은, 도구마다 특성이 다릅니다.

- **Claude Code**는 긴 맥락과 설계 질문에 강합니다
- **GitHub Copilot**은 현재 열린 파일의 코드를 보면서 자동 완성에 강합니다
- **Cursor**는 파일 전체를 참조하며 수정 제안에 강합니다

같은 질문이라도 도구에 따라 다르게 물어야 더 좋은 결과를 얻을 수 있습니다. 이 장에서는 그 방법을 배웁니다.

---

## 1. 도구별 특성과 프롬프트 전략

### 1.1 도구별 프롬프트 차이 비교표

| 항목 | Claude Code | GitHub Copilot | Cursor |
|------|------------|----------------|--------|
| **주요 강점** | 긴 대화, 설계, 리팩터링 | 인라인 자동 완성, 빠른 코드 제안 | 파일 전체 편집, 멀티 파일 수정 |
| **프롬프트 입력 방식** | 채팅 (자연어 대화) | 주석 또는 코드 문맥 | 채팅 + 파일 선택 (`@파일명`) |
| **맥락 전달 방법** | 대화로 직접 설명 | 코드 위에 주석으로 의도 작성 | `@` 기호로 파일 참조 |
| **잘 맞는 작업** | "이 구조를 어떻게 바꿀까?", 버그 분석 | "이 다음 줄은 뭘 써야 하지?" | "이 파일의 이 함수를 수정해줘" |
| **잘 안 맞는 작업** | 타이핑 중 실시간 완성 | 복잡한 설계 결정 | 도구 없이 단독 질문 |

### 1.2 Claude Code에 맞는 프롬프트

Claude Code는 **대화형**입니다. 파일 전체를 읽고 설계 수준의 조언을 잘 합니다.

**잘 작동하는 방식:**

```
# 좋은 예 — 맥락과 목적을 함께 설명
"지금 FastAPI로 사용자 API를 만들고 있어.
users.py 파일에 로그인 엔드포인트가 있는데,
비밀번호를 평문으로 저장하고 있어. 어떻게 고쳐야 해?"
```

```
# 안 좋은 예 — 맥락 없이 막연하게
"비밀번호 암호화 코드 짜줘"
```

### 1.3 GitHub Copilot에 맞는 프롬프트

Copilot은 **주석과 함수 이름**을 보고 다음 코드를 예측합니다. 파일 안에서 코드를 쓰는 중에 작동합니다.

```python
# 사용자 이메일 목록에서 중복을 제거하고 정렬해서 반환하는 함수
def get_unique_sorted_emails(email_list: list[str]) -> list[str]:
    # Copilot이 이 아래 코드를 자동으로 제안해 줌
```

```python
# 안 좋은 예 — 함수 이름과 주석이 없으면 Copilot이 맥락을 잡기 어려움
def f(x):
    # ???
```

### 1.4 Cursor에 맞는 프롬프트

Cursor는 **파일을 참조**시키는 것이 핵심입니다. `@파일명` 으로 여러 파일을 동시에 지정할 수 있습니다.

```
# Cursor 채팅창에서
"@users.py @database.py 를 보고,
users.py의 get_user 함수가 database.py의 connection pool을
제대로 사용하도록 수정해줘"
```

```
# 파일 지정 없이 쓰면 Cursor도 맥락을 못 잡음
"get_user 함수 고쳐줘"  # 어떤 파일? 어떻게?
```

---

## 2. 프롬프트 개선 3단계 기법

원하는 결과가 나오지 않을 때, **맥락 → 역할 → 제약** 순서로 프롬프트를 보강합니다.

### 2.1 1단계: 맥락 제공 (Context)

AI는 내 상황을 모릅니다. 지금 어떤 프로젝트인지, 어떤 언어를 쓰는지, 어떤 문제가 있는지를 알려줘야 합니다.

**개선 전:**
```
"리스트에서 중복 제거하는 코드 짜줘"
```

**개선 후 (맥락 추가):**
```
"Python 3.11로 FastAPI 프로젝트를 만들고 있어.
상품 ID 리스트(정수형)에서 중복을 제거하는 코드가 필요해.
원래 순서는 유지해야 해."
```

실제로 받게 되는 결과 차이:

```python
# 맥락 없이 받은 코드 — 순서가 바뀔 수 있음
def remove_duplicates(lst):
    return list(set(lst))  # set은 순서를 보장하지 않음

# 맥락 있는 코드 — 순서 유지
def remove_duplicates(product_ids: list[int]) -> list[int]:
    seen = set()
    result = []
    for pid in product_ids:
        if pid not in seen:
            seen.add(pid)
            result.append(pid)
    return result
```

### 2.2 2단계: 역할 지정 (Role)

AI에게 어떤 관점에서 답해야 하는지를 알려줍니다.

**개선 전 (맥락만 있음):**
```
"Python으로 API 서버 만드는 코드 짜줘"
```

**개선 후 (역할 추가):**
```
"너는 Python 백엔드 개발자야.
FastAPI로 간단한 할 일 목록 API를 만들어줘.
나는 Python 초보라서 각 코드마다 한 줄씩 설명을 달아줘."
```

역할 지정이 효과적인 상황:

| 역할 | 언제 쓰면 좋은가 |
|------|-----------------|
| "Python 초보를 가르치는 선생님처럼" | 설명이 필요할 때 |
| "시니어 코드 리뷰어처럼" | 코드 품질 점검할 때 |
| "보안 전문가처럼" | 취약점 확인할 때 |
| "이 프로젝트의 팀원처럼" | 기존 코드 스타일을 유지해야 할 때 |

### 2.3 3단계: 제약 추가 (Constraint)

AI가 너무 복잡하게 만들거나, 원하지 않는 라이브러리를 쓸 때 제약을 겁니다.

**개선 전 (역할까지 있음):**
```
"너는 Python 백엔드 개발자야.
이메일 유효성 검사 함수 만들어줘."
```

**개선 후 (제약 추가):**
```
"너는 Python 백엔드 개발자야.
이메일 유효성 검사 함수 만들어줘.

제약 조건:
- 외부 라이브러리 사용 금지 (표준 라이브러리만)
- 함수 길이 20줄 이내
- 타입 힌트 반드시 포함
- 이메일 형식이 아닐 때 ValueError 발생"
```

결과 비교:

```python
# 제약 없이 받은 코드 — 외부 라이브러리 사용
import email_validator  # pip install 필요

def validate_email(email: str) -> bool:
    try:
        email_validator.validate_email(email)
        return True
    except email_validator.EmailNotValidError:
        return False

# 제약 있는 코드 — 표준 라이브러리만 사용
import re

def validate_email(email: str) -> bool:
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    if not re.match(pattern, email):
        raise ValueError(f"유효하지 않은 이메일 형식: {email}")
    return True
```

---

## 3. AI가 엉뚱한 답을 했을 때 대응 패턴

### 3.1 엉뚱한 답의 4가지 유형과 대응

**유형 1: 질문을 오해했을 때**

```
# 내가 원한 것: 기존 함수를 수정해 달라고 했는데
# AI가 한 것: 새 함수를 처음부터 다시 작성

대응 프롬프트:
"기존 calculate_price 함수를 삭제하지 말고,
그 안에 discount_rate 파라미터만 추가해줘.
다른 부분은 그대로 유지해."
```

**유형 2: 너무 복잡하게 만들었을 때**

```python
# AI가 만든 코드 — 필요 이상으로 복잡
class UserEmailValidator:
    def __init__(self, config: ValidationConfig):
        self.config = config
        self._cache = {}
    
    def validate(self, email: str) -> ValidationResult:
        # 50줄의 코드...

# 내가 원한 것: 간단한 함수 하나

대응 프롬프트:
"너무 복잡해. 클래스 말고 그냥 함수 하나로 단순하게 만들어줘.
is_valid_email(email: str) -> bool 이 형태로."
```

**유형 3: 다른 언어나 라이브러리를 썼을 때**

```
대응 프롬프트:
"이 코드에서 asyncio를 사용했는데, 나는 비동기를 아직 모르거든.
동기(sync) 방식으로 다시 작성해줘."
```

**유형 4: 에러가 있는 코드를 줬을 때**

```
대응 프롬프트:
"이 코드를 실행했더니 이런 에러가 났어:

TypeError: unsupported operand type(s) for +: 'int' and 'str'
  File "calculator.py", line 12, in add_numbers
    return a + b

에러 메시지를 보고 어디가 문제인지 설명해주고 고쳐줘."
```

### 3.2 AI 답변 검증 체크리스트

AI가 코드를 줬을 때, 바로 쓰기 전에 확인합니다:

```
□ 실제로 실행했을 때 에러 없이 돌아가는가?
□ 내가 요청한 기능이 실제로 구현됐는가?
□ 외부 라이브러리가 포함됐다면 설치 가능한가?
□ 내 프로젝트의 Python 버전과 호환되는가?
□ 보안에 문제가 없는가? (예: SQL 직접 삽입, 평문 비밀번호)
```

---

## 4. 프롬프트 개선 전/후 비교 예시

### 예시 1: 파일 읽기 함수

**Before:**
```
"파일 읽는 함수 만들어줘"
```

**After:**
```
"Python으로 CSV 파일을 읽어서
딕셔너리 리스트로 반환하는 함수를 만들어줘.
파일이 없으면 FileNotFoundError 대신 빈 리스트를 반환해.
인코딩은 UTF-8로 고정."
```

```python
# Before — 너무 범용적
def read_file(path):
    with open(path, 'r') as f:
        return f.read()

# After — 요구사항 정확히 반영
import csv
from pathlib import Path

def read_csv_as_dicts(file_path: str) -> list[dict]:
    path = Path(file_path)
    if not path.exists():
        return []
    
    with open(path, encoding='utf-8', newline='') as f:
        reader = csv.DictReader(f)
        return list(reader)
```

### 예시 2: API 호출 함수

**Before:**
```
"날씨 API 호출하는 코드 짜줘"
```

**After:**
```
"Python requests 라이브러리로
OpenWeatherMap API를 호출하는 함수를 만들어줘.

- 함수 시그니처: get_weather(city: str, api_key: str) -> dict
- API 호출 실패 시 None 반환 (예외 발생 금지)
- 타임아웃은 5초로 설정
- 응답에서 온도(temp)와 날씨 설명(description)만 추출해서 반환"
```

```python
# Before — 에러 처리 없음, 타임아웃 없음
import requests

def get_weather(city):
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}"
    return requests.get(url).json()

# After — 안전하고 명확한 코드
import requests

def get_weather(city: str, api_key: str) -> dict | None:
    url = "https://api.openweathermap.org/data/2.5/weather"
    params = {"q": city, "appid": api_key, "units": "metric"}
    
    try:
        response = requests.get(url, params=params, timeout=5)
        response.raise_for_status()
        data = response.json()
        return {
            "temp": data["main"]["temp"],
            "description": data["weather"][0]["description"]
        }
    except requests.RequestException:
        return None
```

### 예시 3: 데이터 검증 함수

**Before:**
```
"사용자 입력 검증해줘"
```

**After:**
```
"회원가입 폼의 사용자 입력을 검증하는 함수를 만들어줘.

검증 항목:
- 이름: 2~20자, 한글 또는 영문만
- 나이: 1~150 사이 정수
- 이메일: 기본 이메일 형식

검증 실패 시 어떤 항목이 왜 실패했는지 알 수 있는 딕셔너리를 반환해.
예: {'name': '이름은 2자 이상이어야 합니다'}"
```

```python
# Before — 뭘 검증하는지 불명확
def validate_user(data):
    if not data:
        return False
    return True

# After — 명확한 검증 로직과 에러 메시지
import re

def validate_signup_form(name: str, age: int, email: str) -> dict[str, str]:
    errors = {}
    
    if not 2 <= len(name) <= 20:
        errors["name"] = "이름은 2자 이상 20자 이하여야 합니다"
    elif not re.match(r'^[가-힣a-zA-Z]+$', name):
        errors["name"] = "이름은 한글 또는 영문만 사용할 수 있습니다"
    
    if not isinstance(age, int) or not 1 <= age <= 150:
        errors["age"] = "나이는 1~150 사이의 정수여야 합니다"
    
    if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', email):
        errors["email"] = "올바른 이메일 형식이 아닙니다"
    
    return errors
```

---

## 따라 하기 실습

### 실습 1: 프롬프트 단계별 개선해보기

**목표:** 같은 문제를 3단계로 프롬프트를 보강하며 결과가 어떻게 달라지는지 체험한다.

**파일:** `practice/prompt_practice.py`

**1단계 — 맥락 없이 질문해보기:**

Claude Code 또는 Cursor에 아래 프롬프트를 그대로 입력하세요.

```
"주문 금액 계산 함수 만들어줘"
```

결과를 메모해 두세요. 어떤 코드가 나왔나요?

**2단계 — 맥락 추가:**

```
"Python으로 쇼핑몰 주문 금액을 계산하는 함수를 만들어줘.
상품 가격과 수량을 받아서 총액을 반환하는데,
5만 원 이상이면 배송비가 무료야. 미만이면 배송비 3000원 추가."
```

**3단계 — 역할과 제약 추가:**

```
"너는 Python 초보를 가르치는 선생님이야.
Python으로 쇼핑몰 주문 금액을 계산하는 함수를 만들어줘.

요구사항:
- 함수 시그니처: calculate_order_total(price: int, quantity: int) -> int
- 총액 = 가격 × 수량
- 5만 원 이상이면 배송비 0원, 미만이면 3000원 추가
- 타입 힌트 포함
- 각 줄마다 한국어 주석

제약:
- 외부 라이브러리 사용 금지
- 10줄 이내"
```

세 단계 결과를 비교해서 `practice/prompt_practice.py` 안에 주석으로 기록하세요:

```python
# 1단계 결과 (맥락 없음)
# ...

# 2단계 결과 (맥락 추가)
# ...

# 3단계 결과 (역할 + 제약 추가)
def calculate_order_total(price: int, quantity: int) -> int:
    # 총 상품 금액 계산
    subtotal = price * quantity
    # 5만 원 미만이면 배송비 추가
    shipping = 0 if subtotal >= 50000 else 3000
    return subtotal + shipping
```

---

### 실습 2: 엉뚱한 답 수정하기

**목표:** AI가 잘못된 코드를 줬을 때 어떻게 수정 요청하는지 연습한다.

**파일:** `practice/fix_prompt.py`

아래 코드는 AI가 생성한 코드입니다. **의도적으로 문제가 있습니다.**

```python
# AI가 생성한 코드 — 문제 있음
def calculate_discount(price, discount_rate):
    return price - discount_rate  # 할인율이 아니라 할인금액으로 빼버림

def get_final_price(price, discount_percent):
    discounted = calculate_discount(price, discount_percent)
    return discounted
```

**Step 1:** 이 코드를 실행해서 에러를 확인하세요.

```python
print(get_final_price(10000, 10))  # 10% 할인이면 9000이 나와야 하는데?
# 결과: 9990 (틀림! 10%가 아니라 10원을 뺐음)
```

**Step 2:** AI에게 수정 요청 프롬프트를 작성하세요.

```
"이 코드에 버그가 있어.

calculate_discount(10000, 10)을 호출하면
10%를 할인한 9000이 나와야 하는데, 9990이 나오고 있어.

discount_rate는 퍼센트 값이야 (예: 10 → 10%).
올바르게 계산하도록 고쳐줘."
```

**Step 3:** 수정된 코드를 `practice/fix_prompt.py`에 저장하고 테스트하세요.

```python
def calculate_discount(price: int, discount_rate: int) -> int:
    # discount_rate는 퍼센트 (예: 10 → 10%)
    discount_amount = price * discount_rate // 100
    return price - discount_amount

def get_final_price(price: int, discount_percent: int) -> int:
    return calculate_discount(price, discount_percent)

# 테스트
assert get_final_price(10000, 10) == 9000, "10% 할인 테스트 실패"
assert get_final_price(10000, 0) == 10000, "할인 없음 테스트 실패"
assert get_final_price(10000, 100) == 0, "100% 할인 테스트 실패"
print("모든 테스트 통과!")
```

---

### 실습 3: 도구별 프롬프트 차이 체험하기

**목표:** Claude Code와 Copilot(또는 Cursor)에 같은 요구사항을 다른 방식으로 전달해본다.

**파일:** `practice/tool_comparison.py`

**요구사항:** "학생 점수 리스트를 받아서 평균, 최고점, 최저점을 반환하는 함수"

**Claude Code용 프롬프트 (채팅창에 입력):**
```
"Python으로 학생 점수를 분석하는 함수를 만들어줘.

- 함수 시그니처: analyze_scores(scores: list[int]) -> dict
- 반환값: {'average': float, 'max': int, 'min': int}
- 빈 리스트 입력 시 ValueError 발생
- 소수점 첫째 자리까지만 반올림"
```

**Copilot용 프롬프트 (파일 안에 주석으로 작성):**
```python
# 학생 점수 리스트를 받아서 평균(소수점 1자리), 최고점, 최저점을 딕셔너리로 반환
# 빈 리스트면 ValueError 발생
# 반환 형태: {'average': float, 'max': int, 'min': int}
def analyze_scores(scores: list[int]) -> dict:
    # Copilot이 여기서 자동 완성을 제안합니다
```

두 방식의 결과를 `practice/tool_comparison.py`에 나란히 저장하고, 어떤 차이가 있었는지 주석으로 기록하세요.

---

## 자주 하는 실수

| 실수 | 증상 / 에러 메시지 | 해결 방법 |
|------|-----------------|----------|
| 맥락 없이 질문 | 코드는 나오는데 내 상황과 전혀 다른 코드 | "나는 지금 [프로젝트 종류]를 만들고 있어. [언어/버전]. [원하는 것]" 형식으로 맥락 추가 |
| 에러 없이 결과만 복사 | `TypeError`, `NameError`, `ModuleNotFoundError` 등 | 항상 실행해서 에러 확인 후, 에러 메시지 그대로 AI에게 붙여 넣기 |
| 너무 많은 요구사항 한 번에 | 코드가 너무 복잡하거나 일부만 구현됨 | 요구사항을 하나씩 단계별로 요청 |
| "고쳐줘"만 반복 | AI가 같은 코드를 또 생성 | 어떤 부분이 왜 문제인지 구체적으로 설명 |
| AI 코드를 테스트 없이 사용 | 나중에 예상치 못한 버그 발생 | 받은 코드는 반드시 경계값 포함해서 직접 실행 |
| Cursor에서 파일 지정 안 함 | "어떤 파일을 수정해야 하나요?" 라고 되묻거나 엉뚱한 파일 수정 | `@파일명` 으로 대상 파일을 명시 |
| Copilot 주석이 영어 | 제안 코드가 영어 변수명, 영어 주석으로 가득 | 주석을 한국어로 쓰면 한국어 스타일 코드 제안 확률이 높아짐 |

---

## 확인 체크리스트

```
□ Claude Code, Copilot, Cursor의 주요 특성 차이를 말할 수 있다
□ 맥락(Context) 없이 쓴 프롬프트와 맥락 있는 프롬프트의 결과 차이를 체험했다
□ 역할 지정(Role)이 답변 방식을 어떻게 바꾸는지 이해했다
□ 제약 추가(Constraint)로 원하는 코드 형태를 유도해봤다
□ AI 코드에 에러가 생겼을 때 에러 메시지를 그대로 붙여 넣어 수정 요청해봤다
□ 실습 1~3의 파일(prompt_practice.py, fix_prompt.py, tool_comparison.py)을 실행해서 에러 없이 돌아간다
□ AI가 생성한 코드를 assert 또는 직접 실행으로 검증해봤다
```

---

## 한 번 더 생각해 보기

1. Claude Code에게 역할을 "시니어 코드 리뷰어"로 지정하면 어떤 답변이 달라질까요? 같은 코드를 "초보 개발자에게 설명하는 선생님"으로 물어보면 어떻게 다를지 직접 비교해보세요.

2. 같은 프롬프트를 Claude Code와 Cursor에 각각 입력했을 때 결과가 달랐다면, 그 차이는 도구의 특성 때문인가요, 아니면 프롬프트를 도구에 맞게 조정하지 않았기 때문인가요?

3. 이 장에서 배운 "맥락 → 역할 → 제약" 3단계 기법 중, 여러분이 지금까지 가장 자주 빠뜨렸던 단계는 무엇인가요? 다음 AI 대화에서 의식적으로 그 단계를 추가해보세요.

---

## 다음 장

다음 장에서는 AI가 생성한 코드를 팀 프로젝트에 통합하고, GitHub을 통해 협업하는 Vibe Coding 워크플로우를 배웁니다.