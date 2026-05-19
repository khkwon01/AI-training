# VS Code Chapter 02: Python 설치와 VS Code Python 설정

## 이 장에서 배우는 것

- Python을 Mac/Windows에 설치하는 방법
- VS Code에서 Python 확장 기능을 설치하고 설정하는 방법
- Python 인터프리터를 선택하는 방법과 왜 중요한지
- 가상환경(venv)을 VS Code에서 만들고 연결하는 방법
- 코드 실행 버튼과 터미널 실행의 차이

---

## 먼저 쉬운 설명

VS Code를 설치했다고 Python이 자동으로 쓸 수 있는 것은 아니다.

Python은 별도로 설치해야 하고, VS Code에게 "이 Python을 써라"고 알려줘야 한다.

처음에는 이 연결 과정이 헷갈릴 수 있다. 이 장을 따라 하면 VS Code에서 Python 코드를 바로 실행할 수 있는 상태가 된다.

---

## 1. Python 설치

### Mac

Mac에는 Python이 기본으로 들어있지만 버전이 오래됐을 수 있다. 최신 버전을 따로 설치하는 것이 좋다.

**방법 1 — 공식 사이트에서 직접 설치 (초보자 추천)**

1. https://www.python.org/downloads 접속
2. **Download Python 3.x.x** 버튼 클릭 (가장 큰 노란 버튼)
3. 다운로드된 `.pkg` 파일 실행
4. 설치 안내를 따라 Continue → Install
5. 설치 완료 후 터미널에서 확인

```bash
python3 --version
```

`Python 3.x.x` 가 출력되면 성공이다.

**방법 2 — Homebrew 사용 (Mac 패키지 관리자)**

Homebrew가 설치되어 있다면:

```bash
brew install python
```

### Windows

1. https://www.python.org/downloads 접속
2. **Download Python 3.x.x** 클릭
3. 설치 파일 실행
4. **반드시 체크**: 설치 화면 하단의 **"Add Python to PATH"** 체크박스

   이 옵션을 체크하지 않으면 터미널에서 `python` 명령이 인식되지 않는다.

5. **Install Now** 클릭
6. 설치 완료 후 VS Code를 재시작하고 확인

```bash
python --version
```

---

## 2. VS Code Python 확장 기능 설치

Python을 설치했으면 VS Code에 Python 확장 기능을 설치한다.

1. VS Code 왼쪽 Activity Bar에서 **Extensions** 아이콘 클릭 (블록 조각 모양)
2. 검색창에 `Python` 입력
3. **Python** (제작자: Microsoft, 파란 마이크로소프트 로고) 선택
4. **Install** 클릭

설치 후 `.py` 파일을 열면 VS Code 하단 상태 표시줄에 Python 버전이 표시된다.

```
Python 3.11.4 64-bit
```

### 추가로 설치하면 좋은 확장

| 확장 이름 | 제작자 | 역할 |
|-----------|--------|------|
| Pylance | Microsoft | 더 정밀한 자동완성과 타입 분석 |
| Python Debugger | Microsoft | 중단점 디버깅 |
| Ruff | Astral Software | 코드 스타일 자동 교정 |

처음에는 **Python** 하나만 있어도 충분하다.

---

## 3. Python 인터프리터 선택

**인터프리터(interpreter)**는 "VS Code가 코드를 실행할 때 사용할 Python 프로그램"이다.

컴퓨터에 Python이 여러 버전 설치되어 있을 수 있고, 가상환경마다 다른 Python을 사용할 수 있다. 인터프리터를 명확히 선택해야 원하는 Python이 실행된다.

### 인터프리터 선택 방법

**방법 1 — 상태 표시줄 클릭**

VS Code 하단 상태 표시줄에서 Python 버전 표시 부분을 클릭한다.

```
Python 3.11.4 64-bit  ←  이 부분 클릭
```

**방법 2 — Command Palette 사용**

- Mac: `Cmd + Shift + P`
- Windows: `Ctrl + Shift + P`

`Python: Select Interpreter` 입력 후 Enter

목록에서 원하는 Python 버전 선택:

```
Python 3.11.4 64-bit  /usr/local/bin/python3
Python 3.10.0 64-bit  /usr/bin/python3
```

### 인터프리터를 왜 직접 선택해야 할까

- Python이 여러 버전 있을 때 어떤 것을 쓸지 명확히 해야 한다
- 나중에 가상환경(venv)을 만들면 그 환경의 Python을 따로 선택해야 한다
- 잘못된 인터프리터가 선택되면 설치한 패키지가 보이지 않는 문제가 생긴다

---

