# Chapter 07. 환경변수와 설정 관리

## 이 장에서 배우는 것

- 환경변수가 무엇인지, 왜 필요한지
- 왜 API 키를 코드에 직접 쓰면 안 되는지 (보안)
- `os.environ`으로 환경변수 읽는 법
- `.env` 파일이 무엇인지
- `python-dotenv`로 `.env` 파일 사용하는 법
- PATH가 무엇인지
- 실습: API 키를 `.env` 파일에 저장하고 불러오기
- 팀 프로젝트에서 `.env`를 `.gitignore`에 추가하는 이유

---

## 왜 환경변수가 필요한가

프로그램을 만들다 보면 이런 값들이 필요해진다.

- 데이터베이스 비밀번호
- API 키 (OpenAI, Google Maps 등)
- 서버 주소
- 개발 모드인지, 배포 모드인지

이 값들을 코드 안에 직접 쓰면 두 가지 큰 문제가 생긴다.

**문제 1: 보안 위험**

```python
# 절대로 이렇게 하면 안 된다
api_key = "sk-abc123xyz456supersecretkey"
```

코드를 GitHub에 올리는 순간, API 키가 전 세계에 공개된다.  
실제로 매년 수많은 개발자가 GitHub에 API 키를 올려서 요금이 청구되는 사고를 겪는다.

**문제 2: 환경마다 다른 값이 필요하다**

개발 서버와 실제 서비스 서버의 주소가 다르고, 비밀번호도 다르다.  
코드를 매번 수정하는 것은 불편하고 실수가 생기기 쉽다.

**환경변수(Environment Variable)**는 이 두 가지 문제를 한 번에 해결한다.  
"코드 바깥에서 값을 넣어주는 방법"이다.

---

## 1. 환경변수란 무엇인가

환경변수는 운영체제(Windows, macOS, Linux)가 관리하는 키-값 쌍이다.  
프로그램이 시작될 때 운영체제가 이 값들을 프로그램에 전달한다.

비유: 환경변수는 프로그램에게 건네주는 "메모지"와 같다.  
프로그램 코드를 바꾸지 않고, 메모지의 내용만 바꿔서 동작을 조절할 수 있다.

**터미널에서 환경변수 설정하기 (macOS/Linux)**:
```bash
export APP_MODE=development
export DB_PASSWORD=mypassword
```

**Python에서 읽기**:
```python
import os

app_mode = os.environ.get("APP_MODE")
print(app_mode)  # development
```

---

## 2. `os.environ`으로 환경변수 읽기

Python의 `os` 모듈을 통해 환경변수를 읽을 수 있다.

### `os.environ["KEY"]` — 값이 없으면 에러

```python
import os

api_key = os.environ["OPENAI_API_KEY"]  # 환경변수가 없으면 KeyError 발생
```

```
KeyError: 'OPENAI_API_KEY'
```

### `os.environ.get("KEY")` — 값이 없으면 None

```python
import os

api_key = os.environ.get("OPENAI_API_KEY")
print(api_key)  # None (환경변수가 없을 때)
```

### `os.environ.get("KEY", "기본값")` — 없으면 기본값 사용

```python
import os

app_mode = os.environ.get("APP_MODE", "development")
print(app_mode)  # 환경변수가 없으면 "development" 출력
```

**권장 패턴**:
```python
import os

def get_required_env(key):
    """반드시 있어야 하는 환경변수를 읽는다. 없으면 명확한 에러를 낸다."""
    value = os.environ.get(key)
    if value is None:
        raise ValueError(f"환경변수 '{key}'가 설정되지 않았습니다. .env 파일을 확인하세요.")
    return value

api_key = get_required_env("OPENAI_API_KEY")
```

---

## 3. PATH란 무엇인가

PATH는 운영체제가 명령어를 찾는 폴더 목록을 담은 환경변수다.

터미널에서 `python`을 입력하면 운영체제는 PATH에 있는 폴더들을 순서대로 뒤져서 `python` 실행 파일을 찾는다.

