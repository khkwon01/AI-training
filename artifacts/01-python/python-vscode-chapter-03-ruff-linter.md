## 이 장에서 배우는 것

- `ruff`가 무엇인지, 왜 Python 개발자들이 사용하는지 이해한다
- VS Code에 `ruff` 확장 프로그램을 설치하고 설정하는 방법을 익힌다
- `ruff`가 코드의 문제를 자동으로 찾아주고 고쳐주는 과정을 경험한다
- 프로젝트에 `ruff` 설정 파일을 만들어 팀 전체가 같은 규칙을 쓸 수 있게 한다

---

## 먼저 쉬운 설명

글을 쓸 때 맞춤법 검사기를 쓰면 오타나 문법 실수를 빠르게 찾을 수 있죠? Python 코드에도 똑같은 역할을 하는 도구가 있습니다. 이런 도구를 **린터(linter)** 라고 부릅니다.

`ruff`는 Python 린터 중에서 가장 빠르고 인기 있는 도구입니다. Rust 언어로 만들어졌기 때문에 기존 도구들보다 **10~100배 빠르게** 코드를 검사합니다.

린터가 없으면 이런 일이 생깁니다:

- 쓰지 않는 변수를 선언해도 모른다
- import 순서가 뒤죽박죽이어도 모른다
- 팀원마다 코드 스타일이 달라서 협업이 어려워진다

`ruff`를 VS Code에 연결하면 파일을 저장하는 순간 이런 문제들을 바로 알려줍니다. 나중에 코드 리뷰에서 지적받기 전에 미리 고칠 수 있습니다.

---

## 1. ruff 설치하기

`ruff`를 사용하려면 먼저 Python 패키지로 설치해야 합니다.

**터미널에서 실행:**

```bash
# 가상환경이 활성화된 상태에서 실행합니다
pip install ruff
```

설치가 잘 됐는지 확인합니다:

```bash
ruff --version
```

아래와 같이 버전이 출력되면 성공입니다:

```
ruff 0.4.4
```

> **팁:** 프로젝트마다 가상환경을 따로 만들어 `ruff`를 설치하는 것이 좋습니다. 프로젝트별로 버전을 다르게 관리할 수 있기 때문입니다.

---

## 2. VS Code에 ruff 확장 프로그램 설치하기

터미널에서 `ruff`를 쓸 수도 있지만, VS Code와 연결하면 코드를 작성하는 동안 실시간으로 피드백을 받을 수 있습니다.

**설치 순서:**

1. VS Code를 엽니다
2. 왼쪽 사이드바에서 확장 프로그램 아이콘(네모 4개)을 클릭합니다
3. 검색창에 `ruff`를 입력합니다
4. **"Ruff" (Astral Software 제작)** 을 찾아 설치합니다

설치 후 VS Code를 다시 시작하면 확장 프로그램이 활성화됩니다.

**설치 확인 방법:**

다음 코드를 담은 파일을 열어보세요. 노란색 또는 빨간색 밑줄이 생기면 `ruff`가 동작하는 것입니다.

```python
# test_ruff.py
import os
import sys

x = 10
y = 20
print(x)
```

`sys`와 `y`에 밑줄이 생기고, 마우스를 올리면 오류 메시지가 나타납니다:

```
F401 `sys` imported but unused
F841 Local variable `y` is assigned to but never used
```

---

## 3. ruff 설정 파일 만들기

`ruff`는 프로젝트 루트에 설정 파일을 두면 팀 전체가 같은 규칙을 사용할 수 있습니다. 설정 파일 이름은 `ruff.toml` 또는 `pyproject.toml`을 사용합니다.

**`ruff.toml` 예시 (초보자 추천):**

```toml
# ruff.toml

# 한 줄의 최대 글자 수
line-length = 88

# 검사할 규칙 선택
[lint]
select = [
    "E",   # pycodestyle 오류
    "F",   # pyflakes (미사용 변수, import 등)
    "I",   # isort (import 정렬)
    "W",   # pycodestyle 경고
]

# 무시할 규칙 (필요할 때만 추가)
ignore = []
```

**실제 프로젝트 예시 — `pyproject.toml` 사용:**

```toml
# pyproject.toml

[project]
name = "my-python-app"
version = "0.1.0"

[tool.ruff]
line-length = 88

[tool.ruff.lint]
select = ["E", "F", "I", "W"]
ignore = ["E501"]  # 긴 줄 경고는 무시
```

> **참고:** `pyproject.toml`은 여러 도구의 설정을 한 파일에 모을 수 있어서 프로젝트가 커질수록 편리합니다.

---

## 4. 저장할 때 자동으로 고치기 (Format on Save)

`ruff`는 린터 기능뿐만 아니라 코드 포매터 기능도 있습니다. VS Code 설정을 바꾸면 파일을 저장할 때 자동으로 코드를 정리할 수 있습니다.

**VS Code 설정 방법:**

1. `Ctrl + Shift + P` (Mac: `Cmd + Shift + P`) 를 눌러 명령 팔레트를 엽니다
2. `Open User Settings (JSON)` 을 입력하고 선택합니다
3. `settings.json` 파일에 다음 내용을 추가합니다:

```json
{
    "[python]": {
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.fixAll.ruff": "explicit",
            "source.organizeImports.ruff": "explicit"
        }
    }
}
```

