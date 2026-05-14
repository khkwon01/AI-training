## 이 장에서 배우는 것

- 가상 환경(venv)이 무엇인지, 왜 필요한지 설명할 수 있다
- `try` / `except`로 예외를 처리하는 방법을 안다
- 리스트·딕셔너리 컴프리헨션으로 코드를 짧게 쓸 수 있다
- `datetime` 모듈로 날짜와 시간을 다루는 기본 방법을 안다
- 환경 변수(environment variable)를 읽고 쓰는 방법을 안다

---

## 먼저 쉬운 설명

코드를 혼자 짤 때는 아무 라이브러리나 설치해도 괜찮습니다. 하지만 팀 프로젝트를 하거나 여러 프로젝트를 동시에 관리하면 "왜 내 컴퓨터에서는 되는데 너 컴퓨터에서는 안 돼?"라는 문제가 생깁니다. 가상 환경은 **프로젝트마다 독립된 작업실**을 만들어 줍니다.

예외 처리는 **프로그램이 갑자기 멈추는 걸 막는 안전망**입니다. 날짜·시간 계산은 실무에서 정말 자주 쓰이고, 컴프리헨션은 "파이썬답게" 쓰는 첫걸음입니다. 이번 Quick Check로 Chapter 05–08의 핵심만 빠르게 정리해 봅시다.

---

## 1. 가상 환경 (venv)

가상 환경은 프로젝트별로 패키지를 격리해 주는 폴더입니다.

```bash
# 가상 환경 만들기
python -m venv .venv

# 활성화 (Mac / Linux)
source .venv/bin/activate

# 활성화 (Windows PowerShell)
.venv\Scripts\Activate.ps1

# 현재 설치된 패키지 목록을 파일로 저장
pip freeze > requirements.txt

# 다른 컴퓨터에서 그대로 설치
pip install -r requirements.txt

# 가상 환경 비활성화
deactivate
```

> **체크 포인트:** 프롬프트 앞에 `(.venv)` 가 보이면 활성화된 것입니다.

---

## 2. 환경 변수 (environment variable)

비밀번호나 API 키 같은 민감한 정보는 코드에 직접 쓰지 않고 환경 변수로 관리합니다.

```python
# env_check.py
import os

# 환경 변수 읽기 (없으면 기본값 사용)
db_host = os.getenv("DB_HOST", "localhost")
api_key = os.environ.get("API_KEY")  # 없으면 None

if api_key is None:
    print("경고: API_KEY 환경 변수가 설정되지 않았습니다.")
else:
    print(f"API 키 앞 4자리: {api_key[:4]}****")

print(f"DB 호스트: {db_host}")
```

```bash
# 터미널에서 환경 변수 설정 후 실행 (Mac/Linux)
export API_KEY="abc12345"
python env_check.py
```

---

## 3. 예외 처리 (exception handling)

```python
# safe_divide.py

def 나누기(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        print("오류: 0으로 나눌 수 없습니다.")
        return None
    except TypeError as e:
        print(f"타입 오류: {e}")
        return None
    else:
        # 예외가 없을 때만 실행
        print(f"결과: {result}")
        return result
    finally:
        # 예외 여부와 상관없이 항상 실행
        print("나누기 연산을 시도했습니다.")

나누기(10, 2)   # 결과: 5.0
나누기(10, 0)   # ZeroDivisionError 처리
나누기(10, "a") # TypeError 처리
```

**자주 나오는 예외 종류:**

| 예외 이름 | 언제 발생하나 |
|---|---|
| `ZeroDivisionError` | 0으로 나눌 때 |
| `ValueError` | 잘못된 값 변환 시 (`int("abc")`) |
| `FileNotFoundError` | 없는 파일을 열 때 |
| `KeyError` | 딕셔너리에 없는 키 접근 시 |
| `IndexError` | 리스트 범위 초과 접근 시 |

---

## 4. 컴프리헨션 (comprehension)

반복문을 한 줄로 압축하는 파이썬만의 문법입니다.

```python
# comprehension_demo.py

# 일반 for 루프
squares_loop = []
for n in range(1, 6):
    squares_loop.append(n ** 2)

# 리스트 컴프리헨션 (같은 결과)
squares = [n ** 2 for n in range(1, 6)]
print(squares)  # [1, 4, 9, 16, 25]

# 조건 필터링: 짝수만
even_squares = [n ** 2 for n in range(1, 11) if n % 2 == 0]
print(even_squares)  # [4, 16, 36, 64, 100]

# 딕셔너리 컴프리헨션
과일_가격 = {"사과": 1000, "바나나": 500, "포도": 2000}
비싼_과일 = {k: v for k, v in 과일_가격.items() if v >= 1000}
print(비싼_과일)  # {'사과': 1000, '포도': 2000}

# 집합(set) 컴프리헨션 — 중복 제거
점수_목록 = [80, 90, 80, 70, 90]
고유_점수 = {점수 for 점수 in 점수_목록}
print(고유_점수)  # {70, 80, 90}
```

---

## 5. datetime 모듈

```python
# date_practice.py
from datetime import datetime, date, timedelta

# 현재 날짜와 시간
지금 = datetime.now()
print(지금)                          # 2026-05-14 09:30:00.123456

# 포맷 지정해서 출력
print(지금.strftime("%Y년 %m월 %d일 %H시 %M분"))  # 2026년 05월 14일 09시 30분

# 문자열 → datetime 으로 변환
문자열 = "2026-01-01"
새해 = datetime.strptime(문자열, "%Y-%m-%d")
print(새해)  # 2026-01-01 00:00:00

# 날짜 계산 (timedelta)
오늘 = date.today()
일주일_뒤 = 오늘 + timedelta(days=7)
print(f"오늘: {오늘}, 일주일 뒤: {일주일_뒤}")

# 두 날짜 사이의 차이
d1 = date(2026, 1, 1)
d2 = date(2026, 5, 14)
차이 = d2 - d1
print(f"경과 일수: {차이.days}일")  # 133일
```

