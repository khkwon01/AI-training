# VS Code Basics Chapter 01: VS Code 설치와 기본 사용

## 이 장에서 배우는 것

- VS Code가 무엇인지, 왜 쓰는지
- VS Code 설치 방법
- 폴더 열기, 파일 만들기, 저장하기
- 내장 터미널 열고 Python 파일 실행하기
- Python 확장 기능 설치하기
- Python 인터프리터 선택과 가장 기본적인 실행 흐름

---

## 먼저 쉬운 설명

코드를 쓸 때 메모장도 사용할 수 있다. 하지만 메모장에는 이런 것들이 없다.

- 코드에 색을 입혀서 읽기 쉽게 해주는 기능
- 오타나 문법 오류를 미리 알려주는 기능
- 터미널(명령줄)을 같이 보여주는 기능
- Python 실행 결과를 같은 창에서 보는 기능

VS Code는 이 모든 것을 제공하는 **코드 편집기**다. 초보자부터 전문 개발자까지 가장 널리 쓰이는 도구 중 하나다.

---

## 1. VS Code 설치

### 설치 파일 받기

1. https://code.visualstudio.com 접속
2. 운영체제에 맞는 버튼 클릭
   - Mac: **Download Mac Universal**
   - Windows: **Download for Windows**

### Mac 설치

1. 다운로드된 `.zip` 파일 더블클릭 → `Visual Studio Code.app` 파일 생성
2. `Visual Studio Code.app`을 **Applications 폴더로 드래그**
3. Applications에서 VS Code 실행
4. 처음 실행 시 "인터넷에서 다운로드된 파일입니다" 경고 → **열기** 클릭

### Windows 설치

1. 다운로드된 `.exe` 파일 실행
2. 라이선스 동의 → **I accept** 체크 → Next
3. **"Add to PATH"** 옵션이 체크되어 있는지 확인 (중요)
4. Install → Finish

---

## 2. VS Code 처음 실행 화면

VS Code를 처음 열면 **Welcome** 탭이 보인다. 닫아도 된다.

왼쪽에 세로로 아이콘들이 늘어서 있는 것이 **Activity Bar**다.

| 아이콘 | 이름 | 용도 |
|--------|------|------|
| 파일 겹친 모양 | Explorer | 파일과 폴더 목록 보기 |
| 돋보기 | Search | 파일 안에서 내용 찾기 |
| 가지 모양 | Source Control | Git 상태 확인 |
| 블록 조각 | Extensions | 확장 기능 관리 |

추가로 초보자가 먼저 알아두면 좋은 위치는 아래와 같다.

| 위치 | 쉬운 설명 |
|------|----------|
| 탭 영역 | 현재 열려 있는 파일들 |
| 편집 영역 | 코드를 직접 쓰는 큰 화면 |
| 하단 패널 | 터미널, 출력, 문제 목록이 열리는 곳 |
| 상태 표시줄 | 현재 언어, Python 버전, Git branch 등을 보여 주는 줄 |

### Command Palette

VS Code에서 기능을 가장 빨리 찾는 방법은 **Command Palette**다.

- Mac: `Cmd + Shift + P`
- Windows: `Ctrl + Shift + P`

예를 들어 아래 기능을 이름으로 바로 찾을 수 있다.

- `Python: Select Interpreter`
- `View: Toggle Terminal`
- `File: Open Folder`

---

## 3. 폴더 열기

VS Code는 **파일 하나**를 여는 것보다 **폴더 전체**를 여는 방식으로 쓰는 것이 좋다. 폴더를 열면 그 안의 모든 파일을 왼쪽 탐색기에서 바로 볼 수 있다.

### 공부용 폴더 만들기

먼저 바탕화면이나 Documents 폴더 안에 `python-study` 폴더를 만든다.

- Mac: Finder → 원하는 위치에서 오른쪽 클릭 → **새 폴더** → 이름: `python-study`
- Windows: 탐색기 → 원하는 위치에서 오른쪽 클릭 → **새 폴더** → 이름: `python-study`

### VS Code에서 폴더 열기

방법 1 — 메뉴 사용:
1. VS Code 상단 메뉴 **File > Open Folder** (Mac: **File > Open...**)
2. `python-study` 폴더 선택
3. **Open** 클릭

