## 이 장에서 배우는 것

- 환경 변수(Environment Variable)가 무엇인지, 왜 쓰는지 이해한다
- `.env` 파일을 만들고 `python-dotenv`로 불러오는 방법을 익힌다
- `os.environ`과 `os.getenv()`의 차이를 구분한다
- 설정값을 코드에 하드코딩하지 않고 분리하는 습관을 기른다
- `.gitignore`에 `.env`를 추가해 비밀 정보를 보호하는 방법을 안다

---

## 먼저 쉬운 설명

여러분이 배달 앱을 만든다고 상상해 보세요. 앱에서 지도 API를 쓰려면 API 키가 필요합니다. 그런데 이 키를 코드에 그냥 적어버리면 어떻게 될까요?

```python
# ❌ 이렇게 하면 절대 안 됩니다
api_key = "sk-1234abcd비밀키"
```

GitHub에 올리는 순간 전 세계 누구나 이 키를 볼 수 있습니다. 실제로 이런 실수로 하루 만에 수백만 원의 요금이 청구된 사례가 있습니다.

**환경 변수**는 이 문제를 해결합니다. 비밀 정보를 코드 바깥, 즉 운영 환경(Environment)에 저장해두고 코드는 그것을 읽어오기만 하는 방식입니다. 코드는 GitHub에 올려도 되지만, 비밀 정보는 내 컴퓨터와 서버에만 남아 있습니다.

---

## 1. 환경 변수란 무엇인가

운영체제는 프로그램이 실행될 때 참조할 수 있는 **이름=값** 쌍의 목록을 관리합니다. 이것이 환경 변수입니다.

터미널에서 현재 환경 변수를 확인해 볼 수 있습니다.

```bash
# macOS / Linux
echo $HOME
echo $PATH

# Windows (PowerShell)
echo $env:USERNAME
```

Python에서는 `os` 모듈로 환경 변수에 접근합니다.

```python
import os

# 현재 사용자 이름 읽기
username = os.environ["USER"]
print(f"현재 사용자: {username}")

# 없는 키를 읽으면 KeyError 발생
db_password = os.environ["DB_PASSWORD"]  # 아직 설정 안 했다면 에러!
```

실행하면 이런 에러가 날 수 있습니다.

```
KeyError: 'DB_PASSWORD'
```

이를 방지하려면 기본값을 함께 제공하는 `os.getenv()`를 씁니다.

```python
import os

# DB_PASSWORD가 없으면 None 반환 (에러 없음)
db_password = os.getenv("DB_PASSWORD")
print(db_password)  # None

# 기본값 지정
db_host = os.getenv("DB_HOST", "localhost")
print(db_host)  # localhost
```

> **`os.environ[키]` vs `os.getenv(키)`**
>
> | 방식 | 키가 없을 때 |
> |---|---|
> | `os.environ["KEY"]` | `KeyError` 발생 |
> | `os.getenv("KEY")` | `None` 반환 |
> | `os.getenv("KEY", "기본값")` | `"기본값"` 반환 |

---

## 2. .env 파일과 python-dotenv

매번 터미널에서 환경 변수를 직접 입력하기는 불편합니다. `.env` 파일에 모아두고 자동으로 불러오는 방식이 훨씬 편리합니다.

**설치**

```bash
pip install python-dotenv
```

**프로젝트 구조**

```
my_project/
├── .env          ← 비밀 정보 (절대 GitHub에 올리지 않음)
├── .gitignore    ← .env를 여기서 제외
├── config.py     ← 설정을 한 곳에 모음
└── main.py
```

**.env 파일 작성** (`my_project/.env`)

```dotenv
# 데이터베이스 설정
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=admin
DB_PASSWORD=supersecret123

# 외부 API
MAP_API_KEY=abcd-efgh-1234-5678
DEBUG=true
```

> `.env` 파일에는 따옴표 없이 값을 씁니다. 공백도 넣지 않습니다.

**config.py — 설정을 한 곳에서 관리하기**

```python
import os
from dotenv import load_dotenv

# .env 파일을 읽어서 환경 변수로 등록
load_dotenv()

# 데이터베이스 설정
DB_HOST = os.getenv("DB_HOST", "localhost")
DB_PORT = int(os.getenv("DB_PORT", "5432"))  # 숫자는 형변환 필요!
DB_NAME = os.getenv("DB_NAME", "default_db")
DB_USER = os.getenv("DB_USER", "")
DB_PASSWORD = os.getenv("DB_PASSWORD", "")

# API 키
MAP_API_KEY = os.getenv("MAP_API_KEY", "")

# 디버그 모드 (문자열 "true"를 bool로 변환)
DEBUG = os.getenv("DEBUG", "false").lower() == "true"
```

