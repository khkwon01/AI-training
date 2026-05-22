# Python 개발 환경 설정 실습 01

## 학습 목표

VS Code + Python + 가상환경(venv)의 전체 환경을 처음부터 끝까지 직접 설정한다. 이 실습을 마치면 어떤 Python 프로젝트를 시작하든 동일한 방법으로 환경을 구성할 수 있다.

---

## 이 실습에서 만들 환경

```
my-python-study/          ← 프로젝트 루트 폴더
├── .venv/                ← 가상환경 (Python 패키지 창고)
├── data/                 ← 데이터 파일 저장 폴더
├── hello.py              ← 첫 번째 Python 파일
├── requirements.txt      ← 패키지 목록
└── .gitignore            ← .venv를 git에서 제외
```

---

## 단계 1: Python 설치 확인

터미널(Terminal 또는 PowerShell)을 열고 Python 버전을 확인한다.

```bash
python3 --version
```

예상 출력:
```
Python 3.11.4
```

버전이 3.8 이상이면 이 실습을 진행할 수 있다.

### Python이 설치되어 있지 않을 때

**Mac:**
```bash
# Homebrew가 있으면
brew install python3

# Homebrew 설치 방법 (아직 없는 경우)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Windows:**
1. [python.org/downloads](https://python.org/downloads) 접속
2. "Download Python 3.x.x" 버튼 클릭
3. 설치 중 **"Add Python to PATH"** 체크박스를 반드시 체크한다 (이걸 빠뜨리면 터미널에서 python 명령어가 안 됨)
4. "Install Now" 클릭

**Ubuntu / Debian:**
```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv
```

### 체크포인트 1

- [ ] `python3 --version`이 3.8 이상 버전을 출력하는가?

---

## 단계 2: VS Code 설치 확인 + Python 확장 설치

### VS Code 설치 확인

```bash
code --version
```

예상 출력:
```
1.87.0
...
```

VS Code가 없으면 [code.visualstudio.com](https://code.visualstudio.com) 에서 다운로드한다.

> Mac에서 `code` 명령어가 없다면: VS Code를 열고 `Cmd+Shift+P` → "Shell Command: Install 'code' command in PATH" 실행

### Python 확장 설치

1. VS Code를 연다
2. 왼쪽 사이드바에서 Extensions 아이콘(네모 4개 모양) 클릭
3. 검색창에 `Python` 입력
4. Microsoft가 만든 **Python** 확장 (가장 위에 나오는 것) 클릭
5. **Install** 클릭

설치 완료 후 확인 방법:
- VS Code 왼쪽 하단에 Python 버전이 표시된다 (예: `Python 3.11.4`)

### 체크포인트 2

- [ ] VS Code가 실행되는가?
- [ ] Python 확장이 설치되었는가?
- [ ] VS Code 하단에 Python 버전이 표시되는가?

---

## 단계 3: 프로젝트 폴더 만들고 VS Code에서 열기

### 터미널에서 폴더 만들기

```bash
# 홈 폴더 아래에 프로젝트 폴더 생성
mkdir ~/my-python-study
cd ~/my-python-study
```

현재 위치 확인:
```bash
pwd
```

출력 예시:
```
/Users/yourname/my-python-study
```

### VS Code에서 폴더 열기

**방법 1 — 터미널에서 열기 (권장):**
```bash
code .
```

**방법 2 — VS Code 메뉴에서 열기:**
1. VS Code 메뉴 → File → Open Folder
2. 방금 만든 `my-python-study` 폴더 선택
3. Open 클릭

### data 폴더 만들기

```bash
mkdir data
```

폴더 구조 확인:
```bash
ls -la
```

출력:
```
drwxr-xr-x  3 user staff   96 May 21 10:00 .
drwxr-xr-x 20 user staff  640 May 21 09:59 ..
drwxr-xr-x  2 user staff   64 May 21 10:00 data
```

### 체크포인트 3

- [ ] `my-python-study` 폴더가 생성되었는가?
- [ ] VS Code에서 해당 폴더가 열렸는가?
- [ ] VS Code 왼쪽 Explorer에 `data` 폴더가 보이는가?

---

## 단계 4: 터미널에서 가상환경 만들기

**VS Code 내부 터미널**을 사용하면 폴더 위치가 자동으로 맞춰져 편리하다.

VS Code 내부 터미널 열기:
- Mac: `Ctrl + `` (백틱)
- Windows: `Ctrl + `` (백틱)
- 또는 메뉴 → Terminal → New Terminal

### 가상환경 생성

터미널에서:
```bash
python3 -m venv .venv
```

성공하면 아무 메시지도 나오지 않는다. VS Code 왼쪽 Explorer에 `.venv` 폴더가 나타난다.

### 가상환경 활성화

**Mac / Linux:**
```bash
source .venv/bin/activate
```

**Windows (PowerShell):**
```powershell
.venv\Scripts\Activate.ps1
```

활성화 후 터미널 프롬프트:
```
(.venv) user@MacBook my-python-study %
```

프롬프트 앞에 `(.venv)` 가 붙으면 성공이다.

### 체크포인트 4

- [ ] `.venv` 폴더가 생성되었는가?
- [ ] 가상환경 활성화 후 프롬프트에 `(.venv)` 가 보이는가?

---

## 단계 5: VS Code에서 Python 인터프리터 선택

가상환경을 만들었으면 VS Code에게 "이 가상환경 안의 Python을 사용해"라고 알려줘야 한다.

### 인터프리터 선택 방법

1. `Cmd+Shift+P` (Mac) 또는 `Ctrl+Shift+P` (Windows) 로 Command Palette 열기
2. `Python: Select Interpreter` 입력 후 Enter
3. 목록에서 `.venv` 경로가 포함된 항목 선택

예시 목록:
```
Python 3.11.4 64-bit ('.venv': venv)   <-- 이것을 선택
Python 3.11.4 64-bit ('/usr/local/bin/python3')
```

`.venv` 라고 표시된 항목을 선택한다.

### 선택 확인

VS Code 하단 상태바를 확인한다.

```
Python 3.11.4 ('.venv': venv)
```

이렇게 표시되면 가상환경의 Python이 선택된 것이다.

### 목록에 .venv가 안 보이는 경우

VS Code가 가상환경을 자동으로 감지하지 못한 경우다.

해결 방법:
1. `Python: Select Interpreter` 에서 목록 최하단의 **"Enter interpreter path..."** 클릭
2. 직접 경로 입력:
   - Mac/Linux: `.venv/bin/python3`
   - Windows: `.venv\Scripts\python.exe`

### 체크포인트 5

- [ ] VS Code 하단에 `.venv` 가 포함된 Python 버전이 표시되는가?

---

## 단계 6: 첫 번째 Python 파일 만들고 실행

### 파일 생성

VS Code 왼쪽 Explorer에서 `+` (New File) 버튼 클릭 → `hello.py` 입력 → Enter

또는 터미널에서:
```bash
touch hello.py
```

### 코드 작성

`hello.py` 파일에 아래 코드를 입력한다.

```python
# hello.py
# 환경 설정이 제대로 됐는지 확인하는 파일