## 4. 가상환경 만들고 VS Code에 연결하기

가상환경은 프로젝트마다 독립된 Python 환경을 만드는 방법이다. 한 프로젝트에서 설치한 패키지가 다른 프로젝트에 영향을 주지 않는다.

### 가상환경 만들기

VS Code 터미널에서 프로젝트 폴더 안에서 실행:

```bash
python3 -m venv .venv
```

`.venv` 폴더가 생성되면 성공이다.

### VS Code에서 가상환경 활성화

VS Code가 새 가상환경을 감지하면 오른쪽 하단에 팝업이 뜬다.

```
We noticed a new environment was created. Do you want to select it for this workspace?
→ Yes
```

**Yes**를 클릭하면 자동으로 연결된다.

팝업이 안 뜨면 수동으로 선택:
- `Cmd/Ctrl + Shift + P` → `Python: Select Interpreter`
- 목록에서 `.venv` 가 포함된 항목 선택

### 가상환경이 활성화됐는지 확인

터미널 프롬프트 앞에 `(.venv)` 가 표시되면 활성화된 상태다.

```bash
(.venv) $ python --version
```

---

## 5. 코드 실행 방법 비교

| 방법 | 사용법 | 장점 | 주의 |
|------|--------|------|------|
| 터미널 실행 | `python3 파일.py` | 오류 메시지 전체 확인 가능 | 터미널 위치가 맞아야 함 |
| 실행 버튼 | 오른쪽 위 ▶ 버튼 | 빠르고 편함 | 어떤 Python이 실행되는지 확인 필요 |
| 디버그 실행 | `F5` | 중단점 설정 가능 | launch.json 설정 필요할 수 있음 |

처음에는 **터미널 실행**을 권장한다. 어떤 명령이 실행되는지 직접 볼 수 있어서 오류 원인을 찾기 쉽다.

---

## 6. 따라 하기 실습

### 실습 1. Python 설치 확인

```bash
python3 --version
```

`Python 3.x.x` 가 출력되는지 확인한다.

### 실습 2. VS Code Python 확장 설치 및 인터프리터 선택

1. Extensions에서 **Python (Microsoft)** 설치
2. `python-study` 폴더를 VS Code에서 열기
3. `hello.py` 파일 열기
4. 상태 표시줄에서 Python 버전 클릭 → 인터프리터 선택

### 실습 3. 가상환경 만들기

```bash
python3 -m venv .venv
```

VS Code 팝업에서 **Yes** 클릭 또는 수동으로 `.venv` 인터프리터 선택

### 실습 4. 가상환경에서 파일 실행

터미널 프롬프트에 `(.venv)`가 표시된 상태에서:

```bash
python hello.py
```

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| Python 설치 시 "Add to PATH" 미체크 (Windows) | `python` 명령 인식 안 됨 | Python 재설치 → PATH 체크 |
| 인터프리터 선택 안 함 | 자동완성·오류 표시 없음 | Command Palette → `Python: Select Interpreter` |
| 가상환경 비활성 상태에서 pip 설치 | 전역 Python에 패키지가 설치됨 | `.venv` 인터프리터 선택 후 재설치 |
| `python3` vs `python` 혼용 | `command not found` | 설치된 Python 버전에 맞는 명령 사용 |
| 터미널 위치가 프로젝트 폴더 밖 | `No such file or directory` | `cd python-study` 로 이동 |

---

## 확인 체크리스트

- [ ] `python3 --version` 으로 Python 버전을 확인할 수 있는가
- [ ] VS Code에 Python (Microsoft) 확장이 설치되어 있는가
- [ ] 상태 표시줄에서 Python 인터프리터를 선택할 수 있는가
- [ ] `.venv` 가상환경을 만들고 VS Code에 연결할 수 있는가
- [ ] 터미널에서 `python hello.py` 로 파일을 실행할 수 있는가

---

## 한 번 더 생각해 보기

1. 인터프리터를 선택하지 않으면 어떤 문제가 생길까?
2. 가상환경이 없으면 두 프로젝트가 서로 다른 버전의 패키지를 쓸 때 어떻게 될까?
3. `python3` 와 `python` 명령이 다른 결과를 내는 이유는 무엇일까?

---

## 다음 장

다음 장에서는 Python의 첫 번째 개념인 `print()`와 변수를 배운다.

---

## 참고 자료

- Python 공식 다운로드 — https://www.python.org/downloads
- VS Code Python 설정 가이드 — https://code.visualstudio.com/docs/python/python-tutorial
- VS Code 가상환경 — https://code.visualstudio.com/docs/python/environments
