## 이 장에서 배우는 것

- GitHub Actions가 무엇인지, 왜 쓰는지 이해한다
- `.github/workflows/` 폴더와 YAML 파일 구조를 파악한다
- Python 프로젝트에 자동 테스트 파이프라인을 직접 만든다
- `push`할 때마다 테스트가 자동으로 돌아가게 설정한다
- CI 파이프라인이 실패했을 때 원인을 찾아 고친다

---

## 먼저 쉬운 설명

코드를 GitHub에 올릴 때마다 이런 생각 해본 적 있나요?

> "내가 방금 올린 코드가 다른 사람 코드를 망가뜨리진 않았을까?"

매번 직접 확인하려면 시간이 너무 오래 걸립니다. 그래서 **GitHub Actions**가 있습니다.

`git push`를 하는 순간, GitHub가 알아서 가상 컴퓨터를 켜고, 여러분의 코드를 받아서, 테스트를 돌리고, 결과를 알려줍니다. 통과하면 초록 체크, 실패하면 빨간 X가 표시됩니다.

이것을 **CI (Continuous Integration, 지속적 통합)** 라고 부릅니다. 팀에서 일할 때 필수이고, 혼자 개발할 때도 실수를 빨리 잡을 수 있어서 매우 유용합니다.

---

## 1. GitHub Actions의 기본 구조

GitHub Actions는 **워크플로(workflow)** 파일로 동작합니다. 워크플로 파일은 YAML 형식으로 작성하고, 반드시 이 경로에 저장해야 합니다:

```
프로젝트 폴더/
├── .github/
│   └── workflows/
│       └── ci.yml   ← 여기에 작성합니다
├── calculator.py
└── test_calculator.py
```

YAML은 들여쓰기(indent)가 매우 중요합니다. **스페이스 2칸** 또는 **스페이스 4칸**을 통일해서 사용하세요. 탭(Tab)은 사용하면 안 됩니다.

기본 구조는 아래와 같습니다:

```yaml
# .github/workflows/ci.yml

name: Python CI          # 워크플로 이름 (GitHub에서 보이는 이름)

on:                      # 언제 실행할지
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:                    # 어떤 작업을 할지
  test:                  # 작업 이름 (자유롭게 지어도 됩니다)
    runs-on: ubuntu-latest   # 어떤 OS에서 실행할지

    steps:               # 실행 순서
      - name: 코드 가져오기
        uses: actions/checkout@v4

      - name: Python 설치
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: 패키지 설치
        run: pip install pytest

      - name: 테스트 실행
        run: pytest
```

**핵심 키워드 정리:**

| 키워드 | 의미 |
|--------|------|
| `on` | 어떤 이벤트가 발생하면 실행할지 |
| `jobs` | 실행할 작업 묶음 |
| `runs-on` | 실행 환경 (가상 컴퓨터 종류) |
| `steps` | 작업 안의 순서 있는 단계 |
| `uses` | 미리 만들어진 액션을 가져다 씀 |
| `run` | 직접 터미널 명령어를 실행함 |

---

## 2. 실제 Python 프로젝트에 연결하기

간단한 계산기 모듈과 테스트 파일을 만들고, 이 코드가 CI에서 자동으로 검사되도록 해봅시다.

**`calculator.py`**

```python
# calculator.py

def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("0으로 나눌 수 없습니다.")
    return a / b
```

**`test_calculator.py`**

```python
# test_calculator.py
import pytest
from calculator import add, subtract, multiply, divide

def test_add():
    assert add(3, 4) == 7
    assert add(-1, 1) == 0

def test_subtract():
    assert subtract(10, 3) == 7

def test_multiply():
    assert multiply(3, 5) == 15

def test_divide():
    assert divide(10, 2) == 5.0

def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(5, 0)
```

이 두 파일이 있는 상태에서 `.github/workflows/ci.yml`을 만들면, `push` 할 때마다 GitHub이 자동으로 `pytest`를 실행합니다.

