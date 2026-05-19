## 이 장에서 배우는 것

- 정규 표현식(Regex)이 무엇인지, 왜 필요한지 이해한다
- `re` 모듈의 기본 함수(`search`, `match`, `findall`, `sub`)를 사용할 수 있다
- 자주 쓰는 패턴 문자(`\d`, `\w`, `\s`, `+`, `*`, `?`, `[]`, `^`, `$`)를 읽고 쓴다
- 이메일, 전화번호, 날짜 같은 실무 텍스트를 추출·검증한다
- 그룹(`()`)으로 원하는 부분만 꺼내는 방법을 익힌다

---

## 먼저 쉬운 설명

텍스트 데이터를 다루다 보면 이런 상황이 생깁니다.

> "이 긴 문서에서 이메일 주소만 전부 뽑아내야 해."
> "사용자가 입력한 전화번호 형식이 맞는지 확인해야 해."
> "로그 파일에서 날짜만 골라내야 해."

`if "."` 이런 단순 비교로는 한계가 있습니다. **정규 표현식(Regular Expression, Regex)**은 "이런 모양의 글자 패턴을 찾아줘"라고 컴퓨터에게 말할 수 있는 미니 언어입니다.

처음에는 기호들이 낯설어서 어렵게 느껴지지만, 자주 쓰는 패턴 10여 개만 익히면 텍스트 처리 작업이 몇 십 줄에서 한 줄로 줄어드는 경험을 하게 됩니다.

---

## 1. re 모듈 기초 — search와 match

Python에는 정규 표현식을 위한 내장 모듈 `re`가 있습니다. 따로 설치할 필요 없이 `import re`만 하면 됩니다.

```python
# regex_intro.py
import re

text = "주문번호 A1042가 오늘 배송 완료되었습니다."

# search: 문자열 어디서든 패턴을 찾는다
result = re.search(r"A\d+", text)
if result:
    print("찾은 값:", result.group())   # A1042
    print("시작 위치:", result.start()) # 4
    print("끝 위치:", result.end())     # 9
```

```python
# match vs search 차이
import re

text = "배송완료 A1042"

m = re.match(r"A\d+", text)    # 문자열 맨 앞에서만 찾음 → None
s = re.search(r"A\d+", text)   # 어디서든 찾음 → A1042

print(m)  # None
print(s.group())  # A1042
```

> **핵심 차이**: `match`는 문자열의 **맨 처음**부터 패턴이 일치해야 합니다. `search`는 **어디서든** 찾습니다.

---

## 2. 자주 쓰는 패턴 문자

| 패턴 | 의미 | 예시 |
|------|------|------|
| `\d` | 숫자 0~9 | `\d\d\d` → `042` |
| `\D` | 숫자가 아닌 것 | `\D+` → `ABC` |
| `\w` | 영문자·숫자·밑줄 | `\w+` → `hello_42` |
| `\s` | 공백·탭·줄바꿈 | `\s+` → `   ` |
| `.` | 줄바꿈 제외 모든 문자 1개 | `a.c` → `abc` |
| `+` | 1개 이상 반복 | `\d+` → `123` |
| `*` | 0개 이상 반복 | `\d*` → `` 또는 `99` |
| `?` | 0개 또는 1개 | `colou?r` → `color`, `colour` |
| `{n,m}` | n개 이상 m개 이하 | `\d{2,4}` → `12`, `1234` |
| `[]` | 문자 집합 | `[가-힣]` → 한글 한 글자 |
| `^` | 문자열 시작 | `^Hello` |
| `$` | 문자열 끝 | `end$` |

```python
# pattern_examples.py
import re

# 한글 이름 찾기 (2~4글자)
names_text = "참석자: 김철수, John, 박영희, 이민준"
korean_names = re.findall(r"[가-힣]{2,4}", names_text)
print(korean_names)  # ['김철수', '박영희', '이민준']

# 숫자만 추출
price_text = "사과 1,200원, 배 2,500원, 감 800원"
numbers = re.findall(r"\d+", price_text)
print(numbers)  # ['1', '200', '2', '500', '800']
```

