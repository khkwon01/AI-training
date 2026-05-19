# Chapter 09: 가상환경과 패키지 관리 (venv & pip)

## 이 장에서 배우는 것

- venv가 왜 필요한지 (충돌 예시로 이해하기)
- venv 생성 → 활성화 → 패키지 설치 → 비활성화 전체 흐름
- `requirements.txt`를 만들고 재설치하는 방법
- VS Code에서 인터프리터를 선택하는 방법
- 실습 3개: 새 프로젝트 환경 만들기, requests로 HTTP 요청, 팀원과 환경 공유하기

---

## 왜 가상환경이 필요한가

처음 Python을 배울 때는 그냥 `pip install 패키지이름`으로 뭐든 설치하면 된다고 생각하기 쉽다. 그런데 프로젝트가 두 개 이상이 되면 문제가 생기기 시작한다.

### 충돌 예시

상황: 내 컴퓨터에 두 개의 프로젝트가 있다.

- **프로젝트 A** (2022년 만든 쇼핑몰): `requests 2.25.0` 버전이 필요
- **프로젝트 B** (2024년 만든 새 앱): `requests 2.31.0` 버전이 필요

가상환경 없이 그냥 설치하면 어떻게 될까?

```
내 컴퓨터 전체 Python
└── requests 2.31.0 (나중에 설치한 것만 남는다)
```

프로젝트 A를 실행하면 2.31.0이 설치되어 있는데, 2.25.0과 API가 달라서 오류가 난다. 반대로 2.25.0을 다시 설치하면 프로젝트 B가 망가진다.

가상환경을 쓰면 이렇게 된다.

```
내 컴퓨터
├── 프로젝트 A 가상환경
│   └── requests 2.25.0
└── 프로젝트 B 가상환경
    └── requests 2.31.0
```

각 프로젝트가 자기만의 독립된 Python 상자를 갖는다. 서로 영향을 주지 않는다.

### 가상환경이 필요한 또 다른 이유

- **팀원과 공유**: 내가 어떤 버전을 쓰는지 `requirements.txt`로 알려줄 수 있다
- **서버 배포**: 서버에도 똑같은 환경을 만들어야 프로그램이 동일하게 동작한다
- **정리**: 테스트 목적으로 설치한 것들이 다른 프로젝트에 끼어들지 않는다

---

## 1. venv란 무엇인가

`venv`는 Python에 기본으로 포함된 가상환경 도구다. 별도로 설치할 필요가 없다.

가상환경을 만들면 프로젝트 폴더 안에 `.venv`라는 폴더가 생성된다. 이 폴더 안에 Python 실행 파일과 pip, 그리고 그 가상환경에만 설치된 패키지들이 모두 들어있다.

```
my_project/
├── .venv/           <- 가상환경 폴더 (Python, pip, 패키지 모두 여기)
├── main.py
└── requirements.txt
```

---

## 2. pip란 무엇인가

`pip`는 Python 패키지를 설치, 삭제, 목록 확인하는 도구다. Python과 함께 자동으로 설치된다.

```bash
pip install requests       # 설치
pip uninstall requests     # 삭제
pip list                   # 설치된 목록 보기
pip show requests          # 특정 패키지 정보 보기
```

---

## 3. 가상환경 전체 흐름

### 3-1. 가상환경 만들기

프로젝트 폴더로 이동한 뒤 아래 명령어를 실행한다.

```bash
python3 -m venv .venv
```

- `python3 -m venv`: Python의 venv 모듈을 실행한다
- `.venv`: 가상환경 폴더 이름 (`.`으로 시작하면 숨김 폴더가 된다)

폴더 이름은 `venv`, `.venv`, `env` 등 자유롭게 정할 수 있지만, `.venv`가 가장 많이 쓰인다.

### 3-2. 가상환경 활성화하기

가상환경을 만들었다고 바로 사용되는 것이 아니다. 반드시 **활성화**해야 한다.

**macOS / Linux:**
```bash
source .venv/bin/activate
```