**main.py — config를 불러다 쓰기**

```python
import config

print(f"DB 호스트: {config.DB_HOST}")
print(f"DB 포트: {config.DB_PORT}")
print(f"디버그 모드: {config.DEBUG}")

if config.DEBUG:
    print("[DEBUG] 개발 모드로 실행 중입니다.")
```

---

## 3. .gitignore로 비밀 정보 보호하기

`.env` 파일이 실수로 GitHub에 올라가지 않도록 반드시 `.gitignore`에 추가해야 합니다.

**.gitignore 파일**

```gitignore
# 환경 변수 파일 (절대 커밋 금지)
.env
.env.local
.env.*.local

# Python 캐시
__pycache__/
*.pyc
```

**확인 방법**

```bash
git status
```

출력에서 `.env`가 보이지 않으면 정상입니다. 만약 보인다면 `.gitignore`가 제대로 적용되지 않은 것입니다.

팀원이 처음 프로젝트를 받을 때 참고할 수 있도록, 실제 값은 없지만 어떤 키가 필요한지 알려주는 `.env.example` 파일을 만들어 커밋합니다.

**.env.example**

```dotenv
DB_HOST=
DB_PORT=5432
DB_NAME=
DB_USER=
DB_PASSWORD=
MAP_API_KEY=
DEBUG=false
```

---

## 4. 운영/개발 환경 분리하기

실제 프로젝트는 개발(dev), 테스트(test), 운영(production) 환경을 분리합니다.

**파일 구조**

```
my_project/
├── .env.development   ← 개발용 (로컬 DB, 테스트 키)
├── .env.production    ← 운영용 (실제 DB, 실제 키)
├── .env.example       ← 커밋용 샘플
└── config.py
```

**config.py — 환경에 따라 다른 .env 파일 불러오기**

```python
import os
from dotenv import load_dotenv

# APP_ENV 환경 변수로 어떤 설정 파일을 쓸지 결정
# 기본값은 "development"
app_env = os.getenv("APP_ENV", "development")
env_file = f".env.{app_env}"

# 해당 파일이 있으면 불러오고, 없으면 기본 .env를 사용
if os.path.exists(env_file):
    load_dotenv(env_file)
    print(f"[설정] {env_file} 로드 완료")
else:
    load_dotenv()
    print("[설정] .env 로드 완료")

DB_HOST = os.getenv("DB_HOST", "localhost")
DB_PASSWORD = os.getenv("DB_PASSWORD", "")
DEBUG = os.getenv("DEBUG", "false").lower() == "true"
```

운영 서버에서는 이렇게 실행합니다.

```bash
APP_ENV=production python main.py
```

---

## 따라 하기 실습

### 실습 1 — 나만의 .env 파일 만들기

1. 새 폴더 `env-practice`를 만들고 그 안에 이동합니다.

   ```bash
   mkdir env-practice
   cd env-practice
   ```

2. `.env` 파일을 만들고 아래 내용을 입력합니다.

   ```dotenv
   MY_NAME=홍길동
   GREETING=안녕하세요
   MAX_RETRY=3
   ```

3. `python-dotenv`를 설치합니다.

   ```bash
   pip install python-dotenv
   ```

4. `hello.py` 파일을 만들고 실행합니다.

   ```python
   import os
   from dotenv import load_dotenv

   load_dotenv()

   name = os.getenv("MY_NAME", "이름없음")
   greeting = os.getenv("GREETING", "Hello")
   max_retry = int(os.getenv("MAX_RETRY", "1"))

   print(f"{greeting}, {name}님!")
   print(f"최대 재시도 횟수: {max_retry}번")
   ```

   ```bash
   python hello.py
   ```

   예상 출력:
   ```
   안녕하세요, 홍길동님!
   최대 재시도 횟수: 3번
   ```

---

### 실습 2 — 설정 모듈 분리하기

실습 1에서 만든 폴더에 아래 파일들을 추가합니다.

1. `config.py`를 만듭니다.

   ```python
   import os
   from dotenv import load_dotenv

   load_dotenv()

   MY_NAME = os.getenv("MY_NAME", "이름없음")
   GREETING = os.getenv("GREETING", "Hello")
   MAX_RETRY = int(os.getenv("MAX_RETRY", "1"))
   IS_VERBOSE = os.getenv("IS_VERBOSE", "false").lower() == "true"
   ```