**효과 확인:**

다음 코드를 작성하고 저장해 보세요:

```python
# before_save.py — 저장 전
import sys
import os
import json

def greet( name ):
    print( "안녕하세요, " + name )
```

저장하면 자동으로 이렇게 바뀝니다:

```python
# after_save.py — 저장 후 (ruff가 자동 정리)
import json
import os

def greet(name):
    print("안녕하세요, " + name)
```

`sys`는 사용하지 않으므로 삭제되고, import 순서가 알파벳순으로 정렬되고, 불필요한 공백이 제거됩니다.

---

## 따라 하기 실습

### 실습 1 — ruff로 문제 코드 분석하기

프로젝트 폴더에 `calculator.py` 파일을 만들고 아래 코드를 붙여넣으세요:

```python
# calculator.py
import math
import os
import random

def add(a,b):
    result = a+b
    unused_var = "나는 쓰이지 않아요"
    return result

def subtract( a, b ):
    return a-b

PI = 3.14159
```

터미널에서 ruff를 실행해 어떤 문제가 있는지 확인합니다:

```bash
ruff check calculator.py
```

출력 결과를 읽고, 어떤 줄에 어떤 문제가 있는지 파악합니다.

---

### 실습 2 — ruff로 자동 수정하기

실습 1에서 분석한 `calculator.py`를 ruff가 자동으로 고치게 합니다:

```bash
ruff check --fix calculator.py
```

고쳐진 파일을 열어 확인하세요. 자동으로 고칠 수 없는 문제(예: 사용하지 않는 변수)는 직접 삭제해야 합니다.

최종 목표 코드:

```python
# calculator.py — 정리 후
def add(a, b):
    result = a + b
    return result

def subtract(a, b):
    return a - b

PI = 3.14159
```

---

### 실습 3 — 프로젝트에 ruff 설정 파일 추가하기

프로젝트 폴더 최상단에 `ruff.toml` 파일을 만들고 아래 내용을 작성하세요:

```toml
# ruff.toml
line-length = 79

[lint]
select = ["E", "F", "I"]
ignore = ["E402"]
```

설정을 저장한 뒤 다시 `ruff check calculator.py`를 실행합니다. 이제 한 줄 최대 길이 기준이 79자로 바뀐 것을 확인하세요. 설정 파일이 있으면 터미널에서 매번 옵션을 입력할 필요가 없습니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 해결 방법 |
|------|-------------|-----------|
| 가상환경 밖에서 ruff 설치 | `ModuleNotFoundError` 또는 명령어가 없다고 뜸 | `source venv/bin/activate` 로 가상환경 활성화 후 설치 |
| VS Code가 ruff를 못 찾음 | `Ruff: Unable to find ruff` | VS Code에서 사용하는 Python 인터프리터가 ruff가 설치된 가상환경인지 확인 (`Ctrl+Shift+P` → `Python: Select Interpreter`) |
| 저장해도 자동 수정이 안 됨 | 아무 변화 없음 | `settings.json`에서 `"editor.formatOnSave": true` 와 `"editor.defaultFormatter": "charliermarsh.ruff"` 가 올바르게 입력됐는지 확인 |
| `ruff.toml`이 적용 안 됨 | 설정이 무시됨 | 파일이 프로젝트 **루트** 폴더에 있는지 확인. 하위 폴더에 있으면 인식되지 않음 |
| `E501` 경고가 너무 많이 뜸 | `E501 Line too long (XX > 88 characters)` | `ruff.toml`의 `ignore`에 `"E501"` 추가, 또는 `line-length`를 늘림 |

---

## 확인 체크리스트

- [ ] 터미널에서 `ruff --version`을 실행했을 때 버전 번호가 출력된다
- [ ] VS Code 확장 프로그램 목록에서 "Ruff (Astral Software)"가 설치되어 있다
- [ ] 사용하지 않는 import가 있는 파일을 열었을 때 노란 밑줄이 생긴다
- [ ] `ruff check 파일명.py` 명령어로 터미널에서 검사 결과를 볼 수 있다
- [ ] `ruff check --fix 파일명.py` 명령어로 자동 수정이 된다
- [ ] `ruff.toml` 또는 `pyproject.toml`에 ruff 설정이 추가되어 있다
- [ ] 파일을 저장하면 import 순서가 자동으로 정렬된다

---

## 한 번 더 생각해 보기

1. `ruff`가 고칠 수 없다고 알려주는 문제(예: `unused_var`)는 왜 자동으로 고치지 않을까요? 자동으로 삭제했을 때 어떤 위험이 있을지 생각해 보세요.

2. 팀 프로젝트에서 `ruff.toml` 파일을 Git에 포함시켜야 할까요, 말아야 할까요? 포함시켰을 때와 포함시키지 않았을 때 각각 어떤 일이 생길지 생각해 보세요.

3. 린터 규칙 중에는 "경고(warning)"와 "오류(error)"가 있습니다. 둘의 차이는 무엇이고, 여러분의 프로젝트에서는 어느 수준까지 규칙을 적용하는 것이 좋을까요?

---

## 다음 장

다음 장에서는 `ruff`와 함께 쓰면 더욱 강력해지는 타입 검사 도구 `mypy`를 VS Code에 설정하는 방법을 배웁니다.