**Windows (명령 프롬프트):**
```
.venv\Scripts\activate.bat
```

**Windows (PowerShell):**
```
.venv\Scripts\Activate.ps1
```

활성화가 되면 터미널 프롬프트 앞에 `(.venv)`가 붙는다.

```
# 활성화 전
user@computer:~/my_project$

# 활성화 후
(.venv) user@computer:~/my_project$
```

이 `(.venv)` 표시가 보이면 지금 가상환경 안에서 작업 중이라는 뜻이다.

### 3-3. 패키지 설치하기

활성화 상태에서 `pip install`을 실행하면 가상환경 안에만 설치된다.

```bash
pip install requests
pip install flask
pip install pandas numpy
```

설치가 잘 됐는지 확인:
```bash
pip list
```

### 3-4. 가상환경 비활성화하기

작업을 마치고 나올 때는 `deactivate`를 입력한다.

```bash
deactivate
```

그러면 `(.venv)` 표시가 사라지고 원래 터미널로 돌아온다.

---

## 4. requirements.txt: 패키지 목록 파일

### 왜 필요한가

`.venv` 폴더는 용량이 크고 (수백 MB) 컴퓨터마다 다르기 때문에 Git에 올리지 않는다. 대신 **어떤 패키지가 필요한지 목록**을 `requirements.txt` 파일로 저장해서 공유한다.

### 현재 설치된 목록 저장하기

```bash
pip freeze > requirements.txt
```

이 명령어를 실행하면 현재 가상환경에 설치된 모든 패키지와 버전이 파일로 저장된다.

```
# requirements.txt 내용 예시
certifi==2024.2.2
charset-normalizer==3.3.2
idna==3.6
requests==2.31.0
urllib3==2.2.1
```

### 목록 파일로 한 번에 설치하기

새 컴퓨터나 다른 팀원이 프로젝트를 받았을 때, 가상환경을 만들고 아래 명령어 하나로 필요한 패키지를 모두 설치할 수 있다.

```bash
pip install -r requirements.txt
```

이것이 팀 프로젝트에서 venv와 requirements.txt를 같이 쓰는 이유다.

---

## 5. VS Code에서 인터프리터 선택하기

VS Code에서 Python 파일을 열면 오른쪽 아래에 현재 Python 버전이 표시된다. 가상환경을 만들었으면 VS Code가 그 가상환경의 Python을 쓰도록 설정해야 한다.

### 방법 1: 자동 감지

VS Code는 프로젝트 폴더에 `.venv` 폴더가 있으면 대부분 자동으로 감지해서 "새 가상환경을 발견했습니다. 이것을 사용하겠습니까?" 팝업을 보여준다. "예"를 누르면 된다.

### 방법 2: 수동 선택

자동 감지가 안 됐을 때:

1. `Cmd+Shift+P` (macOS) 또는 `Ctrl+Shift+P` (Windows)를 누른다
2. `Python: Select Interpreter`를 입력하고 선택한다
3. 목록에서 `.venv` 경로가 포함된 항목을 선택한다
   - 예: `Python 3.12.0 ('.venv': venv) ./venv/bin/python`

### 확인 방법

인터프리터를 선택한 뒤 VS Code 하단 상태바를 보면 Python 버전과 함께 `('.venv')` 표시가 나타난다.

```
Python 3.12.0 ('.venv': venv)
```

이 표시가 보이면 VS Code 터미널에서 `pip install`을 해도 가상환경 안에 설치된다.

---

## 실습 1. 새 프로젝트 가상환경 만들기

### 따라 하기

새 프로젝트 폴더를 만들고 가상환경을 설정하는 전체 과정을 처음부터 끝까지 해보자.

