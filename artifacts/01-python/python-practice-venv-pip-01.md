# Python venv/pip 실습 01 — 가상환경과 패키지 관리

## 학습 목표

가상환경이 왜 필요한지 이해하고, 만들기 → 활성화 → 패키지 설치 → requirements.txt 저장 → 재설치 → 비활성화의 전체 흐름을 직접 따라 해 본다.

---

## 왜 가상환경이 필요한가?

Python을 처음 배울 때는 그냥 `pip install` 하면 된다고 생각하기 쉽다. 그런데 프로젝트가 2개, 3개로 늘어나면 문제가 생긴다.

### 충돌 예시

```
프로젝트 A: requests 버전 2.20 필요
프로젝트 B: requests 버전 2.31 필요
```

전역(global)에 하나만 설치할 수 있으므로, 두 프로젝트를 동시에 운영하면 버전 충돌이 발생한다.

가상환경(venv)은 프로젝트마다 Python 패키지 설치 공간을 따로 만들어 주는 도구다. 마치 프로젝트마다 독립된 창고를 하나씩 주는 것과 같다.

```
my-project-a/
    .venv/          <-- 프로젝트 A만의 패키지 창고
        requests 2.20

my-project-b/
    .venv/          <-- 프로젝트 B만의 패키지 창고
        requests 2.31
```

두 프로젝트가 서로 영향을 주지 않는다.

---

## 사전 확인

터미널을 열고 Python이 설치되어 있는지 확인한다.

```bash
python3 --version
```

예상 출력:
```
Python 3.11.4
```

버전 숫자가 3.8 이상이면 venv 모듈이 기본으로 포함되어 있다.

> Mac에서 `python3` 대신 `python`으로 시도해서 `Python 2.x`가 뜨면, 반드시 `python3`를 사용해야 한다. Python 2와 3은 서로 다른 언어라고 생각해도 좋다.

---

## 실습 환경 준비

작업할 폴더를 만들고 그 안에서 진행한다.

```bash
mkdir my-venv-practice
cd my-venv-practice
```

---

## 단계 1: 가상환경 만들기

```bash
python3 -m venv .venv
```

명령어 구조 설명:
- `python3 -m venv` — Python의 venv 모듈을 실행한다
- `.venv` — 생성할 폴더 이름 (점으로 시작하면 숨김 폴더가 됨. 관례상 `.venv`를 많이 쓴다)

### 예상 출력

성공하면 아무 메시지도 나오지 않는다. 에러가 없으면 성공이다.

### 생성된 폴더 구조 확인

```bash
ls -la
```

출력 예시:
```
drwxr-xr-x   3 user  staff   96  5 21 10:00 .
drwxr-xr-x  20 user  staff  640  5 21 10:00 ..
drwxr-xr-x   6 user  staff  192  5 21 10:00 .venv
```

`.venv` 폴더 안을 살펴보자.

```bash
ls .venv
```

출력 예시:
```
bin     include     lib     pyvenv.cfg
```

- `bin/` — Python 실행 파일과 pip가 들어 있다
- `lib/` — 설치된 패키지가 저장된다
- `pyvenv.cfg` — 가상환경 설정 파일

### 체크포인트 1

- [ ] `.venv` 폴더가 생성되었는가?
- [ ] `ls .venv` 했을 때 `bin`, `lib` 폴더가 보이는가?

---

## 단계 2: 가상환경 활성화하기

가상환경을 만들었다고 자동으로 적용되지 않는다. 반드시 활성화해야 한다.

### Mac / Linux

```bash
source .venv/bin/activate
```

### Windows (PowerShell)

```powershell
.venv\Scripts\Activate.ps1
```

### Windows (Command Prompt)

```cmd
.venv\Scripts\activate.bat
```

### 활성화 후 프롬프트 변화

활성화에 성공하면 터미널 프롬프트 앞에 `(.venv)`가 붙는다.

활성화 전:
```
user@MacBook my-venv-practice %
```

활성화 후:
```
(.venv) user@MacBook my-venv-practice %
```

이 `(.venv)` 표시가 보이면 지금 가상환경 안에 있다는 뜻이다.

### 활성화 확인

```bash
which python3
```

출력 예시:
```
/Users/user/my-venv-practice/.venv/bin/python3
```

경로가 `.venv/bin/` 안을 가리키고 있으면 정상이다. 전역 Python(`/usr/bin/python3` 같은 경로)을 가리키고 있으면 활성화가 안 된 것이다.

### 체크포인트 2

- [ ] 프롬프트 앞에 `(.venv)` 가 보이는가?
- [ ] `which python3` 결과가 `.venv/bin/python3`를 가리키는가?

---