방법 2 — 드래그:
- `python-study` 폴더를 VS Code 창으로 드래그

폴더가 열리면 왼쪽 탐색기(Explorer)에 폴더 이름 **PYTHON-STUDY** 가 표시된다.

### 왜 파일 하나보다 폴더를 여는가

- 새 파일을 같은 위치에 만들기 쉽다
- 터미널 위치가 폴더와 맞기 쉽다
- Git과 Python 설정이 폴더 기준으로 보이기 쉽다

---

## 4. 새 파일 만들기

폴더를 열었으면 그 안에 파일을 만든다.

방법 1 — 탐색기에서 아이콘 사용:
1. 왼쪽 탐색기에서 폴더 이름 위에 마우스를 올리면 아이콘 4개가 나타난다
2. 첫 번째 아이콘 (파일 + 플러스 모양) 클릭
3. 파일 이름 입력: `hello.py` → Enter

방법 2 — 메뉴 사용:
1. **File > New File**
2. 파일 이름 입력 창에 `hello.py` 입력 → **Create File in python-study** 선택

파일 이름 끝에 `.py`를 반드시 붙여야 한다.

---

## 5. 코드 작성과 저장

파일이 열리면 오른쪽 편집 영역에 코드를 입력한다.

```python
print("Hello, VS Code")
```

저장 단축키:
- Mac: `Cmd + S`
- Windows: `Ctrl + S`

저장하지 않으면 파일 탭에 **●** (점) 표시가 남는다. 저장하면 사라진다.

### Auto Save는 나중에 켜도 된다

처음에는 저장 동작을 직접 느끼는 편이 좋다.
나중에 익숙해지면 `File > Auto Save`를 켜도 된다.

---

## 6. 터미널 열기

VS Code 안에 터미널이 내장되어 있다. 별도 창을 열지 않아도 코드를 실행할 수 있다.

터미널 여는 방법:
- 메뉴: **View > Terminal**
- 단축키 (Mac): `Ctrl + `` ` ` (백틱, 숫자 1 왼쪽 키)
- 단축키 (Windows): `Ctrl + `` ` `

터미널이 VS Code 하단에 열린다. 현재 위치가 열려 있는 폴더 안이어야 한다.

현재 위치 확인:
```bash
pwd
```

`/Users/이름/Documents/python-study` 처럼 `python-study` 안에 있으면 정상이다.

Windows에서는 아래처럼 보일 수 있다.

```text
C:\Users\이름\Documents\python-study
```

### 터미널에서 먼저 확인할 것

```bash
ls
```

현재 폴더 안에 `hello.py`가 보이면 파일 위치가 맞는 것이다.

---

## 7. Python 파일 실행하기

터미널에서 아래 명령을 입력한다.

```bash
python3 hello.py
```

출력:
```
Hello, VS Code
```

Windows에서 `python3`이 안 되면 `python`으로 시도한다.
```bash
python hello.py
```

### 실행 버튼과 터미널 실행의 차이

오른쪽 위의 실행 아이콘으로도 돌릴 수 있지만,
처음에는 터미널 명령으로 실행하는 편이 좋다.

이유:

- 어떤 명령이 실행되는지 직접 볼 수 있다
- 폴더 위치를 함께 확인할 수 있다
- 오류 메시지를 읽는 연습이 된다

### 실행이 안 될 때 확인할 것

**`python3: command not found` 오류**
→ Python이 설치되지 않았다. Python 챕터의 설치 안내를 먼저 따른다.

**`No such file or directory: hello.py` 오류**
→ 터미널의 현재 위치가 파일이 있는 폴더가 아니다.

```bash
ls       # 현재 폴더의 파일 목록 확인
cd python-study   # python-study 폴더로 이동
```

---

## 8. Python 확장 기능 설치하기

VS Code에 Python 확장 기능을 설치하면 자동완성, 오류 표시 등이 활성화된다.

1. 왼쪽 Activity Bar에서 블록 조각 모양 아이콘 (Extensions) 클릭
2. 검색창에 `Python` 입력
3. **Python** (제작자: Microsoft) 선택 → **Install** 클릭
4. 설치 완료 후 `.py` 파일을 열면 상태 표시줄 오른쪽에 Python 버전이 표시됨