```bash
# 1. 새 프로젝트 폴더 만들기
mkdir my_first_project
cd my_first_project

# 2. 가상환경 만들기
python3 -m venv .venv

# 3. 가상환경 활성화
source .venv/bin/activate
# Windows: .venv\Scripts\activate.bat

# 4. 활성화 확인 (프롬프트에 (.venv)가 보이는지 확인)

# 5. requests 패키지 설치
pip install requests

# 6. 설치 확인
pip list

# 7. requirements.txt 저장
pip freeze > requirements.txt

# 8. 내용 확인
cat requirements.txt
```

### 직접 해보기

1. 위 과정을 직접 따라 해보자
2. `pip list` 결과와 `requirements.txt` 내용을 비교해 보자
3. `deactivate`로 가상환경을 비활성화하고, 다시 `pip list`를 실행해서 어떻게 다른지 비교해 보자

---

## 실습 2. requests로 간단한 HTTP 요청하기

### 따라 하기

`requests` 패키지를 사용해서 인터넷에서 데이터를 가져오는 프로그램을 만들어 보자.

가상환경이 활성화된 상태에서 `main.py` 파일을 만들고 아래 코드를 입력한다.

```python
import requests

def get_public_ip():
    """현재 컴퓨터의 공개 IP 주소를 조회하는 함수"""
    try:
        response = requests.get("https://api.ipify.org?format=json", timeout=5)
        response.raise_for_status()  # 오류 응답이면 예외 발생
        data = response.json()
        return data["ip"]
    except requests.exceptions.ConnectionError:
        print("인터넷 연결을 확인해 주세요.")
        return None
    except requests.exceptions.Timeout:
        print("응답 시간이 초과됐습니다.")
        return None
    except requests.exceptions.RequestException as e:
        print(f"요청 오류: {e}")
        return None

def get_joke():
    """영어 랜덤 농담을 가져오는 함수"""
    try:
        response = requests.get(
            "https://official-joke-api.appspot.com/random_joke",
            timeout=5
        )
        response.raise_for_status()
        joke = response.json()
        return f"질문: {joke['setup']}\n답: {joke['punchline']}"
    except requests.exceptions.RequestException as e:
        return f"농담을 가져오지 못했습니다: {e}"

# 실행
print("=== HTTP 요청 실습 ===\n")

ip = get_public_ip()
if ip:
    print(f"내 공개 IP: {ip}\n")

print("--- 랜덤 농담 ---")
print(get_joke())
```

실행:
```bash
python main.py
```

### 직접 해보기

1. `requests.get()` 안의 URL을 `https://jsonplaceholder.typicode.com/todos/1`로 바꿔서 실행해 보자. 어떤 데이터가 나오는지 확인한다
2. `timeout=5`를 `timeout=0.001`로 바꾸면 어떤 오류가 발생하는지 확인해 보자

---

## 실습 3. 팀원과 환경 공유하기 (시뮬레이션)

### 따라 하기

`requirements.txt`로 환경을 재현하는 과정을 실습해 보자.

```bash
# 상황: 팀원에게서 프로젝트를 받았다. requirements.txt만 있는 상태.

# 1. 새 가상환경 만들기
python3 -m venv .venv_fresh

# 2. 활성화
source .venv_fresh/bin/activate

# 3. 현재 설치된 것이 없는지 확인
pip list
# 출력: pip, setuptools 만 보인다

# 4. requirements.txt로 한 번에 설치
pip install -r requirements.txt

# 5. 설치 완료 확인
pip list

# 6. 이전과 같은 환경이 재현됐는지 확인
python main.py
```

### 직접 해보기

1. `.venv` 폴더는 Git에 올리면 안 된다. `.gitignore` 파일에 `.venv`를 추가해 보자
2. 왜 `.venv` 폴더를 Git에 올리면 안 되는지 이유를 정리해 보자 (힌트: 용량, 운영체제 차이)

---

## 초보자가 자주 막히는 지점

### 막힘 1: 가상환경 활성화를 잊고 pip install 하기

```bash
# 잘못된 흐름
python3 -m venv .venv          # 가상환경 만들기
pip install requests            # 활성화 안 하고 설치! -> 전체 Python에 설치됨

# 올바른 흐름
python3 -m venv .venv          # 가상환경 만들기
source .venv/bin/activate      # 활성화 먼저!
pip install requests            # 이제 가상환경 안에 설치됨
```