import os
from pathlib import Path

# 1. 현재 Python 위치 확인
import sys
print("Python 실행 경로:", sys.executable)

# 2. 현재 폴더 확인
print("현재 작업 폴더:", Path.cwd())

# 3. 데이터 폴더 경로 만들기
data_path = Path("data") / "student.json"
print("데이터 파일 경로:", data_path)

# 4. 환경변수 읽기
app_mode = os.environ.get("APP_MODE", "dev")
print("앱 모드:", app_mode)

# 5. 간단한 계산
numbers = [85, 90, 78, 95, 88]
average = sum(numbers) / len(numbers)
print(f"점수 평균: {average:.1f}")

print("\n환경 설정 완료!")
```

### 실행 방법 1 — VS Code Run 버튼

파일 오른쪽 상단의 ▷ (Run Python File) 버튼 클릭

### 실행 방법 2 — 터미널에서 실행 (권장)

```bash
python3 hello.py
```

### 예상 출력

```
Python 실행 경로: /Users/user/my-python-study/.venv/bin/python3
현재 작업 폴더: /Users/user/my-python-study
데이터 파일 경로: data/student.json
앱 모드: dev
점수 평균: 87.2

환경 설정 완료!
```

> **중요 확인 포인트:** "Python 실행 경로"가 `.venv/bin/python3`를 가리키고 있는가? 전역 Python 경로(`/usr/bin/python3`)가 나온다면 가상환경이 활성화되지 않은 것이다.

### requirements.txt 생성

```bash
pip freeze > requirements.txt
```

지금은 기본 패키지만 있어서 파일이 비어 있거나 매우 짧을 수 있다. 패키지를 추가 설치할 때마다 이 명령어를 다시 실행한다.

### 체크포인트 6

- [ ] `hello.py` 가 에러 없이 실행되었는가?
- [ ] Python 실행 경로가 `.venv/bin/python3`를 가리키는가?
- [ ] `requirements.txt` 파일이 생성되었는가?

---

## 단계 7: .gitignore에 .venv 추가

`.venv` 폴더는 git에 올리면 안 된다. 용량이 크고, 각자 환경에서 새로 만들면 되기 때문이다.

### .gitignore 파일 만들기

터미널에서:
```bash
touch .gitignore
```

VS Code에서 `.gitignore` 파일을 열고 아래 내용을 입력한다:

```
# 가상환경
.venv/
venv/
env/