```bash
# macOS/Linux에서 PATH 확인
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

# Windows에서 PATH 확인
echo %PATH%
```

**Python 개발에서 PATH가 문제가 되는 경우**:

```bash
python main.py
# zsh: command not found: python
```

이 에러는 PATH에 Python이 설치된 폴더가 포함되어 있지 않기 때문에 생긴다.

**해결**: Python이 설치된 폴더를 PATH에 추가하거나, `python3` 명령어를 사용한다.

```bash
python3 main.py  # macOS에서는 python3가 기본인 경우가 많다
```

---

## 4. `.env` 파일이란 무엇인가

매번 터미널에서 `export KEY=VALUE`를 입력하는 것은 불편하다.  
`.env` 파일은 환경변수를 파일로 관리하는 방법이다.

`.env` 파일 예시:
```
OPENAI_API_KEY=sk-abc123xyz456supersecretkey
APP_MODE=development
DB_HOST=localhost
DB_PORT=5432
DB_PASSWORD=my_local_password
```

**`.env` 파일 규칙**:
- 한 줄에 `KEY=VALUE` 형식
- `#`으로 시작하는 줄은 주석
- 따옴표는 선택 사항 (있어도 되고 없어도 된다)
- 공백 주의: `KEY = VALUE` (공백 있음)는 일부 라이브러리에서 오작동할 수 있다

```
# .env 파일 예시 (주석 가능)
# OpenAI API 키
OPENAI_API_KEY=sk-abc123xyz456supersecretkey

# 앱 설정
APP_MODE=development
APP_PORT=8000
```

---

## 5. `python-dotenv` 설치와 사용

`.env` 파일을 Python에서 읽으려면 `python-dotenv` 라이브러리가 필요하다.

### 설치

```bash
pip install python-dotenv
```

### 기본 사용법

```python
from dotenv import load_dotenv
import os

# .env 파일을 읽어서 환경변수로 설정한다
load_dotenv()

api_key = os.environ.get("OPENAI_API_KEY")
print(api_key)
```

`load_dotenv()`는 현재 디렉터리의 `.env` 파일을 찾아서 환경변수로 등록한다.

### `.env` 파일 경로 직접 지정

```python
from dotenv import load_dotenv
from pathlib import Path

# .env 파일의 위치를 직접 지정할 수 있다
env_path = Path(".") / ".env"
load_dotenv(dotenv_path=env_path)
```

---

## 6. 왜 `.env`를 `.gitignore`에 추가해야 하는가

### Git이란

Git은 코드의 변경 이력을 관리하는 도구다. GitHub은 Git 저장소를 온라인에 올려두는 서비스다.

### 문제

`.env` 파일에는 API 키, 비밀번호 같은 민감한 정보가 들어있다.  
이 파일을 실수로 GitHub에 올리면 누구나 볼 수 있게 된다.

**실제 피해 사례**:
- AWS 키가 노출되어 수백만 원의 요금이 청구된다
- OpenAI API 키가 노출되어 다른 사람이 내 API를 무단으로 사용한다
- 데이터베이스 비밀번호가 노출되어 데이터가 유출된다

### 해결 방법: `.gitignore` 파일

`.gitignore` 파일에 `.env`를 추가하면 Git이 해당 파일을 무시하고 업로드하지 않는다.

```
# .gitignore 파일 내용
.env
.env.local
.env.*.local
```

### `.env.example` 파일로 가이드 제공

`.env`는 공유하지 않지만, 어떤 환경변수가 필요한지는 알려줘야 한다.  
이를 위해 `.env.example` (또는 `.env.sample`) 파일을 만들어 GitHub에 올린다.

```
# .env.example — 실제 값은 없고 키 이름만 있다
OPENAI_API_KEY=여기에_API_키를_입력하세요
APP_MODE=development
DB_PASSWORD=여기에_데이터베이스_비밀번호를_입력하세요
```

새 팀원이 프로젝트를 클론하면:
1. `.env.example`을 복사해서 `.env`로 이름을 바꾼다
2. 실제 값을 채워 넣는다
3. 프로그램을 실행한다

