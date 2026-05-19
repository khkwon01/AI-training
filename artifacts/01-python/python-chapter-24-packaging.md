## 이 장에서 배우는 것

- `pyproject.toml`이 무엇이고 왜 필요한지 이해한다
- 패키지의 메타데이터(이름, 버전, 의존성 등)를 `pyproject.toml`에 작성할 수 있다
- `pip install -e .`로 개발 모드 설치를 할 수 있다
- PyPI에 패키지를 배포하는 전체 흐름을 설명할 수 있다
- `build`와 `twine`으로 배포 파일을 만들고 업로드할 수 있다

---

## 먼저 쉬운 설명

Python으로 유용한 코드를 만들었다고 상상해 보세요. 친구에게 전달하려면 어떻게 해야 할까요? 파일을 직접 보내는 것도 방법이지만, `pip install 내패키지` 한 줄로 설치할 수 있다면 훨씬 멋지지 않을까요?

`pyproject.toml`은 바로 그 "포장 설명서"입니다. 택배 상자에 붙이는 송장처럼, 패키지 이름이 무엇인지, 버전은 몇인지, 어떤 라이브러리가 필요한지를 한 파일에 정리합니다.

옛날에는 `setup.py`라는 파일을 썼는데, 지금은 `pyproject.toml`이 공식 표준(PEP 517/518)입니다. 현대 Python 프로젝트를 만든다면 반드시 알아야 합니다.

---

## 1. 프로젝트 폴더 구조 이해하기

패키지를 배포하려면 파일들을 특정 구조로 배치해야 합니다.

```
my_calculator/          ← 프로젝트 루트 폴더
├── pyproject.toml      ← 패키지 설명서 (핵심!)
├── README.md           ← 사용 설명서
├── src/                ← 소스 코드 폴더 (권장 방식)
│   └── my_calculator/  ← 실제 패키지 폴더
│       ├── __init__.py
│       └── calc.py
└── tests/
    └── test_calc.py
```

`src/` 폴더를 쓰는 이유는, 설치하지 않은 상태에서 실수로 로컬 코드를 import하는 문제를 막기 위해서입니다.

**`src/my_calculator/__init__.py`**
```python
# 패키지의 대문 역할을 하는 파일
from .calc import add, subtract

__version__ = "0.1.0"
```

**`src/my_calculator/calc.py`**
```python
def add(a: float, b: float) -> float:
    """두 수를 더합니다."""
    return a + b

def subtract(a: float, b: float) -> float:
    """두 수를 뺍니다."""
    return a - b
```

---

## 2. pyproject.toml 작성하기

`pyproject.toml`은 크게 세 부분으로 나뉩니다.

**`pyproject.toml`**
```toml
# ── 1. 빌드 시스템 설정 ──────────────────────────────
[build-system]
requires = ["hatchling"]          # 빌드 도구 지정
build-backend = "hatchling.build" # 백엔드 지정

# ── 2. 패키지 핵심 정보 ──────────────────────────────
[project]
name = "my-calculator"            # pip install 할 때 쓰는 이름
version = "0.1.0"
description = "간단한 계산기 패키지"
readme = "README.md"
requires-python = ">=3.10"        # 지원하는 Python 최소 버전
license = { text = "MIT" }

# 패키지 작성자 정보
authors = [
    { name = "홍길동", email = "hong@example.com" }
]

# PyPI 검색 태그
keywords = ["calculator", "math"]
classifiers = [
    "Programming Language :: Python :: 3",
    "License :: OSI Approved :: MIT License",
]

# 이 패키지를 쓰려면 필요한 라이브러리
dependencies = [
    "requests>=2.28.0",
]

# ── 3. 선택 의존성 (개발용, 테스트용 등) ────────────
[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "ruff>=0.4.0",
]

# ── 4. 패키지 위치 알려주기 (src 레이아웃 사용 시) ──
[tool.hatch.build.targets.wheel]
packages = ["src/my_calculator"]
```