---

## 따라 하기 실습

### 실습 1 — 가상 환경 세팅 + 패키지 관리

```bash
# 1. 프로젝트 폴더 만들기
mkdir quick_check_02 && cd quick_check_02

# 2. 가상 환경 생성 및 활성화
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\Activate.ps1

# 3. 패키지 설치 후 requirements.txt 생성
pip install requests
pip freeze > requirements.txt
cat requirements.txt         # requests==x.x.x 가 보이면 성공
```

---

### 실습 2 — 예외 처리가 포함된 파일 읽기

`safe_reader.py` 파일을 만들고 아래 코드를 작성합니다.

```python
# safe_reader.py
import os

def 파일_읽기(경로: str) -> str:
    try:
        with open(경로, "r", encoding="utf-8") as f:
            내용 = f.read()
        return 내용
    except FileNotFoundError:
        return f"오류: '{경로}' 파일을 찾을 수 없습니다."
    except PermissionError:
        return f"오류: '{경로}' 파일을 읽을 권한이 없습니다."

# 존재하는 파일
with open("hello.txt", "w") as f:
    f.write("안녕하세요!")

print(파일_읽기("hello.txt"))       # 안녕하세요!
print(파일_읽기("없는파일.txt"))    # 오류 메시지 출력
```

```bash
python safe_reader.py
```

---

### 실습 3 — 날짜 + 컴프리헨션 결합

`birthday_checker.py` 파일을 만듭니다.

```python
# birthday_checker.py
from datetime import date

친구_생일 = {
    "민준": date(1998, 5, 14),
    "서연": date(1999, 11, 3),
    "지호": date(1997, 5, 20),
    "하은": date(2000, 1, 7),
}

오늘 = date.today()

# 이번 달에 생일인 친구만 컴프리헨션으로 추출
이번달_생일 = [
    이름
    for 이름, 생일 in 친구_생일.items()
    if 생일.month == 오늘.month
]

# 나이 계산 딕셔너리
나이_dict = {
    이름: 오늘.year - 생일.year
    for 이름, 생일 in 친구_생일.items()
}

print(f"이번 달 생일인 친구: {이번달_생일}")
print(f"친구들 나이: {나이_dict}")
```

```bash
python birthday_checker.py
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 원인 | 해결 방법 |
|---|---|---|---|
| venv 활성화 안 하고 pip 설치 | 패키지가 엉뚱한 곳에 설치됨 (에러 없음) | 전역 Python에 설치됨 | `source .venv/bin/activate` 먼저 실행 |
| `except` 순서 잘못됨 | 넓은 예외가 먼저 잡혀 구체적 처리 안 됨 | `Exception`을 맨 위에 적음 | 구체적 예외 → 넓은 예외 순으로 배치 |
| 컴프리헨션에서 `if-else` 위치 혼동 | `SyntaxError: invalid syntax` | `if-else`(값 선택)와 `if`(필터)를 헷갈림 | 값 선택: `[x if 조건 else y for x in ...]`, 필터: `[x for x in ... if 조건]` |
| `strptime` 포맷 불일치 | `ValueError: time data '...' does not match format '...'` | 날짜 문자열과 포맷 문자열이 다름 | 문자열과 포맷을 1:1 대조해서 확인 |
| `os.environ["KEY"]` 키 없음 | `KeyError: 'KEY'` | 환경 변수가 설정 안 됨 | `os.getenv("KEY", "기본값")` 사용 |
| `timedelta`를 `date`가 아닌 `int`에 더함 | `TypeError: unsupported operand type(s)` | `date` 객체가 아닌 정수에 `timedelta` 더함 | `date.today() + timedelta(days=7)` 형태로 수정 |

---

## 확인 체크리스트

- [ ] `python -m venv .venv` 명령으로 가상 환경을 만들 수 있다
- [ ] 가상 환경을 활성화·비활성화하는 명령어를 안다
- [ ] `pip freeze > requirements.txt` 로 의존성을 저장할 수 있다
- [ ] `os.getenv()`와 `os.environ.get()`의 차이를 설명할 수 있다
- [ ] `try / except / else / finally` 블록의 역할을 각각 설명할 수 있다
- [ ] 자주 쓰는 예외 3가지 이상을 말할 수 있다
- [ ] 리스트 컴프리헨션으로 짝수만 걸러내는 코드를 쓸 수 있다
- [ ] 딕셔너리 컴프리헨션으로 조건부 딕셔너리를 만들 수 있다
- [ ] `datetime.now()`로 현재 시각을 출력할 수 있다
- [ ] `strftime` / `strptime`의 차이를 안다
- [ ] `timedelta`로 날짜 덧셈·뺄셈을 할 수 있다

---

## 한 번 더 생각해 보기

1. 가상 환경 없이 모든 프로젝트에서 패키지를 공유하면 어떤 문제가 생길까요? 구체적인 상황을 하나 만들어 보세요.

2. `try / except`를 너무 넓게 쓰면(`except Exception:`) 어떤 단점이 있을까요? 반대로 너무 좁게 쓰면 어떤 문제가 생길까요?

3. 리스트 컴프리헨션이 일반 `for` 루프보다 항상 더 좋을까요? 컴프리헨션을 쓰지 말아야 할 경우는 언제일지 생각해 보세요.

---

## 다음 장

다음 장에서는 파일 입출력과 CSV·JSON 데이터 처리를 배우며, 지금까지 배운 예외 처리와 컴프리헨션을 실제 데이터 파이프라인에 적용해 봅니다.