## 이 장에서 배우는 것

- 리팩터링이 무엇인지, 왜 코드 품질이 중요한지 이해한다
- AI(Claude)와 함께 기존 코드를 더 읽기 쉽고 유지보수하기 좋게 개선한다
- 함수 분리, 변수명 개선, 중복 제거 등 기본 리팩터링 기법을 익힌다
- AI에게 리팩터링을 요청하는 효과적인 프롬프트 패턴을 배운다
- 리팩터링 전후를 비교하며 코드 품질 변화를 직접 확인한다

---

## 먼저 쉬운 설명

코드를 처음 짤 때는 일단 "동작하게" 만드는 데 집중합니다. 그런데 시간이 지나면 이런 생각이 듭니다.

> *"이게 뭘 하는 코드였지? 내가 짠 건데 왜 모르지?"*

이렇게 나중에 읽기 어려워진 코드를 **개선하는 작업**을 **리팩터링(Refactoring)** 이라고 합니다. 기능은 바꾸지 않으면서, 코드의 구조와 가독성을 좋게 만드는 과정입니다.

혼자 하면 어디서 시작해야 할지 막막하지만, **AI와 함께하면** 막막함이 줄어듭니다. AI는 코드를 보고 "이 부분이 복잡하네요, 이렇게 나눠보는 건 어떨까요?" 하고 제안해 줍니다. 여러분은 그 제안을 이해하고, 적용하고, 더 나은 코드를 만들어 가면 됩니다.

---

## 1. 리팩터링이란 무엇인가

리팩터링은 **"동작은 그대로, 코드 모양만 개선"** 하는 것입니다.

아래 두 코드는 같은 일을 합니다. 그런데 느낌이 다르죠?

**리팩터링 전 — `before_order.py`**

```python
def f(a, b, c):
    x = a * b
    if c == 1:
        x = x - (x * 0.1)
    elif c == 2:
        x = x - (x * 0.2)
    elif c == 3:
        x = x - (x * 0.3)
    return x

print(f(5, 3000, 2))
```

**리팩터링 후 — `after_order.py`**

```python
DISCOUNT_RATES = {1: 0.1, 2: 0.2, 3: 0.3}

def calculate_order_price(quantity: int, unit_price: int, grade: int) -> float:
    total = quantity * unit_price
    discount = DISCOUNT_RATES.get(grade, 0.0)
    return total * (1 - discount)

print(calculate_order_price(quantity=5, unit_price=3000, grade=2))
```

두 번째 코드는 변수명만 봐도 무엇을 계산하는지 알 수 있습니다. 이게 리팩터링의 핵심입니다.

---

## 2. AI에게 리팩터링 요청하는 법

AI에게 막연하게 "이 코드 고쳐줘"라고 하면 결과가 들쭉날쭉합니다. **구체적인 목표**를 함께 말해야 좋은 결과를 얻습니다.

**효과적인 프롬프트 패턴**

| 나쁜 프롬프트 | 좋은 프롬프트 |
|---|---|
| "코드 고쳐줘" | "변수명을 의미 있게 바꿔줘" |
| "더 좋게 만들어줘" | "이 함수가 너무 길어. 역할별로 분리해줘" |
| "깔끔하게 해줘" | "중복된 if-elif 블록을 딕셔너리로 바꿔줘" |

**실제 프롬프트 예시 — `score_calculator.py`**

```python
# AI에게 이렇게 요청해보세요:
# "아래 코드에서 함수 이름과 변수 이름이 너무 짧아서 의미를 알기 어려워.
#  PEP 8 스타일에 맞게 이름을 개선하고, 반복되는 로직은 함수로 분리해줘."

def calc(s1, s2, s3, b=False):
    t = s1 + s2 + s3
    if b:
        t = t * 1.1
    avg = t / 3
    if avg >= 90:
        g = 'A'
    elif avg >= 80:
        g = 'B'
    elif avg >= 70:
        g = 'C'
    else:
        g = 'F'
    return avg, g
```

**AI가 제안한 리팩터링 결과**

```python
def get_grade(average: float) -> str:
    if average >= 90:
        return 'A'
    elif average >= 80:
        return 'B'
    elif average >= 70:
        return 'C'
    return 'F'


def calculate_final_score(
    score1: float,
    score2: float,
    score3: float,
    is_bonus_eligible: bool = False,
) -> tuple[float, str]:
    total = score1 + score2 + score3
    if is_bonus_eligible:
        total *= 1.1
    average = total / 3
    grade = get_grade(average)
    return average, grade
```

---

## 3. 자주 쓰는 리팩터링 기법 세 가지

### 3-1. 함수 분리 (Extract Function)