---

## 3. `requirements.txt`로 패키지 관리하기

실제 프로젝트에서는 패키지가 여러 개입니다. 매번 `pip install 패키지1 패키지2 ...`처럼 나열하면 번거롭습니다. `requirements.txt` 파일로 한 번에 관리합시다.

**`requirements.txt`**

```
pytest==8.2.0
pytest-cov==5.0.0
```

**`.github/workflows/ci.yml` (개선된 버전)**

```yaml
# .github/workflows/ci.yml

name: Python CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: 코드 가져오기
        uses: actions/checkout@v4

      - name: Python 3.11 설치
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: 패키지 설치
        run: pip install -r requirements.txt   # requirements.txt에서 한 번에 설치

      - name: 테스트 실행 (커버리지 포함)
        run: pytest --cov=. --cov-report=term-missing
```

`--cov=.` 옵션을 추가하면 어느 코드가 테스트되었는지 퍼센트로 보여줍니다:

```
---------- coverage: platform linux, python 3.11 ----------
Name                  Stmts   Miss  Cover
-----------------------------------------
calculator.py            10      0   100%
test_calculator.py       14      0   100%
-----------------------------------------
TOTAL                    24      0   100%
```

---

## 4. 여러 Python 버전에서 테스트하기

"내 컴퓨터에서는 되는데 팀원 컴퓨터에서는 안 돼요"라는 상황을 방지하려면, 여러 Python 버전에서 동시에 테스트하면 됩니다. 이것을 **매트릭스(matrix) 전략**이라고 합니다.

```yaml
# .github/workflows/ci.yml

name: Python CI (멀티 버전)

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']  # 세 버전 동시 테스트

    steps:
      - name: 코드 가져오기
        uses: actions/checkout@v4

      - name: Python ${{ matrix.python-version }} 설치
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}  # matrix 값 사용

      - name: 패키지 설치
        run: pip install -r requirements.txt

      - name: 테스트 실행
        run: pytest
```

`${{ matrix.python-version }}`은 YAML 안에서 변수를 쓰는 문법입니다. GitHub Actions가 자동으로 3개의 작업을 만들어 병렬로 실행합니다.

---

## 따라 하기 실습

### 실습 1 — 기본 CI 파이프라인 만들기

아래 파일들을 순서대로 만들고 GitHub에 올려보세요.

**1단계:** 프로젝트 폴더를 만들고 파일을 작성합니다.

```bash
mkdir my-calculator
cd my-calculator
git init
```

`calculator.py`와 `test_calculator.py`를 위에서 배운 내용대로 작성합니다.

**2단계:** `requirements.txt`를 만듭니다.

```bash
# requirements.txt 파일 내용
pytest==8.2.0
```

**3단계:** `.github/workflows/ci.yml` 파일을 만듭니다.

```bash
mkdir -p .github/workflows
```

에디터로 `.github/workflows/ci.yml`을 열고 아래 내용을 작성합니다:

```yaml
name: Python CI

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest
```

**4단계:** GitHub에 올립니다.

```bash
git add .
git commit -m "feat: GitHub Actions CI 파이프라인 추가"
git push origin main
```

GitHub 저장소 페이지에서 **Actions** 탭을 클릭하면 파이프라인이 실행되는 것을 볼 수 있습니다.

---

### 실습 2 — 일부러 테스트를 실패시켜 보기

CI가 실제로 오류를 잡는지 확인해 봅시다.

**1단계:** `calculator.py`의 `add` 함수를 일부러 틀리게 수정합니다.

```python
def add(a, b):
    return a - b  # 버그: + 대신 - 를 씀
```

**2단계:** 변경 사항을 push합니다.

```bash
git add calculator.py
git commit -m "bug: 버그가 있는 코드 (CI 테스트용)"
git push origin main
```