Python 버전이 표시되지 않으면 상태 표시줄 하단을 클릭해서 직접 선택한다.

### 인터프리터 선택하기

확장 기능 설치 후 가장 많이 막히는 지점이 **Python 인터프리터 선택**이다.

인터프리터는 "이 프로젝트에서 어떤 Python을 사용할지" 정하는 것이다.

방법:

1. `hello.py` 파일을 연다
2. 상태 표시줄의 Python 버전 표시를 클릭하거나
3. `Cmd/Ctrl + Shift + P`를 누른 뒤 `Python: Select Interpreter` 입력
4. 목록에서 내가 설치한 Python 버전을 선택

처음에는 아래처럼 보이면 충분하다.

```text
Python 3.x.x
```

### 인터프리터를 왜 선택할까

- 실행할 Python 위치가 분명해진다
- 확장 기능이 문법 검사와 자동완성을 더 정확하게 해 준다
- 나중에 가상환경을 쓸 때도 같은 방식으로 선택할 수 있다

---

## 9. 따라 하기 실습

### 실습 1. VS Code 설치 확인

VS Code를 실행하고, 좌측 Activity Bar에서 다음을 확인한다.
- Explorer 아이콘이 있는가
- Extensions 아이콘이 있는가

### 실습 2. 폴더 열고 파일 만들기

1. `python-study` 폴더를 만든다
2. VS Code에서 **File > Open Folder** 로 연다
3. 탐색기에서 `hello.py` 파일을 만든다
4. 아래 코드를 입력하고 저장한다

```python
print("Hello, VS Code")
print("공부 시작!")
```

### 실습 3. 터미널에서 실행하기

1. **View > Terminal** 로 터미널을 연다
2. 현재 위치가 `python-study` 폴더인지 확인한다

```bash
pwd
```

3. 파일을 실행한다

```bash
python3 hello.py
```

예상 출력:
```
Hello, VS Code
공부 시작!
```

### 실습 4. Python 확장 기능 설치하기

Extensions에서 **Python (Microsoft)** 을 설치하고, `hello.py` 파일 하단에 Python 버전이 표시되는지 확인한다.

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| 파일 저장 안 함 | 실행해도 이전 내용이 그대로 | `Cmd+S` / `Ctrl+S` 로 저장 후 실행 |
| 파일 이름에 `.py` 없음 | Python 확장 기능이 동작 안 함 | 파일 이름 오른쪽 클릭 → Rename → `.py` 추가 |
| 터미널 위치가 다른 폴더 | `No such file or directory` | `cd python-study` 로 이동 |
| 폴더가 아닌 파일만 열림 | 탐색기에 파일 목록이 없음 | **File > Open Folder** 로 폴더 전체 열기 |
| Python 확장 기능 미설치 | 코드에 색이 없고 자동완성 없음 | Extensions에서 Python (Microsoft) 설치 |

---

## 확인 체크리스트

- [ ] VS Code를 설치하고 실행할 수 있는가
- [ ] `python-study` 폴더를 VS Code에서 열 수 있는가
- [ ] 탐색기에서 새 `.py` 파일을 만들 수 있는가
- [ ] 코드를 입력하고 `Cmd+S` / `Ctrl+S` 로 저장할 수 있는가
- [ ] `View > Terminal` 로 터미널을 열 수 있는가
- [ ] `python3 hello.py` 로 파일을 실행할 수 있는가
- [ ] Python 확장 기능이 설치되어 있는가

---

## 한 번 더 생각해 보기

1. 파일을 저장하지 않고 실행하면 어떤 일이 생길까?
2. VS Code 안의 터미널과 Mac/Windows 기본 터미널의 차이는 무엇인가?
3. Python 확장 기능을 설치하지 않아도 `.py` 파일을 실행할 수 있을까?

---

## 다음 장

다음 장에서는 VS Code를 더 편하게 쓰는 단축키와 유용한 기능들을 배운다. 자동완성, 오류 줄 표시, 다중 커서 등을 익히면 코딩 속도가 빨라진다.

---

## 참고 자료

- VS Code 공식 사이트 — https://code.visualstudio.com
- VS Code Python 시작 가이드 — https://code.visualstudio.com/docs/python/python-tutorial
- VS Code 기본 단축키 — https://code.visualstudio.com/docs/getstarted/keybindings