# Python 캐시
__pycache__/
*.pyc
*.pyo
*.pyd

# 에디터 설정
.vscode/settings.json

# 환경변수 파일 (비밀 정보 포함 가능)
.env
.env.local

# macOS 시스템 파일
.DS_Store

# 데이터 파일 (필요에 따라 주석 처리)
# data/*.json
```

저장: `Cmd+S` (Mac) 또는 `Ctrl+S` (Windows)

### git 초기화 (선택 사항)

```bash
git init
git add hello.py requirements.txt .gitignore
git status
```

출력 예시:
```
On branch main

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   .gitignore
        new file:   hello.py
        new file:   requirements.txt
```

`.venv/`가 목록에 없는 것을 확인한다. `.gitignore`가 제대로 작동하고 있다는 뜻이다.

### 체크포인트 7

- [ ] `.gitignore` 파일이 생성되었는가?
- [ ] `.venv/` 가 `.gitignore` 안에 포함되어 있는가?
- [ ] `git status`에서 `.venv` 폴더가 나타나지 않는가?

---

## 완료 확인: 환경 설정 체크리스트 10개

아래 10개를 모두 확인할 수 있으면 개발 환경이 제대로 구성된 것이다.

1. [ ] `python3 --version` 이 3.8 이상을 출력한다
2. [ ] VS Code에 Microsoft Python 확장이 설치되어 있다
3. [ ] 프로젝트 폴더가 VS Code에서 열려 있다
4. [ ] `.venv` 폴더가 프로젝트 루트에 있다
5. [ ] 터미널 프롬프트 앞에 `(.venv)` 가 표시된다
6. [ ] `which python3`가 `.venv/bin/python3`를 가리킨다
7. [ ] VS Code 하단 상태바에 `('.venv': venv)` 가 표시된다
8. [ ] `hello.py`가 에러 없이 실행된다
9. [ ] `requirements.txt` 파일이 존재한다
10. [ ] `.gitignore`에 `.venv/`가 포함되어 있다

---

## 자주 겪는 문제와 해결 방법

### 문제 1: VS Code에서 인터프리터 목록에 .venv가 안 보임

원인: VS Code가 가상환경을 아직 감지하지 못했다.

해결:
1. `Cmd+Shift+P` → "Python: Select Interpreter"
2. 목록 하단 "Enter interpreter path..." 클릭
3. Mac/Linux: `.venv/bin/python3` 입력
4. Windows: `.venv\Scripts\python.exe` 입력

또는 VS Code를 완전히 닫았다가 다시 열면 자동으로 감지되기도 한다.

---

### 문제 2: 터미널에서 python 명령어가 인식 안 됨

증상:
```
zsh: command not found: python
```

원인: `python` (Python 2)은 없고 `python3`만 있는 경우

해결:
- `python` 대신 `python3`를 사용한다
- 또는 alias를 만든다: `alias python=python3` (`.zshrc` 또는 `.bashrc`에 추가)

---

### 문제 3: VS Code 터미널에서 가상환경이 자동으로 활성화되지 않음

원인: VS Code가 새 터미널을 열 때 가상환경을 자동 활성화하는 설정이 꺼져 있다.

해결:
1. `Cmd+Shift+P` → "Preferences: Open Settings (JSON)"
2. 아래 설정 추가:
```json
{
    "python.terminal.activateEnvironment": true
}
```
3. VS Code 터미널을 닫고 새로 열기

---

### 문제 4: `pip` 명령어가 없다

증상:
```
zsh: command not found: pip
```

해결:
```bash
# pip 대신 python3 -m pip 사용
python3 -m pip install requests
python3 -m pip list
python3 -m pip freeze > requirements.txt
```

---

### 문제 5: 가상환경 활성화 후에도 전역 Python을 사용하는 것 같음

확인 방법:
```bash
which python3
```

`.venv/bin/python3`가 나오지 않으면 활성화가 안 된 것이다.

해결:
```bash
# 현재 디렉토리 확인
pwd

# 가상환경이 있는 폴더인지 확인
ls -la | grep .venv

# 활성화
source .venv/bin/activate
```

---

## 이해 확인 질문

1. VS Code에서 Python 인터프리터를 `.venv`로 선택하는 이유는 무엇인가?
2. `.venv` 폴더를 git에 올리면 안 되는 이유 두 가지를 말해보자.
3. 팀원에게 이 프로젝트를 전달할 때 어떤 파일이 필요한가?
4. `sys.executable`로 확인한 Python 경로가 환경 설정이 올바른지 알려주는 이유는?
5. `os.environ.get("APP_MODE", "dev")`에서 두 번째 인자 `"dev"`의 역할은?