---

## 3. findall — 모두 찾기

`search`는 첫 번째 결과만 반환하지만, `findall`은 **일치하는 모든 결과를 리스트**로 반환합니다.

```python
# extract_emails.py
import re

log_text = """
신청자 이메일 목록:
홍길동 hong@example.com 승인
김영수 kim.ys@company.co.kr 대기
이순신 lee@test.org 승인
잘못된형식 notanemail 제외
"""

# 이메일 주소 패턴
email_pattern = r"[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}"
emails = re.findall(email_pattern, log_text)
print(emails)
# ['hong@example.com', 'kim.ys@company.co.kr', 'lee@test.org']
```

---

## 4. 그룹 — 원하는 부분만 꺼내기

패턴에서 특정 부분만 따로 추출하고 싶을 때 `()`로 그룹을 만듭니다.

```python
# extract_phone.py
import re

contacts = """
고객센터: 02-1234-5678
영업팀: 031-9876-5432
모바일: 010-5555-1234
"""

# 지역번호와 나머지를 각각 그룹으로
phone_pattern = r"(\d{2,3})-(\d{3,4})-(\d{4})"
matches = re.findall(phone_pattern, contacts)

for match in matches:
    area, middle, last = match
    print(f"지역번호: {area}, 번호: {middle}-{last}")

# 지역번호: 02, 번호: 1234-5678
# 지역번호: 031, 번호: 9876-5432
# 지역번호: 010, 번호: 5555-1234
```

```python
# 그룹 이름 지정 (더 읽기 쉬운 코드)
import re

date_text = "계약일: 2025-07-15, 만료일: 2026-07-14"
date_pattern = r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})"

for m in re.finditer(date_pattern, date_text):
    print(f"{m.group('year')}년 {m.group('month')}월 {m.group('day')}일")

# 2025년 07월 15일
# 2026년 07월 14일
```

---

## 5. sub — 찾아서 바꾸기

`re.sub(패턴, 바꿀_문자열, 원본)`으로 특정 패턴을 다른 텍스트로 교체합니다.

```python
# sanitize_log.py
import re

# 로그에서 개인정보 마스킹
raw_log = """
2025-07-15 10:32:01 사용자 hong@example.com 로그인
2025-07-15 10:45:22 사용자 kim@test.co.kr 결제 시도
2025-07-15 11:00:00 사용자 lee@company.com 로그아웃
"""

email_pattern = r"[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}"
masked_log = re.sub(email_pattern, "***@***.***", raw_log)
print(masked_log)
# 2025-07-15 10:32:01 사용자 ***@***.*** 로그인
# ...
```

```python
# 전화번호 형식 통일 (하이픈 없애기)
import re

messy_phones = "010-1234-5678 / 010.9876.5432 / 01011112222"
clean = re.sub(r"[-.]", "", messy_phones)
print(clean)  # 01012345678 / 01098765432 / 01011112222
```

---

## 6. 컴파일 — 패턴 재사용하기

같은 패턴을 반복해서 쓴다면 `re.compile()`로 미리 컴파일해 두는 것이 효율적입니다.

```python
# validate_input.py
import re

# 패턴을 한 번만 컴파일
email_re = re.compile(r"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$")
phone_re = re.compile(r"^01[016789]-\d{3,4}-\d{4}$")

def validate_user_input(email: str, phone: str) -> dict:
    return {
        "email_ok": bool(email_re.match(email)),
        "phone_ok": bool(phone_re.match(phone)),
    }

print(validate_user_input("user@example.com", "010-1234-5678"))
# {'email_ok': True, 'phone_ok': True}

print(validate_user_input("not-an-email", "01012345678"))
# {'email_ok': False, 'phone_ok': False}
```

---

## 7. 원시 문자열(raw string) r"..."을 써야 하는 이유

정규 표현식 패턴은 항상 `r"..."` 형태로 작성하는 것이 안전합니다.