확인 방법: 터미널 프롬프트에 `(.venv)`가 붙어있는지 항상 확인한다.

### 막힘 2: Windows에서 PowerShell 실행 정책 오류

Windows PowerShell에서 `.venv\Scripts\Activate.ps1`을 실행하면 이런 오류가 날 수 있다.

```
.venv\Scripts\Activate.ps1 cannot be loaded because running scripts is disabled on this system.
```

해결 방법:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

이 명령어를 한 번 실행하면 이후로는 정상 작동한다.

### 막힘 3: python3가 아니라 python을 써야 하는 경우

macOS/Linux에서는 보통 `python3`를 써야 한다. `python`이라고 치면 Python 2가 실행되거나 명령어를 찾지 못하는 경우가 있다.

```bash
python3 --version    # Python 3.x.x 확인
python3 -m venv .venv
```

Windows에서는 `python`이 Python 3를 가리키는 경우가 많다.

### 막힘 4: `.venv` 폴더를 실수로 지웠을 때

`.venv` 폴더를 지워도 괜찮다. `requirements.txt`가 있으면 언제든 다시 만들 수 있다.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

이것이 `requirements.txt`를 항상 최신 상태로 유지해야 하는 이유다.

---

## 자주 만나는 에러와 해결법

| 에러 메시지 | 원인 | 해결 방법 |
|-------------|------|-----------|
| `command not found: python3` | Python이 설치되어 있지 않음 | Python 공식 사이트에서 설치 |
| `(.venv)`가 안 보임 | 가상환경 활성화를 안 함 | `source .venv/bin/activate` 실행 |
| `ModuleNotFoundError: No module named 'requests'` | 현재 환경에 패키지가 없음 | 가상환경 활성화 후 `pip install requests` |
| `pip: command not found` | pip가 설치되어 있지 않음 | `python3 -m pip install` 형식으로 사용 |

---

## 확인 체크리스트

- venv가 필요한 이유를 충돌 예시로 설명할 수 있는가
- `python3 -m venv .venv`로 가상환경을 만들 수 있는가
- 가상환경을 활성화하고 `(.venv)` 표시를 확인할 수 있는가
- 활성화 상태에서 패키지를 설치하고 `pip list`로 확인할 수 있는가
- `pip freeze > requirements.txt`로 목록을 저장할 수 있는가
- `pip install -r requirements.txt`로 목록에서 재설치할 수 있는가
- VS Code에서 가상환경 인터프리터를 선택할 수 있는가
- `deactivate`로 가상환경을 비활성화할 수 있는가

---

## 한 번 더 생각해 보기

1. 가상환경 없이 작업하면 어떤 문제가 생길 수 있을까?
2. `requirements.txt`가 없는 프로젝트를 받으면 어떻게 해야 할까?
3. 같은 패키지를 두 프로젝트에서 다른 버전으로 쓰는 경우가 실제로 있을까?

---

## 더 알아보기: uv (선택 사항)

`uv`는 2024년부터 많이 쓰이는 빠른 Python 패키지 관리 도구다. `pip`와 `venv`를 따로 쓰는 대신 하나로 처리한다.

초보자는 `pip`와 `venv`를 먼저 익히고, 익숙해지면 `uv`를 써보는 것이 좋다.

```bash
# uv 설치 (Mac/Linux)
curl -LsSf https://astral.sh/uv/install.sh | sh

# 새 프로젝트 시작
uv init my_project
cd my_project

# 패키지 추가 (pip install 대신)
uv add requests

# 실행 (venv 활성화 없이도 가능)
uv run python main.py
```

| 항목 | pip + venv | uv |
|------|-----------|-----|
| 속도 | 보통 | 매우 빠름 |
| 명령어 수 | 여러 개 | 적음 |
| 초보자 권장 | 먼저 배우기 | 나중에 시도 |
