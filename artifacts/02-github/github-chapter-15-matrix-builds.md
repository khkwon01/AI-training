# Chapter 15: Matrix 빌드와 병렬 테스팅

## 이 장에서 배우는 것

- GitHub Actions의 **matrix 전략**이 무엇인지 이해한다
- 여러 Python 버전, OS 환경에서 동시에 테스트를 실행하는 워크플로를 작성한다
- 병렬(parallel) 테스트가 왜 시간을 절약하는지 설명할 수 있다
- matrix 빌드에서 특정 조합을 제외(`exclude`)하거나 추가(`include`)하는 방법을 안다
- 실패한 job만 골라서 다시 실행하는 방법을 안다

---

## 먼저 쉬운 설명

코드를 작성하면 "내 컴퓨터에서는 되는데 친구 컴퓨터에서는 안 돼요"라는 상황을 겪게 됩니다. 운영체제가 다르거나, Python 버전이 다르거나, 라이브러리 버전이 달라서 생기는 문제입니다.

**matrix 빌드**는 이 문제를 미리 잡아주는 도구입니다. 예를 들어:

- Python 3.10, 3.11, 3.12
- Ubuntu, Windows, macOS

이 세 가지 버전과 세 가지 OS를 **모두 조합**해서 자동으로 테스트해 줍니다. 직접 하면 9번 반복해야 할 작업을 GitHub이 한 번에 병렬로 처리해 줍니다.

> 팀 프로젝트에서 "제 환경에서는 잘 돼요"라는 말이 사라집니다.

---

## 1. matrix 전략의 기본 구조

matrix는 `strategy.matrix` 키 아래에 변수 목록을 정의합니다. GitHub Actions는 모든 조합에 대해 독립적인 job을 생성합니다.

```yaml
# .github/workflows/matrix-test.yml

name: Matrix Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ${{ matrix.os }}           # 변수 참조는 ${{ }} 문법 사용
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ["3.10", "3.11", "3.12"]

    steps:
      - name: 코드 내려받기
        uses: actions/checkout@v4

      - name: Python ${{ matrix.python-version }} 설정
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: 의존성 설치
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: 테스트 실행
        run: pytest tests/ -v
```

위 워크플로는 **2 OS × 3 Python 버전 = 6개의 job**을 병렬로 실행합니다.

---

## 2. fail-fast와 continue-on-error

기본적으로 matrix의 job 하나가 실패하면 나머지 job도 즉시 중단됩니다. 이것을 `fail-fast` 옵션으로 제어할 수 있습니다.

```yaml
# .github/workflows/matrix-test.yml (이어서)

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false          # 하나가 실패해도 나머지는 계속 실행
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ["3.10", "3.11", "3.12"]
```

특정 조합만 실패해도 계속 진행하고 싶을 때는 `continue-on-error`를 씁니다:

```yaml
    steps:
      - name: 테스트 실행
        run: pytest tests/ -v
        continue-on-error: ${{ matrix.python-version == '3.10' }}
        # Python 3.10에서만 실패해도 전체 워크플로가 멈추지 않음
```

| 옵션 | 기본값 | 설명 |
|------|--------|------|
| `fail-fast: true` | 기본값 | 하나 실패 → 전체 중단 |
| `fail-fast: false` | 수동 설정 | 하나 실패 → 나머지 계속 |
| `continue-on-error: true` | step 단위 | 해당 step 실패를 무시 |

---

## 3. exclude와 include로 조합 조정하기

모든 조합이 항상 필요하지는 않습니다. 예를 들어 Windows에서는 Python 3.10만 지원하지 않는 상황이라면 `exclude`로 제외합니다.