```python
# raw_string_demo.py
import re

text = "파일 경로: C:\\Users\\홍길동\\문서"

# r"" 없이 쓰면 \U, \d 등이 이스케이프 시퀀스로 해석될 수 있음
# 아래 두 줄은 같은 결과지만, r"" 버전이 훨씬 읽기 쉽다
pattern_bad  = "\\\\Users\\\\[가-힣]+"   # 백슬래시 4개 필요
pattern_good = r"\\Users\\[가-힣]+"      # 백슬래시 2개로 충분

print(re.search(pattern_good, text).group())
# \Users\홍길동
```

---

## 따라 하기 실습

### 실습 1 — 로그 파일에서 에러 라인만 추출하기

**파일명**: `parse_log.py`

아래 로그 데이터를 문자열로 붙여넣고, `ERROR`가 포함된 줄과 그 에러 코드를 추출해 봅니다.

```python
# parse_log.py
import re

log_data = """
2025-07-15 09:00:01 INFO  서버 시작 완료
2025-07-15 09:12:44 ERROR [E404] 파일을 찾을 수 없습니다
2025-07-15 09:15:00 INFO  요청 처리 완료
2025-07-15 09:31:07 ERROR [E500] 내부 서버 오류 발생
2025-07-15 09:45:22 WARN  디스크 공간 부족
2025-07-15 10:00:00 ERROR [E403] 권한이 없습니다
"""

# 1단계: ERROR 라인 전체 찾기
error_lines = re.findall(r".+ERROR.+", log_data)
print("=== 에러 라인 ===")
for line in error_lines:
    print(line.strip())

# 2단계: 에러 코드만 뽑기
error_codes = re.findall(r"\[E(\d{3})\]", log_data)
print("\n=== 에러 코드 목록 ===")
print(error_codes)  # ['404', '500', '403']
```

---

### 실습 2 — 주문 데이터 정제하기

**파일명**: `clean_orders.py`

앞 실습에서 배운 `findall`과 `sub`를 조합해 지저분한 주문 데이터를 정리합니다.

```python
# clean_orders.py
import re

raw_orders = """
주문001 | 상품명: 무선 마우스 | 가격: 25,000원 | 연락처: 010-1111-2222
주문002 | 상품명: 키보드   | 가격: 65,000원 | 연락처: 010.3333.4444
주문003 | 상품명: 모니터   | 가격: 350,000원 | 연락처: 01055556666
"""

# 1단계: 가격 숫자만 추출 (쉼표 포함)
prices = re.findall(r"[\d,]+(?=원)", raw_orders)
print("가격 목록:", prices)  # ['25,000', '65,000', '350,000']

# 2단계: 전화번호 형식 통일 (XXX-XXXX-XXXX)
def normalize_phone(text):
    # 하이픈/점 제거 후 다시 하이픈 삽입
    cleaned = re.sub(r"(01[016789])[-.‐]?(\d{3,4})[-.‐]?(\d{4})", r"\1-\2-\3", text)
    return cleaned

print(normalize_phone(raw_orders))
```

---

### 실습 3 — 회원가입 입력값 검증기 만들기

**파일명**: `signup_validator.py`

앞의 두 실습에서 배운 패턴을 활용해 실제 회원가입 폼을 검증하는 함수를 완성합니다.