---

## 7. 가상환경이란 무엇인가 (간단 설명)

`python-dotenv` 같은 외부 라이브러리를 설치하면 내 컴퓨터의 Python 환경에 설치된다.  
여러 프로젝트를 개발하다 보면 프로젝트마다 다른 버전의 라이브러리가 필요할 수 있다.

**가상환경(Virtual Environment)**은 프로젝트마다 독립된 Python 환경을 만드는 방법이다.

```bash
# 가상환경 만들기
python -m venv venv

# 가상환경 활성화 (macOS/Linux)
source venv/bin/activate

# 가상환경 활성화 (Windows)
venv\Scripts\activate

# 라이브러리 설치 (가상환경이 활성화된 상태에서)
pip install python-dotenv

# 설치된 라이브러리 목록 저장
pip freeze > requirements.txt
```

`requirements.txt`는 팀원이 같은 환경을 만들 때 사용한다:
```bash
pip install -r requirements.txt
```

---

## 실습 1 (따라 하기). `.env` 파일에 API 키 저장하고 불러오기

**목표**: `.env` 파일을 만들고, `python-dotenv`로 API 키를 안전하게 불러온다.

**1단계: `python-dotenv` 설치**

```bash
pip install python-dotenv
```

**2단계: `.env` 파일 만들기**

프로젝트 폴더 안에 `.env` 파일을 만든다.

```
# .env
OPENAI_API_KEY=my-test-api-key-12345
APP_NAME=메모앱
APP_MODE=development
```

**3단계: `.gitignore` 파일 만들기 (또는 수정하기)**

```
# .gitignore
.env
venv/
__pycache__/
```

**4단계: Python에서 읽기**

```python
# config.py
from dotenv import load_dotenv
import os

load_dotenv()  # .env 파일을 읽어서 환경변수로 등록

OPENAI_API_KEY = os.environ.get("OPENAI_API_KEY")
APP_NAME = os.environ.get("APP_NAME", "기본앱이름")
APP_MODE = os.environ.get("APP_MODE", "development")

if __name__ == "__main__":
    print(f"앱 이름: {APP_NAME}")
    print(f"모드: {APP_MODE}")
    if OPENAI_API_KEY:
        print(f"API 키: {OPENAI_API_KEY[:8]}...")  # 앞 8자리만 출력 (보안)
    else:
        print("API 키가 설정되지 않았습니다")
```

**5단계: 실행**

```bash
python config.py
```

출력:
```
앱 이름: 메모앱
모드: development
API 키: my-test...
```

**직접 해보기**: `.env` 파일에서 `APP_MODE=production`으로 바꾸고 다시 실행해보자. 코드를 전혀 수정하지 않아도 출력이 바뀌는가?

---

## 실습 2 (따라 하기). 환경변수를 사용하는 메모 앱 설정 모듈 만들기

**목표**: 환경변수로 앱의 동작을 제어하는 설정 모듈을 만든다.

**`.env` 파일**:
```
APP_MODE=development
MEMO_FILE=data/memos.json
MAX_MEMO_LENGTH=200
```

**`config.py`**:
```python
from dotenv import load_dotenv
import os

load_dotenv()

# 앱 모드 (development / production)
APP_MODE = os.environ.get("APP_MODE", "development")

# 메모 파일 경로
MEMO_FILE = os.environ.get("MEMO_FILE", "data/memos.json")

# 최대 메모 길이
try:
    MAX_MEMO_LENGTH = int(os.environ.get("MAX_MEMO_LENGTH", "500"))
except ValueError:
    print("MAX_MEMO_LENGTH가 숫자가 아닙니다. 기본값 500을 사용합니다.")
    MAX_MEMO_LENGTH = 500

def is_development():
    return APP_MODE == "development"

if __name__ == "__main__":
    print(f"모드: {APP_MODE}")
    print(f"메모 파일: {MEMO_FILE}")
    print(f"최대 길이: {MAX_MEMO_LENGTH}")
    print(f"개발 모드인가: {is_development()}")
```