```yaml
# .github/workflows/matrix-advanced.yml

name: Advanced Matrix

on: [push]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ["3.10", "3.11", "3.12"]
        exclude:
          # Windows + Python 3.10 조합은 제외
          - os: windows-latest
            python-version: "3.10"
        include:
          # 특별히 추가할 조합: macOS + Python 3.12 + 추가 변수
          - os: macos-latest
            python-version: "3.12"
            experimental: true

    steps:
      - uses: actions/checkout@v4

      - name: Python 설정
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}

      - name: 실험적 기능 표시
        if: ${{ matrix.experimental == true }}
        run: echo "이 job은 실험적 환경입니다"

      - name: 테스트 실행
        run: pytest tests/ -v
```

---

## 4. 병렬 테스트로 속도 높이기

matrix 외에도 하나의 테스트 스위트를 여러 job으로 **나눠서** 병렬 실행할 수 있습니다. `pytest-split` 플러그인을 함께 쓰는 방법입니다.

```yaml
# .github/workflows/parallel-test.yml

name: Parallel Tests

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        group: [1, 2, 3]          # 테스트를 3개 그룹으로 분할

    steps:
      - uses: actions/checkout@v4

      - name: Python 설정
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: 의존성 설치
        run: |
          pip install pytest pytest-split

      - name: 테스트 분할 실행
        run: |
          pytest tests/ \
            --splits 3 \
            --group ${{ matrix.group }} \
            -v
```

테스트가 300개라면 각 group이 약 100개씩 나눠받아 **동시에 실행**됩니다. 전체 시간이 약 1/3로 줄어듭니다.

---

## 5. 캐시로 의존성 설치 속도 올리기

matrix 빌드는 job이 많아서 의존성 설치 시간이 곱절로 늘어납니다. `cache` 옵션으로 이를 해결합니다.

```yaml
# .github/workflows/matrix-cache.yml

name: Matrix with Cache

on: [push]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ["3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4

      - name: Python 설정 (캐시 포함)
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: "pip"              # pip 캐시 자동 활성화
          cache-dependency-path: "requirements.txt"

      - name: 의존성 설치
        run: pip install -r requirements.txt

      - name: 테스트 실행
        run: pytest tests/ -v --tb=short
```

`cache: "pip"`를 추가하면 `requirements.txt`가 변경되지 않는 한 캐시를 재사용해서 **의존성 설치 시간을 건너뜁니다**.

---

## 따라 하기 실습

### 실습 1 — 기본 matrix 워크플로 만들기

**목표:** 간단한 Python 프로젝트에서 두 가지 Python 버전으로 테스트를 돌려봅니다.

1. 프로젝트 루트에 `tests/test_calculator.py` 파일을 만듭니다:

```python
# tests/test_calculator.py

def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5

def test_add_negative():
    assert add(-1, 1) == 0
```

2. `.github/workflows/matrix-basic.yml` 파일을 만듭니다:

```yaml
name: Basic Matrix

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.11", "3.12"]

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install pytest
      - run: pytest tests/ -v
```

3. `main` 브랜치에 push하고 GitHub → **Actions** 탭에서 두 개의 job이 병렬로 실행되는 것을 확인합니다.

---

### 실습 2 — exclude 추가하고 결과 비교하기

**목표:** 실습 1의 워크플로에 OS matrix와 exclude를 추가합니다.

1. `matrix-basic.yml`을 아래처럼 수정합니다:

```yaml
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        python-version: ["3.11", "3.12"]
        exclude:
          - os: windows-latest
            python-version: "3.11"
```

2. push 후 Actions 탭에서 job 목록을 확인합니다. **3개의 job**이 실행되어야 합니다 (4조합 - 1제외):
   - ubuntu + 3.11
   - ubuntu + 3.12
   - windows + 3.12

3. GitHub Actions UI에서 각 job 이름이 `test (ubuntu-latest, 3.11)` 형식으로 표시되는지 확인합니다.

---

### 실습 3 — 실패 시나리오 재현 후 fail-fast 이해하기

**목표:** 일부러 테스트를 실패시켜 `fail-fast` 동작 차이를 체험합니다.

1. `tests/test_calculator.py`에 실패하는 테스트를 추가합니다:

```python
def test_fail_on_312():
    import sys
    # Python 3.12에서만 강제 실패
    if sys.version_info >= (3, 12):
        assert False, "의도적 실패: Python 3.12"
```

2. `strategy.fail-fast: true` (기본값)로 push → 3.12 job이 실패하면 나머지 job도 멈추는지 확인합니다.

3. `fail-fast: false`로 변경 후 재push → 3.12 job이 실패해도 나머지 job은 완료되는지 확인합니다.

4. 확인 후 의도적 실패 테스트는 삭제합니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| Python 버전을 숫자로 씀 | `python-version: 3.11` → 실제로 `3.1`로 해석됨 | 반드시 따옴표로 감싸기: `"3.11"` |
| matrix 변수명 오타 | `${{ matrix.python_version }}` → 빈 문자열 | 변수명은 YAML 키와 정확히 일치해야 함: `python-version` |
| exclude 조건이 매칭 안 됨 | 제외하려는 조합이 여전히 실행됨 | `exclude` 항목의 키/값이 matrix 정의와 **정확히** 동일해야 함 |
| `runs-on`에 matrix 변수 미사용 | 모든 job이 같은 OS에서 실행됨 | `runs-on: ${{ matrix.os }}` 로 참조 확인 |
| 캐시 미스가 계속 발생 | 매 실행마다 의존성 재설치 | `cache-dependency-path`를 `requirements.txt` 경로로 정확히 지정 |
| `continue-on-error`를 job 레벨에 둠 | 문법 오류 또는 의도치 않은 동작 | `continue-on-error`는 `jobs.<job>.continue-on-error` 또는 step 레벨에 위치 |
| YAML 들여쓰기 오류 | `mapping values are not allowed here` | `strategy`, `matrix`, `exclude`는 정확한 2칸 들여쓰기 유지 |

---

## 확인 체크리스트

- [ ] `strategy.matrix` 아래에 변수 목록을 정의할 수 있다
- [ ] `${{ matrix.변수명 }}` 문법으로 값을 참조할 수 있다
- [ ] Python 버전 값을 문자열(`"3.11"`)로 표기해야 하는 이유를 설명할 수 있다
- [ ] `fail-fast: false`를 설정하면 어떤 동작이 달라지는지 말할 수 있다
- [ ] `exclude`로 특정 조합을 제외하는 YAML을 직접 작성할 수 있다
- [ ] `include`로 matrix에 없던 조합을 추가하는 방법을 안다
- [ ] `cache: "pip"` 옵션이 왜 matrix 빌드에서 특히 유용한지 설명할 수 있다
- [ ] Actions 탭에서 각 matrix job의 이름 형식(`test (os, version)`)을 확인했다
- [ ] 병렬 테스트 분할을 위해 `--splits`와 `--group` 옵션이 무엇을 하는지 이해했다

---

## 한 번 더 생각해 보기

1. **matrix 조합이 너무 많아지면 어떤 문제가 생길까요?** 3 OS × 5 Python 버전 × 3 Node 버전 = 45개 job이 생깁니다. GitHub Actions의 동시 실행 제한(무료 플랜: 20개)에 걸릴 수 있고, 빌드 시간도 늘어납니다. 어떤 기준으로 조합을 줄이는 것이 합리적일까요?

2. **`fail-fast: true`가 유리한 상황과 `fail-fast: false`가 유리한 상황은 각각 어떤 경우일까요?** 빠른 피드백이 필요한 개발 중에는 어느 쪽이 더 나을지 생각해 보세요.

3. **캐시를 쓰면 항상 빠를까요?** `requirements.txt`가 자주 바뀌는 프로젝트에서는 캐시 히트율이 낮아집니다. 캐시가 오히려 복잡성을 높이는 경우는 언제일지 생각해 보세요.

---

## 다음 장

다음 장에서는 GitHub Actions **Reusable Workflows**를 사용해 여러 레포지토리에서 공통 워크플로를 재사용하는 방법을 배웁니다.