> **핵심 포인트**
> - `name`은 하이픈(`-`) 사용, import할 때는 언더스코어(`_`) 사용
> - `version`은 [시맨틱 버전](https://semver.org/lang/ko/) 형식(MAJOR.MINOR.PATCH) 권장

---

## 3. 개발 모드로 설치하기

코드를 수정하면서 바로 테스트하고 싶을 때는 **개발 모드 설치**를 사용합니다.

```bash
# 프로젝트 루트 폴더에서 실행
pip install -e .

# 개발 의존성까지 함께 설치
pip install -e ".[dev]"
```

`-e`는 editable의 약자입니다. 설치 후 `src/my_calculator/calc.py`를 수정하면 재설치 없이 바로 반영됩니다.

```python
# 설치 확인
import my_calculator
print(my_calculator.__version__)  # 0.1.0

from my_calculator import add
print(add(3, 5))  # 8
```

설치된 패키지 확인:
```bash
pip show my-calculator
# Name: my-calculator
# Version: 0.1.0
# Location: /path/to/project/src
# Editable project location: /path/to/project
```

---

## 4. 배포 파일 만들기 (빌드)

PyPI에 올리거나 다른 사람에게 배포하려면 먼저 배포 파일을 만들어야 합니다.

```bash
# 빌드 도구 설치
pip install build

# 배포 파일 생성
python -m build
```

성공하면 `dist/` 폴더가 생깁니다:
```
dist/
├── my_calculator-0.1.0-py3-none-any.whl   ← 바이너리 배포 파일
└── my_calculator-0.1.0.tar.gz             ← 소스 배포 파일
```

- **`.whl`** (wheel): 설치가 빠른 바이너리 형식. `pip install` 시 우선 사용
- **`.tar.gz`** (sdist): 소스 코드 형식. 모든 환경에서 빌드 가능

---

## 5. TestPyPI에 먼저 배포해 보기

실제 PyPI에 잘못 올리면 되돌리기 어렵습니다. 먼저 테스트 서버에 올려봅니다.

```bash
# twine 설치
pip install twine

# 배포 파일 검증
twine check dist/*

# TestPyPI에 업로드
twine upload --repository testpypi dist/*
```

업로드 시 계정 정보를 입력합니다:
```
Uploading distributions to https://test.pypi.org/legacy/
Enter your username: 홍길동
Enter your password: ****
```

> **TestPyPI 계정 만들기**: https://test.pypi.org 에서 별도 가입 필요

TestPyPI에서 설치 테스트:
```bash
pip install --index-url https://test.pypi.org/simple/ my-calculator
```

문제없으면 진짜 PyPI에 배포합니다:
```bash
twine upload dist/*
```

이후 누구나 `pip install my-calculator`로 설치할 수 있습니다.

---

## 따라 하기 실습

### 실습 1 — 프로젝트 뼈대 만들기

아래 명령어로 폴더와 파일을 직접 만들어 봅니다.

```bash
mkdir greeting_kit
cd greeting_kit
mkdir -p src/greeting_kit tests

# 필요한 파일 생성
touch src/greeting_kit/__init__.py
touch src/greeting_kit/greet.py
touch tests/test_greet.py
touch pyproject.toml
touch README.md
```

**`src/greeting_kit/greet.py`**
```python
def hello(name: str) -> str:
    return f"안녕하세요, {name}님!"

def goodbye(name: str) -> str:
    return f"안녕히 가세요, {name}님!"
```

**`src/greeting_kit/__init__.py`**
```python
from .greet import hello, goodbye

__version__ = "0.1.0"
```

---

### 실습 2 — pyproject.toml 작성 후 개발 모드 설치

**`pyproject.toml`**
```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "greeting-kit"
version = "0.1.0"
description = "한국어 인사말 패키지"
requires-python = ">=3.10"

[project.optional-dependencies]
dev = ["pytest>=7.0"]

[tool.hatch.build.targets.wheel]
packages = ["src/greeting_kit"]
```

개발 모드로 설치하고 동작을 확인합니다:
```bash
pip install -e ".[dev]"

python -c "from greeting_kit import hello; print(hello('세상'))"
# 안녕하세요, 세상님!
```

---

### 실습 3 — 빌드하고 배포 파일 검증하기

```bash
pip install build twine

python -m build

# dist/ 폴더 확인
ls dist/
# greeting_kit-0.1.0-py3-none-any.whl
# greeting_kit-0.1.0.tar.gz

# 배포 파일에 문제가 없는지 검사
twine check dist/*
# Checking dist/greeting_kit-0.1.0-py3-none-any.whl: PASSED
# Checking dist/greeting_kit-0.1.0.tar.gz: PASSED
```

실습 1 → 2 → 3 순서로 따라가면 실제 배포 전 과정을 한 번씩 경험할 수 있습니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|------|------------|-----------|
| `src/` 폴더를 `pyproject.toml`에 등록 안 함 | `ModuleNotFoundError: No module named 'greeting_kit'` | `[tool.hatch.build.targets.wheel]`에 `packages = ["src/greeting_kit"]` 추가 |
| `__init__.py` 파일 빠뜨림 | `ModuleNotFoundError` | `src/greeting_kit/__init__.py` 파일 생성 |
| `name`에 언더스코어 사용 | `InvalidDistribution: ...` | `name = "greeting-kit"` 처럼 하이픈 사용 |
| 버전을 올리지 않고 재업로드 | `HTTPError: 400 File already exists` | `pyproject.toml`의 `version` 번호를 올린 뒤 다시 빌드 |
| `twine check` 없이 바로 업로드 | `HTTPError: 400 Invalid value for ...` | 반드시 `twine check dist/*` 먼저 실행 |
| `pip install -e .` 후 코드 변경이 반영 안 됨 | 없음 (조용히 구버전 실행) | `import` 캐시 문제. Python 재시작 또는 `importlib.reload()` 사용 |

---

## 확인 체크리스트

- [ ] `pyproject.toml`에 `[build-system]`과 `[project]` 섹션이 모두 있다
- [ ] `src/패키지명/__init__.py` 파일이 존재한다
- [ ] `pip install -e .` 실행 후 패키지를 `import`할 수 있다
- [ ] `python -m build` 실행 후 `dist/` 폴더에 `.whl`과 `.tar.gz` 파일이 생겼다
- [ ] `twine check dist/*`가 `PASSED`를 출력한다
- [ ] `dependencies`에 필요한 외부 라이브러리를 명시했다
- [ ] 개발용 의존성은 `[project.optional-dependencies]`의 `dev`에 분리했다

---

## 한 번 더 생각해 보기

1. `pip install -e .`와 `pip install .`의 차이는 무엇인가요? 개발 중에는 왜 `-e` 옵션이 편리할까요?

2. 패키지 버전을 `0.1.0`에서 `0.2.0`으로 올려야 할 때와 `1.0.0`으로 올려야 할 때는 각각 어떤 상황일까요? 시맨틱 버전(MAJOR.MINOR.PATCH) 규칙을 생각해 보세요.

3. `dependencies`와 `[project.optional-dependencies]`를 나누는 이유는 무엇일까요? 사용자 입장과 개발자 입장에서 각각 생각해 보세요.

---

## 다음 장

다음 장에서는 GitHub Actions를 활용해 코드를 push할 때마다 자동으로 테스트하고 PyPI에 배포하는 CI/CD 파이프라인을 구성하는 방법을 배웁니다.