하나의 함수가 너무 많은 일을 할 때, 역할별로 쪼갭니다.

**분리 전 — `user_report.py`**

```python
def process_user(users):
    result = []
    for u in users:
        # 유효성 검사
        if not u.get('name') or not u.get('email'):
            continue
        if '@' not in u['email']:
            continue
        # 점수 계산
        score = u.get('score', 0)
        if score >= 80:
            level = '우수'
        elif score >= 60:
            level = '보통'
        else:
            level = '미흡'
        result.append({'name': u['name'], 'level': level})
    return result
```

**분리 후**

```python
def is_valid_user(user: dict) -> bool:
    if not user.get('name') or not user.get('email'):
        return False
    return '@' in user['email']


def get_level(score: int) -> str:
    if score >= 80:
        return '우수'
    elif score >= 60:
        return '보통'
    return '미흡'


def process_users(users: list[dict]) -> list[dict]:
    return [
        {'name': u['name'], 'level': get_level(u.get('score', 0))}
        for u in users
        if is_valid_user(u)
    ]
```

### 3-2. 매직 넘버 제거 (Replace Magic Number)

코드 속 의미 없어 보이는 숫자를 상수로 바꿉니다.

```python
# 리팩터링 전 — magic_number_before.py
def check_password(password):
    return len(password) >= 8 and len(password) <= 20

# 리팩터링 후 — magic_number_after.py
MIN_PASSWORD_LENGTH = 8
MAX_PASSWORD_LENGTH = 20

def check_password(password: str) -> bool:
    return MIN_PASSWORD_LENGTH <= len(password) <= MAX_PASSWORD_LENGTH
```

### 3-3. 중복 코드 제거 (DRY — Don't Repeat Yourself)

같은 로직이 두 곳 이상 반복되면, 함수 하나로 묶습니다.

```python
# 리팩터링 전 — duplicate_before.py
def send_welcome_email(name, email):
    subject = f"안녕하세요, {name}님!"
    body = f"가입을 환영합니다, {name}님.\n문의: support@example.com"
    print(f"[이메일 발송] to={email}, subject={subject}")
    print(body)

def send_reset_email(name, email):
    subject = f"비밀번호 재설정 안내, {name}님"
    body = f"비밀번호를 재설정하려면 링크를 클릭하세요.\n문의: support@example.com"
    print(f"[이메일 발송] to={email}, subject={subject}")
    print(body)

# 리팩터링 후 — duplicate_after.py
SUPPORT_EMAIL = "support@example.com"

def send_email(to: str, subject: str, body: str) -> None:
    footer = f"\n문의: {SUPPORT_EMAIL}"
    print(f"[이메일 발송] to={to}, subject={subject}")
    print(body + footer)

def send_welcome_email(name: str, email: str) -> None:
    send_email(
        to=email,
        subject=f"안녕하세요, {name}님!",
        body=f"가입을 환영합니다, {name}님.",
    )

def send_reset_email(name: str, email: str) -> None:
    send_email(
        to=email,
        subject=f"비밀번호 재설정 안내, {name}님",
        body="비밀번호를 재설정하려면 링크를 클릭하세요.",
    )
```

---

## 따라 하기 실습

### 실습 1 — 변수명과 함수명 개선하기

1. 아래 내용을 `messy_cart.py`로 저장합니다.

```python
def f(lst, t):
    r = 0
    for i in lst:
        r += i['p'] * i['q']
    if r >= t:
        r = r * 0.95
    return r

items = [
    {'p': 12000, 'q': 2},
    {'p': 8500, 'q': 3},
]
print(f(items, 30000))
```

2. Claude에 다음 프롬프트로 요청합니다.

```
messy_cart.py 코드를 보여줄게.
변수명과 함수명을 의미 있게 바꾸고,
타입 힌트를 추가해줘.
기능은 절대 바꾸지 마.
```

3. AI의 제안을 받아 `clean_cart.py`로 저장하고, 두 파일을 실행해서 결과가 같은지 비교합니다.

```bash
python messy_cart.py
python clean_cart.py
# 두 결과가 같아야 리팩터링 성공!
```

---

### 실습 2 — 긴 함수를 여러 함수로 분리하기

1. 아래 내용을 `monolith_report.py`로 저장합니다.