## 단계 3: 패키지 설치하기

가상환경이 활성화된 상태에서 `requests` 패키지를 설치한다.

```bash
pip install requests
```

### 설치 로그 읽기

```
Collecting requests
  Downloading requests-2.31.0-py3-none-any.whl (62 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 62.6/62.6 kB 1.2 MB/s eta 0:00:00
Collecting charset-normalizer<4,>=2 (from requests)
  Downloading charset_normalizer-3.3.2-cp311-cp311-macosx_11_0_arm64.whl (122 kB)
Collecting idna<4,>=2.5 (from requests)
  Downloading idna-3.6-py3-none-any.whl (61 kB)
Collecting urllib3<3,>=1.21.1 (from requests)
  Downloading urllib3-2.1.0-py3-none-any.whl (104 kB)
Collecting certifi>=2017.4.17 (from requests)
  Downloading certifi-2023.11.17-py3-none-any.whl (162 kB)
Installing collected packages: urllib3, idna, certifi, charset-normalizer, requests
Successfully installed certifi-2023.11.17 charset-normalizer-3.3.2 idna-3.6 requests-2.31.0 urllib3-2.1.0
```

로그 읽는 법:
- `Collecting requests` — requests 패키지 정보를 PyPI(패키지 저장소)에서 가져오고 있다
- `Collecting charset-normalizer ...` — requests가 의존하는 다른 패키지들도 함께 설치된다
- `Successfully installed ...` — 설치 완료. 설치된 패키지와 버전이 나열된다

### 설치된 패키지 목록 확인

```bash
pip list
```

출력 예시:
```
Package            Version
------------------ ---------
certifi            2023.11.17
charset-normalizer 3.3.2
idna               3.6
requests           2.31.0
urllib3            2.1.0
```

`requests`가 목록에 있으면 정상이다.

### 체크포인트 3

- [ ] `pip install requests` 실행 후 `Successfully installed`가 출력되었는가?
- [ ] `pip list`에서 `requests`가 보이는가?

---

## 단계 4: requirements.txt 생성하기

지금 설치된 패키지 목록을 파일로 저장한다. 이 파일이 있으면 나중에 같은 환경을 그대로 재현할 수 있다.

```bash
pip freeze > requirements.txt
```

### 생성된 파일 내용 확인

```bash
cat requirements.txt
```

출력 예시:
```
certifi==2023.11.17
charset-normalizer==3.3.2
idna==3.6
requests==2.31.0
urllib3==2.1.0
```

`pip list`와 비슷하지만 `==` 기호로 정확한 버전을 명시한다.

> `pip list`는 사람이 읽기 좋은 형식, `pip freeze`는 재설치할 때 쓰는 형식이다.

### 체크포인트 4

- [ ] `requirements.txt` 파일이 생성되었는가?
- [ ] 파일 안에 `requests==2.x.x` 라인이 있는가?

---

## 단계 5: requirements.txt로 재설치하기

다른 사람이 이 프로젝트를 받거나, 새 컴퓨터에서 시작할 때 이 과정을 따라 하면 동일한 환경을 만들 수 있다.

시뮬레이션을 위해 일단 현재 가상환경을 비활성화하고, 새 가상환경을 만들어 본다.

```bash
deactivate
python3 -m venv .venv-new
source .venv-new/bin/activate
```

새 가상환경에서 requirements.txt로 한 번에 설치:

```bash
pip install -r requirements.txt
```

출력 예시:
```
Collecting certifi==2023.11.17 (from -r requirements.txt (line 1))
  Downloading certifi-2023.11.17-py3-none-any.whl (162 kB)
...
Successfully installed certifi-2023.11.17 charset-normalizer-3.3.2 idna-3.6 requests-2.31.0 urllib3-2.1.0
```

같은 버전이 정확하게 설치된다.

확인:

```bash
pip list
```

연습이 끝났으면 `.venv-new`는 지워도 된다.

```bash
deactivate
rm -rf .venv-new
source .venv/bin/activate
```

### 체크포인트 5

- [ ] `pip install -r requirements.txt`가 에러 없이 완료되었는가?
- [ ] 새 환경에서 `pip list`에 `requests`가 있는가?

---

## 단계 6: 가상환경 비활성화하기

작업이 끝나면 가상환경을 비활성화한다.

```bash
deactivate
```

비활성화 후 프롬프트:
```
user@MacBook my-venv-practice %
```

프롬프트 앞의 `(.venv)` 표시가 사라지면 비활성화된 것이다.

### 체크포인트 6

- [ ] `deactivate` 후 프롬프트 앞의 `(.venv)`가 사라졌는가?

---

## 실습 미션: requests로 HTTP GET 요청 만들기