**3단계:** GitHub Actions 탭에서 빨간 X 표시와 함께 아래와 같은 실패 메시지를 확인합니다.

```
FAILED test_calculator.py::test_add - AssertionError: assert -1 == 7
```

**4단계:** 버그를 수정하고 다시 push해서 초록 체크로 돌아오는 것을 확인합니다.

---

### 실습 3 — 매트릭스로 여러 Python 버전 테스트하기

실습 1의 `ci.yml`을 아래처럼 수정하고 push합니다.

```yaml
name: Python CI (멀티 버전)

on:
  push:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ['3.10', '3.11', '3.12']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -r requirements.txt
      - run: pytest
```

Actions 탭에서 3개의 작업이 동시에 실행되는 것을 확인해 보세요.

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|------|-------------|-----------|
| YAML 들여쓰기를 탭(Tab)으로 함 | `yaml.scanner.ScannerError: mapping values are not allowed here` | 탭 대신 스페이스 2칸 또는 4칸으로 통일 |
| `requirements.txt`에 패키지가 없는데 테스트에서 import 함 | `ModuleNotFoundError: No module named 'pytest'` | `requirements.txt`에 `pytest` 추가 |
| `.github/workflows/` 폴더 경로가 틀림 (예: `.Github/workflows/`) | 워크플로가 아예 실행되지 않음 | 소문자 `.github`인지 확인 |
| `on.push.branches`에 현재 브랜치 이름이 없음 | 워크플로가 실행되지 않음 | `branches: [ main ]`을 `branches: [ 현재브랜치이름 ]`으로 수정 |
| 테스트 파일 이름이 `test_`로 시작하지 않음 | `no tests ran` | 파일 이름을 `test_파일명.py` 형식으로 변경 |
| `uses: actions/checkout@v3`처럼 오래된 버전 사용 | `Node.js 16 actions are deprecated` 경고 | `@v4`로 버전 업 |
| YAML 파일 저장 후 push를 안 함 | 변경 사항이 반영되지 않음 | `git add .github/workflows/ci.yml` 후 commit, push |

---

## 확인 체크리스트

- [ ] `.github/workflows/ci.yml` 파일이 정확한 경로에 있다
- [ ] YAML 들여쓰기가 탭이 아닌 스페이스로 되어 있다
- [ ] `on.push.branches`에 내 기본 브랜치 이름이 들어 있다
- [ ] `requirements.txt`에 `pytest`가 포함되어 있다
- [ ] `pip install -r requirements.txt` 단계가 테스트 단계보다 앞에 있다
- [ ] GitHub Actions 탭에서 워크플로가 실행된 것을 직접 확인했다
- [ ] 테스트 통과 시 초록 체크, 실패 시 빨간 X가 표시되는 것을 확인했다
- [ ] 일부러 버그를 만들어 CI가 잡는지 테스트해 봤다

---

## 한 번 더 생각해 보기

1. **"로컬에서 pytest가 통과하는데 GitHub Actions에서는 실패했다"** — 이런 상황이 왜 발생할 수 있을까요? 로컬 환경과 GitHub Actions 환경의 차이점을 생각해 보세요.

2. **`on: push` 대신 `on: pull_request`만 사용한다면** 어떤 차이가 생길까요? 어떤 경우에 pull_request 이벤트에만 CI를 걸어두는 게 더 좋을까요?

3. **현재 파이프라인은 테스트만 실행합니다.** 만약 코드 스타일 검사(lint)도 함께 실행하고 싶다면 어떤 step을 추가해야 할까요? `ruff`나 `flake8` 같은 도구를 어디에, 어떻게 넣으면 좋을지 생각해 보세요.

---

## 다음 장

다음 장에서는 CI 파이프라인에 **코드 스타일 검사(ruff lint)**와 **테스트 커버리지 리포트 자동화**를 추가해서 더 완성도 높은 파이프라인을 만들어 봅니다.