```python
# signup_validator.py
import re

RULES = {
    "username": re.compile(r"^[a-z0-9_]{4,20}$"),
    "email":    re.compile(r"^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$"),
    "phone":    re.compile(r"^01[016789]-\d{3,4}-\d{4}$"),
    "password": re.compile(r"^(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%]).{8,}$"),
}

MESSAGES = {
    "username": "영소문자·숫자·밑줄만 사용, 4~20자",
    "email":    "올바른 이메일 형식을 입력하세요",
    "phone":    "010-XXXX-XXXX 형식으로 입력하세요",
    "password": "대문자·숫자·특수문자(!@#$%) 포함 8자 이상",
}

def validate(field: str, value: str) -> tuple[bool, str]:
    ok = bool(RULES[field].match(value))
    return ok, ("통과" if ok else MESSAGES[field])

# 테스트
test_cases = [
    ("username", "hong_42"),
    ("username", "홍길동"),           # 한글 → 실패
    ("email",    "user@example.com"),
    ("email",    "user@"),             # 불완전 → 실패
    ("phone",    "010-1234-5678"),
    ("password", "Secure1!"),
    ("password", "weakpass"),          # 규칙 미충족 → 실패
]

for field, value in test_cases:
    ok, msg = validate(field, value)
    status = "✓" if ok else "✗"
    print(f"{status} [{field}] '{value}' → {msg}")
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| `r""` 없이 `\d` 사용 | `DeprecationWarning: invalid escape sequence '\d'` | 패턴 앞에 `r` 붙이기: `r"\d+"` |
| `match` 결과를 바로 `.group()` 호출 | `AttributeError: 'NoneType' object has no attribute 'group'` | `if result:` 로 None 체크 후 호출 |
| `findall`에서 그룹을 쓰면 그룹 값만 반환 | 전체 매칭이 아닌 튜플 리스트가 나옴 | 전체가 필요하면 `(?:...)` 비캡처 그룹 사용 |
| `.` 이 줄바꿈을 포함하길 기대 | 여러 줄에 걸친 패턴이 매칭 안 됨 | `re.DOTALL` 플래그 추가: `re.search(p, t, re.DOTALL)` |
| 한글이 `\w`에 포함된다고 가정 | 한글이 매칭되지 않음 | 한글은 `[가-힣]` 로 명시 |
| `^` `$` 가 각 줄이 아닌 전체 문자열에만 적용 | 여러 줄 텍스트에서 `^`가 맨 첫 줄만 봄 | `re.MULTILINE` 플래그 추가 |
| 탐욕적 매칭으로 너무 많이 잡힘 | `<b>진하게</b>와 <b>이것도</b>` 에서 첫 태그만 원했는데 전체가 잡힘 | `+?` `*?` 로 비탐욕 매칭 사용 |

---

## 확인 체크리스트

- [ ] `import re` 없이 정규 표현식을 쓰려다 `NameError`가 난 적이 있고, 해결 방법을 안다
- [ ] `re.search()`와 `re.match()`의 차이를 말로 설명할 수 있다
- [ ] `\d`, `\w`, `\s`, `.`, `+`, `*`, `?` 각각이 무엇을 의미하는지 안다
- [ ] `re.findall()`의 반환값이 리스트임을 알고, 결과를 순회하는 코드를 쓸 수 있다
- [ ] `()`로 그룹을 만들고 `.group(1)` 또는 `(?P<name>...)` 으로 꺼내는 코드를 직접 썼다
- [ ] `re.sub()`으로 이메일이나 전화번호를 마스킹 처리해 봤다
- [ ] 패턴이 None을 반환할 수 있다는 것을 알고 항상 None 체크를 한다
- [ ] 왜 `r"..."` 을 써야 하는지 동료에게 설명할 수 있다

---

## 한 번 더 생각해 보기

1. `re.findall(r"(\d+)", "사과3개 배5개")`는 `['3', '5']`를 반환하고, `re.findall(r"\d+", "사과3개 배5개")`도 `['3', '5']`를 반환합니다. 두 코드의 차이는 무엇이고, 어떤 상황에서 그룹 버전이 다른 결과를 낼까요?

2. 패스워드 검증 패턴 `r"^(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%]).{8,}$"` 에서 `(?=...)` 는 **전방 탐색(lookahead)**이라는 개념입니다. 이 패턴이 "대문자가 반드시 맨 앞에 있어야 한다"가 아니라 "어딘가에만 있으면 된다"고 동작하는 이유는 무엇일까요?

3. 정규 표현식이 강력하지만 HTML/XML 파싱에는 쓰지 말라는 말을 자주 듣습니다. 그 이유가 무엇인지 `re`와 `BeautifulSoup` 같은 파서의 차이 관점에서 생각해 보세요.

---

## 다음 장

다음 장에서는 정규 표현식으로 추출한 데이터를 **pandas DataFrame**으로 가져와 분석·집계하는 방법을 배웁니다.