**`main.py`에서 설정 사용하기**:
```python
from config import MEMO_FILE, MAX_MEMO_LENGTH, is_development

def add_memo(text):
    if len(text) > MAX_MEMO_LENGTH:
        print(f"메모가 너무 깁니다. {MAX_MEMO_LENGTH}자 이하로 입력하세요.")
        return

    if is_development():
        print(f"[개발 모드] 파일 경로: {MEMO_FILE}")

    # 실제 저장 로직 ...

if __name__ == "__main__":
    add_memo("오늘 환경변수를 배웠다")
```

**직접 해보기**: `.env` 파일에서 `MAX_MEMO_LENGTH=10`으로 바꾸고, 10자가 넘는 메모를 추가해보자. 코드 수정 없이 제한이 작동하는가?

---

## 실습 3 (따라 하기). `.env`가 없을 때 명확한 에러 메시지 만들기

**목표**: `.env` 파일이 없거나 필수 환경변수가 빠졌을 때 사용자가 이해하기 쉬운 에러를 출력한다.

```python
# config.py
from dotenv import load_dotenv
import os

load_dotenv()

def get_env(key, default=None, required=False):
    """
    환경변수를 읽는다.
    required=True이면 값이 없을 때 에러를 낸다.
    """
    value = os.environ.get(key, default)
    if required and value is None:
        raise EnvironmentError(
            f"\n[설정 오류] 환경변수 '{key}'가 없습니다.\n"
            f"프로젝트 루트 폴더에 .env 파일을 만들고 다음을 추가하세요:\n"
            f"  {key}=여기에_값을_입력하세요\n"
            f"예시 파일: .env.example 을 참고하세요."
        )
    return value

# 필수 환경변수
OPENAI_API_KEY = get_env("OPENAI_API_KEY", required=True)

# 선택 환경변수 (기본값 있음)
APP_MODE = get_env("APP_MODE", default="development")
```

```bash
python config.py
```

`.env`가 없거나 `OPENAI_API_KEY`가 없으면:
```
EnvironmentError:
[설정 오류] 환경변수 'OPENAI_API_KEY'가 없습니다.
프로젝트 루트 폴더에 .env 파일을 만들고 다음을 추가하세요:
  OPENAI_API_KEY=여기에_값을_입력하세요
예시 파일: .env.example 을 참고하세요.
```

**직접 해보기**: `.env` 파일에서 `OPENAI_API_KEY` 줄을 지우거나 주석 처리(`#`으로 시작)한 뒤 실행해보자. 에러 메시지가 나오는가?

---

## 자주 막히는 지점 (Common Pitfalls)

### Pitfall 1. `.env` 파일을 GitHub에 올린다

```bash
git add .  # 이 명령어는 .env 파일도 포함시킨다!
git commit -m "설정 파일 추가"
git push
# → .env 가 GitHub에 공개됨
```

해결: `.gitignore` 파일에 `.env`를 반드시 추가한다.

```
# .gitignore
.env
```

---

### Pitfall 2. `load_dotenv()`를 호출하지 않는다

```python
import os

# load_dotenv()를 호출하지 않았다
api_key = os.environ.get("OPENAI_API_KEY")
print(api_key)  # None — .env 파일이 있어도 읽히지 않는다
```

해결: `os.environ`을 사용하기 전에 반드시 `load_dotenv()`를 먼저 호출한다.

---

### Pitfall 3. `.env` 값을 숫자로 써도 Python은 문자열로 읽는다

```
# .env
MAX_COUNT=100
```

```python
max_count = os.environ.get("MAX_COUNT")
print(max_count + 1)  # TypeError: can only concatenate str (not "int") to str
```

해결: 필요에 따라 형 변환을 한다.
```python
max_count = int(os.environ.get("MAX_COUNT", "100"))
print(max_count + 1)  # 101
```

---

### Pitfall 4. `.env` 파일에서 따옴표가 값에 포함된다