2. `.env`에 줄을 하나 추가합니다.

   ```dotenv
   IS_VERBOSE=true
   ```

3. `app.py`를 만들고 실행합니다.

   ```python
   import config

   for i in range(config.MAX_RETRY):
       print(f"[{i+1}/{config.MAX_RETRY}] 작업 시도 중...")
       if config.IS_VERBOSE:
           print(f"  → {config.GREETING}, {config.MY_NAME}님. 상세 로그 활성화됨.")

   print("완료!")
   ```

   ```bash
   python app.py
   ```

---

### 실습 3 — .gitignore 설정하고 Git으로 확인하기

이전 실습 폴더에서 이어집니다.

1. Git 저장소를 초기화합니다.

   ```bash
   git init
   ```

2. `.gitignore` 파일을 만듭니다.

   ```gitignore
   .env
   __pycache__/
   *.pyc
   ```

3. `.env.example` 파일을 만듭니다.

   ```dotenv
   MY_NAME=
   GREETING=
   MAX_RETRY=3
   IS_VERBOSE=false
   ```

4. 상태를 확인합니다.

   ```bash
   git status
   ```

   `.env`가 목록에 **없고** `.env.example`은 있으면 성공입니다.

5. 파일을 스테이징하고 커밋합니다.

   ```bash
   git add .
   git commit -m "초기 설정: 환경 변수 구성 추가"
   ```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|---|---|---|
| `load_dotenv()` 호출을 잊음 | `os.getenv()`가 항상 `None` 반환 | `from dotenv import load_dotenv; load_dotenv()` 코드 추가 |
| `.env` 파일 경로가 틀림 | 값이 `None`으로 나옴 | `load_dotenv()` 대신 `load_dotenv("경로/.env")` 로 명시 |
| 숫자 값을 형변환 안 함 | `TypeError: '>' not supported between instances of 'str' and 'int'` | `int(os.getenv("PORT", "8080"))` 처럼 형변환 |
| bool 값을 직접 비교 | `os.getenv("DEBUG") == True` 가 항상 `False` | `.lower() == "true"` 로 문자열 비교 |
| `.env`를 Git에 커밋 | 비밀 정보 유출 | `.gitignore`에 `.env` 추가, `git rm --cached .env` 로 추적 제거 |
| `.env` 파일에 따옴표 사용 | 값에 따옴표가 포함되어 읽힘 | `DB_PASS=secret` (따옴표 없이) 또는 `DB_PASS="secret"` 둘 다 가능하지만 일관성 유지 |
| `python-dotenv` 미설치 | `ModuleNotFoundError: No module named 'dotenv'` | `pip install python-dotenv` 실행 |

---

## 확인 체크리스트

- [ ] `os.environ["KEY"]`와 `os.getenv("KEY")`의 차이를 설명할 수 있다
- [ ] `.env` 파일을 직접 만들고 내용을 채울 수 있다
- [ ] `load_dotenv()`를 코드 상단에 호출해 `.env`를 불러올 수 있다
- [ ] 숫자와 불리언 환경 변수를 올바른 타입으로 변환할 수 있다
- [ ] `config.py`에 설정을 모아두고 다른 파일에서 `import config`로 쓸 수 있다
- [ ] `.gitignore`에 `.env`를 추가하고 `git status`로 확인했다
- [ ] `.env.example` 파일을 만들어 팀원이 참고할 수 있도록 했다

---

## 한 번 더 생각해 보기

1. `.env` 파일을 실수로 GitHub에 올렸다면 어떻게 해야 할까요? 키를 삭제하면 충분할까요, 아니면 다른 조치가 필요할까요?

2. 운영 서버에는 `.env` 파일을 올리는 것 자체가 불안전할 수 있습니다. 클라우드 서비스(AWS, GCP, Heroku 등)는 환경 변수를 어떤 방식으로 관리할까요?

3. 지금까지 여러분이 만든 코드 중에 API 키, 비밀번호, 또는 개인 정보를 코드에 직접 적은 경우가 있었나요? 이 장에서 배운 방법으로 어떻게 개선할 수 있을까요?

---

## 다음 장

다음 장에서는 Python `logging` 모듈을 사용해 환경에 따라 로그 레벨을 다르게 설정하고, 디버그 정보를 파일과 콘솔에 동시에 출력하는 방법을 배웁니다.