```python
def make_report(students):
    output = []
    for s in students:
        if s['score'] < 0 or s['score'] > 100:
            print(f"경고: {s['name']}의 점수가 올바르지 않습니다.")
            continue
        if s['score'] >= 90:
            grade = 'A'
        elif s['score'] >= 75:
            grade = 'B'
        elif s['score'] >= 60:
            grade = 'C'
        else:
            grade = 'F'
        passed = grade != 'F'
        line = f"{s['name']}: {s['score']}점 / {grade}등급 / {'합격' if passed else '불합격'}"
        output.append(line)
    return output

data = [
    {'name': '김민준', 'score': 88},
    {'name': '이서연', 'score': 55},
    {'name': '박지호', 'score': 101},
]
for line in make_report(data):
    print(line)
```

2. Claude에 요청합니다.

```
make_report 함수가 너무 많은 일을 해.
1) 점수 유효성 검사
2) 등급 계산
3) 합격 여부 판단
4) 출력 줄 만들기
이 네 가지를 각각 독립된 함수로 분리해줘.
```

3. 분리된 코드를 `split_report.py`로 저장하고 동일한 입력으로 테스트합니다.

---

### 실습 3 — AI 리팩터링 제안 중 하나 거절해 보기

리팩터링은 AI 제안을 무조건 따르는 게 아닙니다. 비판적으로 검토하는 연습을 합니다.

1. 실습 2에서 받은 리팩터링 결과에서 **마음에 들지 않는 부분**을 찾습니다.
   - 예: "함수가 너무 잘게 쪼개진 것 같다", "변수명이 오히려 더 길어졌다"

2. Claude에 이렇게 말해봅니다.

```
`convert_score_to_grade` 함수는 따로 분리할 필요가 없을 것 같아.
`generate_report_line` 안에 인라인으로 넣어도 충분하지 않을까?
그렇게 하면 코드가 더 간결해질 것 같은데 네 생각은?
```

3. AI의 반응을 읽고, 동의하면 적용하고 동의하지 않으면 원래 버전을 유지합니다. **여러분이 최종 결정권자**입니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|---|---|---|
| 리팩터링 후 테스트를 안 함 | 기능이 조용히 바뀜 (오류 없이 다른 결과) | 리팩터링 전후 출력값을 반드시 비교한다 |
| AI 제안을 복붙 후 이해 못 함 | `NameError: name 'xxx' is not defined` | 새 변수명·함수명이 어디서 왔는지 직접 추적한다 |
| 타입 힌트 추가 후 실행 오류 | `TypeError: 'str' object cannot be interpreted as an integer` | 타입 힌트는 강제가 아닌 힌트지만, 실제 전달하는 값의 타입을 맞춰야 한다 |
| 함수를 너무 잘게 쪼갬 | 코드가 오히려 더 복잡해 보임 | 한 함수가 5~15줄이면 충분. AI에게 "더 읽기 쉬운 수준에서 멈춰줘"라고 요청한다 |
| 상수를 함수 안에 정의 | 함수 호출마다 재정의되어 낭비 | `UPPER_SNAKE_CASE` 상수는 파일 상단 모듈 수준에 선언한다 |
| `return` 없이 함수 종료 | 결과가 `None`으로 나옴 | 함수 마지막 줄에 `return` 문이 있는지 확인한다 |

---

## 확인 체크리스트

- [ ] 리팩터링과 기능 추가의 차이를 설명할 수 있다
- [ ] AI에게 리팩터링을 요청할 때 구체적인 목표를 함께 전달했다
- [ ] 리팩터링 전후 코드를 실행해서 결과가 동일한지 확인했다
- [ ] 함수 이름만 보고도 무엇을 하는 함수인지 알 수 있다
- [ ] 코드 안에 숫자가 그냥 나오면 상수로 빼는 습관이 생겼다
- [ ] 같은 로직이 두 군데 이상 반복되는 부분을 찾아 함수로 묶었다
- [ ] AI 제안 중 한 가지 이상을 비판적으로 검토하고 의견을 말했다

---

## 한 번 더 생각해 보기

1. **리팩터링은 언제 하는 게 좋을까요?** 기능을 다 만들고 나서? 아니면 중간중간? 여러분이 경험한 상황에서 어느 순간이 가장 좋은 타이밍이었나요?

2. **AI가 제안한 리팩터링이 항상 옳을까요?** 팀원 다섯 명이 함께 쓰는 코드에서 변수명을 바꾸면 어떤 문제가 생길 수 있을까요?

3. **"충분히 좋은 코드"는 어디서 멈춰야 할까요?** 계속 리팩터링하다 보면 끝이 없습니다. 어떤 기준으로 "이 정도면 됐다"고 판단하면 좋을까요?

---

## 다음 장

다음 장에서는 AI와 함께 작성한 코드에 **자동화된 테스트(pytest)**를 붙여, 리팩터링 후에도 기능이 깨지지 않았음을 코드로 증명하는 방법을 배웁니다.