```
# .env
API_KEY="my-secret-key"
```

```python
key = os.environ.get("API_KEY")
print(key)  # "my-secret-key" ← 따옴표가 포함될 수 있다
```

`python-dotenv`는 대부분 따옴표를 자동으로 제거하지만, 확신하려면 따옴표 없이 쓰는 것이 안전하다.
```
# .env
API_KEY=my-secret-key
```

---

### Pitfall 5. 환경변수 이름에 공백이 있다

```
# .env
API KEY=my-secret-key  # 공백이 있으면 오작동한다
```

해결: 환경변수 이름에는 공백을 쓰지 않는다. 단어 사이는 `_`를 사용한다.
```
# .env
API_KEY=my-secret-key
```

---

## 프로젝트 최종 구조 예시

```
memo_app/
├── .env              ← 실제 값이 있는 파일 (gitignore에 포함)
├── .env.example      ← 키 이름만 있는 예시 파일 (GitHub에 올림)
├── .gitignore        ← .env 포함
├── requirements.txt  ← 설치된 라이브러리 목록
├── main.py
├── config.py         ← 환경변수를 읽어 설정으로 제공
├── memo/
│   ├── __init__.py
│   ├── storage.py
│   └── formatter.py
└── data/
    └── memos.json
```

**`.env.example`**:
```
# OpenAI API 키 (https://platform.openai.com 에서 발급)
OPENAI_API_KEY=여기에_API_키를_입력하세요

# 앱 모드 (development / production)
APP_MODE=development

# 메모 파일 경로
MEMO_FILE=data/memos.json
```

**`.gitignore`**:
```
.env
venv/
__pycache__/
*.pyc
.DS_Store
```

---

## 확인 체크리스트

- 환경변수가 "코드 바깥에서 주는 설정값"임을 설명할 수 있는가
- API 키를 코드에 직접 쓰면 안 되는 이유를 말할 수 있는가
- `os.environ.get()`와 `os.environ[]`의 차이를 설명할 수 있는가
- `.env` 파일을 만들고 `python-dotenv`로 읽을 수 있는가
- `.gitignore`에 `.env`를 추가해야 하는 이유를 설명할 수 있는가
- PATH가 무엇인지 간단히 설명할 수 있는가

---

## 한 번 더 생각해 보기

1. `APP_MODE=production`으로 설정하면 개발 환경과 다른 동작을 하게 하려면 코드를 어떻게 바꾸면 될까?
2. 환경변수가 없을 때 기본값을 쓰는 것이 왜 중요한가?
3. `.env.example` 파일을 GitHub에 올리는 이유는 무엇인가?
4. 팀원 모두가 서로 다른 `.env` 파일을 가지고 있는 상황을 상상해보자. 이게 왜 좋은 방식인가?

---

## 교사용 메모

- 강조: 환경변수, dependency, path를 각각 외우기보다 "실행에 필요한 설정, 준비물, 위치"로 묶어서 설명한다.
- 막힘 포인트: 환경변수와 Python 변수의 차이, `load_dotenv()` 호출 누락, `.env` 파일을 실수로 Git에 올리는 경우에서 자주 막힌다.
- 실습 1에서 `.env`를 직접 수정하면 코드 변경 없이 출력이 바뀌는 것을 보여주면 개념이 직관적으로 전달된다.
- 보안 이슈는 실제 GitHub에 키가 노출된 사례를 언급하면 학습 동기가 높아진다.
- 질문 1: 코드를 수정하지 않고 앱을 "개발 모드"에서 "배포 모드"로 바꾸려면 무엇을 바꿔야 하는가?
- 질문 2: `.env` 파일을 실수로 GitHub에 올렸다면 어떻게 해야 하는가?

---

## 참고 자료

- Python `os` documentation: https://docs.python.org/3/library/os.html
- python-dotenv 공식 문서: https://pypi.org/project/python-dotenv/
- Python `pathlib` documentation: https://docs.python.org/3/library/pathlib.html
- GitHub Docs — .gitignore: https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files