가상환경을 활성화한 상태에서 아래 코드를 작성하고 실행해 보자.

### 파일 만들기

프로젝트 폴더 안에 `hello_requests.py` 파일을 만든다.

```python
# hello_requests.py
import requests

# 공개 API에 GET 요청 보내기
url = "https://httpbin.org/get"
response = requests.get(url)

# 응답 상태 코드 출력
print("상태 코드:", response.status_code)

# 응답 JSON 내용 출력
data = response.json()
print("응답 데이터:", data)

# 내 IP 주소 확인 (httpbin이 반환해 줌)
print("내 IP:", data.get("origin"))
```

### 실행

```bash
python3 hello_requests.py
```

### 예상 출력

```
상태 코드: 200
응답 데이터: {'args': {}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Host': 'httpbin.org', ...}, 'origin': '123.456.789.0', 'url': 'https://httpbin.org/get'}
내 IP: 123.456.789.0
```

상태 코드 200이 나오면 HTTP 요청이 성공한 것이다.

### 미션 체크포인트

- [ ] 상태 코드 200이 출력되었는가?
- [ ] `origin` 값(IP 주소)이 출력되었는가?

---

## 자주 겪는 오류와 해결 방법

### 오류 1: `python3: command not found`

```
zsh: command not found: python3
```

원인: Python이 설치되어 있지 않거나, PATH에 등록되어 있지 않다.

해결:
- Mac: `brew install python3` (Homebrew가 없으면 먼저 설치)
- Windows: python.org에서 installer 다운로드. 설치 시 "Add Python to PATH" 체크박스 반드시 체크

---

### 오류 2: `pip: command not found`

```
zsh: command not found: pip
```

원인: 가상환경이 활성화되지 않았거나, Python 설치가 불완전하다.

해결:
```bash
# 가상환경 활성화 여부 확인 (프롬프트 앞에 (.venv) 있는지 확인)
# 없으면:
source .venv/bin/activate

# 그래도 안 되면:
python3 -m pip install requests
```

---

### 오류 3: 가상환경이 활성화 안 된 상태에서 설치

증상: `pip list`를 했을 때 이미 수십 개의 패키지가 있고, 전혀 모르는 패키지들이 섞여 있다.

원인: 전역(global) Python 환경에 설치한 것이다.

해결:
1. `deactivate`로 일단 비활성화
2. `source .venv/bin/activate`로 가상환경 활성화
3. 프롬프트에 `(.venv)` 가 붙어 있는지 확인 후 재설치

---

### 오류 4: `ModuleNotFoundError: No module named 'requests'`

```
ModuleNotFoundError: No module named 'requests'
```

원인: `pip install requests`를 전역에 설치했거나, 다른 가상환경에서 실행 중이다.

해결:
```bash
# 현재 활성화된 Python 확인
which python3
# → .venv/bin/python3 가 나와야 함

# 가상환경 내부에 requests가 있는지 확인
pip list | grep requests
```

---

### 오류 5: Windows에서 스크립트 실행 오류

```
.venv\Scripts\Activate.ps1 cannot be loaded because running scripts is disabled on this system.
```

원인: PowerShell 보안 정책이 스크립트 실행을 막고 있다.

해결 (PowerShell을 관리자 권한으로 실행):
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## 전체 완료 체크리스트

이 실습을 끝낸 뒤 아래 항목을 모두 체크할 수 있으면 성공이다.

- [ ] `python3 --version`으로 Python 버전을 확인했다
- [ ] `python3 -m venv .venv`로 가상환경 폴더를 만들었다
- [ ] `source .venv/bin/activate`로 활성화했고 프롬프트에 `(.venv)`가 나타났다
- [ ] `which python3`가 `.venv/bin/python3`를 가리키는 것을 확인했다
- [ ] `pip install requests`로 패키지를 설치했다
- [ ] `pip list`에서 `requests`를 확인했다
- [ ] `pip freeze > requirements.txt`로 설치 목록을 저장했다
- [ ] `pip install -r requirements.txt`로 재설치가 가능함을 확인했다
- [ ] `hello_requests.py`를 실행해서 상태 코드 200을 받았다
- [ ] `deactivate`로 가상환경을 비활성화했다

---

## 이해 확인 질문

1. 가상환경 없이 전역으로 설치하면 어떤 문제가 생기는가?
2. `pip freeze`와 `pip list`의 차이는 무엇인가?
3. 동료에게 이 프로젝트를 넘겨줄 때 어떤 파일이 반드시 필요한가?
4. `.venv` 폴더를 git에 올려야 하는가? 그 이유는?
5. `source .venv/bin/activate`를 하지 않고 `pip install`을 하면 어디에 설치되는가?
