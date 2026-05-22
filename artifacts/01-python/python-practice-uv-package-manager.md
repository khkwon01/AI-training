## 이 장에서 배우는 것

- `uv`가 무엇인지, 왜 `pip`/`venv`보다 빠른지 이해한다
- macOS/Linux와 Windows에 `uv`를 설치한다
- `uv init`으로 새 프로젝트를 만들고, `uv add`로 패키지를 추가한다
- `uv run`으로 스크립트를 실행하고, `uv sync`로 팀원과 환경을 맞춘다
- `pyproject.toml`과 `uv.lock` 파일이 무슨 역할을 하는지 설명할 수 있다
- 기존 `requirements.txt` 프로젝트를 `uv` 방식으로 바꿀 수 있다

---

## 먼저 쉬운 설명

Python을 처음 배울 때 가장 많이 헷갈리는 부분 중 하나가 "패키지 설치"입니다. `pip install requests`를 입력했더니 다른 프로젝트가 망가졌다거나, 가상환경(venv)을 만들었는데 어디에 있는지 모르겠다거나, 팀원이 내 코드를 실행했더니 패키지 버전이 달라서 오류가 났다는 경험, 한 번쯤 해보셨나요?

이런 문제를 해결하기 위해 2024년부터 Python 커뮤니티에서 **uv**라는 도구가 빠르게 표준으로 자리잡고 있습니다. Astral이라는 회사가 만든 `uv`는 Rust로 작성되어 있어서 기존 `pip`보다 **10배에서 100배까지 빠르고**, 가상환경 생성과 패키지 설치를 **한 번에** 처리해 줍니다.

쉽게 비유하자면, `pip`이 동네 슈퍼마켓에서 카트를 직접 끌며 물건을 하나씩 담는 방식이라면, `uv`는 직원이 미리 목록을 보고 창고에서 한 번에 가져다 주는 방식입니다.

이 장을 마치면 새 Python 프로젝트를 시작할 때 `uv`를 자연스럽게 사용할 수 있게 됩니다.

---

## 1. pip/venv와 uv 비교

`uv`가 왜 더 좋은지 표로 먼저 살펴봅시다.

| 항목 | pip + venv (기존 방식) | uv (새 방식) |
|------|----------------------|-------------|
| 설치 속도 | 느림 (패키지마다 순서대로) | 매우 빠름 (병렬 다운로드) |
| 가상환경 생성 | `python -m venv .venv` 별도 실행 | `uv` 명령어가 자동 처리 |
| 패키지 추가 | `pip install requests` | `uv add requests` |
| 의존성 고정 파일 | `requirements.txt` (직접 관리) | `uv.lock` (자동 생성) |
| 프로젝트 설정 파일 | `setup.py` 또는 없음 | `pyproject.toml` (자동 생성) |
| Python 버전 관리 | 별도 도구 필요 (pyenv 등) | `uv python install` |
| 팀원 환경 동기화 | `pip install -r requirements.txt` | `uv sync` |

**핵심 차이점:** `pip`은 "패키지 설치 도구"지만 `uv`는 "프로젝트 전체 환경 관리 도구"입니다. 가상환경 생성, 패키지 설치, 버전 고정, 팀원 동기화까지 `uv` 하나로 처리합니다.

---

## 2. uv 설치하기

### macOS / Linux

터미널을 열고 아래 명령어를 그대로 붙여넣으세요.

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

설치가 끝나면 터미널을 **완전히 닫았다가 다시 열어야** 합니다. 그 다음 아래 명령어로 설치가 잘 됐는지 확인합니다.

```bash
uv --version
```

출력 예시:
```
uv 0.5.21 (2026-01-15)
```

버전 숫자가 나오면 성공입니다.

### Windows (PowerShell)

시작 메뉴에서 **PowerShell**을 찾아 실행하고, 아래 명령어를 붙여넣으세요.

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

설치 후 PowerShell을 닫았다가 다시 열고 확인합니다.

```powershell
uv --version
```

### 설치 후 흔한 오류

```
uv: command not found
```

이 오류가 나오면 터미널을 완전히 닫지 않고 재실행한 경우입니다. 터미널 창을 완전히 닫고 새로 열어서 다시 시도하세요. 그래도 안 되면 아래 명령어로 경로를 직접 추가해 보세요.

```bash
# macOS/Linux
source $HOME/.local/bin/env

# 이후 ~/.bashrc 또는 ~/.zshrc 파일 맨 아래에 아래 줄 추가
export PATH="$HOME/.local/bin:$PATH"
```

---

## 3. 새 프로젝트 만들기 — uv init

프로젝트를 시작할 때 가장 먼저 `uv init`을 사용합니다.

```bash
# 새 폴더를 만들면서 프로젝트 초기화
uv init my-first-project

# 폴더 이동
cd my-first-project
```

`uv init`이 자동으로 만들어 주는 파일들:

```
my-first-project/
├── pyproject.toml   ← 프로젝트 설정과 의존성 목록
├── .python-version  ← 이 프로젝트에서 사용할 Python 버전
├── README.md        ← 프로젝트 설명 파일
└── hello.py         ← 예제 Python 파일
```

생성된 `pyproject.toml` 파일을 열어 보면 이렇게 생겼습니다.

```toml
[project]
name = "my-first-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = []
```

아직 아무 패키지도 설치하지 않아서 `dependencies`가 비어 있습니다. 이제 패키지를 추가해 봅시다.

### 이미 있는 폴더에서 초기화하기

기존 작업 중인 폴더가 있다면 그 폴더 안에서 바로 `uv init`을 실행해도 됩니다.

```bash
cd 기존-프로젝트-폴더
uv init
```

---

## 4. 패키지 추가하기 — uv add

`pip install` 대신 `uv add`를 사용합니다. 아래 예시에서는 웹 요청에 많이 쓰이는 `requests` 패키지를 추가해 봅니다.

```bash
uv add requests
```

실행하면 이런 출력이 나옵니다.

```
Resolved 5 packages in 312ms
Downloaded 5 packages in 1.2s
Installed 5 packages in 89ms
 + certifi==2024.12.14
 + charset-normalizer==3.4.0
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3
```

`pip install`보다 훨씬 빠른 것을 느낄 수 있습니다. 이제 `pyproject.toml`을 열어보면 자동으로 변경되어 있습니다.

```toml
[project]
name = "my-first-project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
    "requests>=2.32.3",
]
```

`requests`가 `dependencies` 목록에 자동으로 추가되었습니다. 그리고 `uv.lock` 파일도 새로 생겼을 것입니다.

### uv.lock 파일이란?

```
my-first-project/
├── pyproject.toml   ← "requests 패키지가 필요해요"
├── uv.lock          ← "requests 2.32.3 정확히 이 버전이에요" (자동 생성)
├── .venv/           ← 실제 패키지가 설치된 가상환경 폴더
└── hello.py
```

`pyproject.toml`은 "어떤 패키지가 필요한지"를 적은 장보기 목록이고, `uv.lock`은 "정확히 어떤 버전을 설치했는지"를 기록한 영수증입니다. `uv.lock` 파일 덕분에 팀원이 같은 버전을 설치할 수 있습니다.

> **중요:** `uv.lock` 파일은 직접 수정하지 마세요. `uv`가 자동으로 관리합니다.

### 여러 패키지 한 번에 추가하기

```bash
uv add pandas numpy matplotlib
```

### 특정 버전 지정하기

```bash
uv add "requests==2.31.0"
uv add "pandas>=2.0,<3.0"
```

### 개발할 때만 필요한 패키지 추가하기

테스트 도구처럼 실제 배포할 때는 필요 없는 패키지는 `--dev` 옵션을 붙입니다.

```bash
uv add --dev pytest black ruff
```

---

## 5. 스크립트 실행하기 — uv run

`uv run`은 프로젝트의 가상환경을 자동으로 사용해서 Python 스크립트를 실행합니다. 가상환경을 직접 활성화(`source .venv/bin/activate`)할 필요가 없습니다.

`hello.py` 파일에 아래 코드를 작성해 봅시다.

```python
# hello.py
import requests

response = requests.get("https://httpbin.org/json")
data = response.json()

print("상태 코드:", response.status_code)
print("응답 데이터:", data)
```

이제 실행합니다.

```bash
uv run hello.py
```

출력:
```
상태 코드: 200
응답 데이터: {'slideshow': {'author': 'Yours Truly', ...}}
```

가상환경을 활성화하지 않아도 `requests` 패키지를 사용할 수 있습니다.

### Python 인터랙티브 셸 실행

```bash
uv run python
```

### 모듈 실행 (-m 옵션)

```bash
uv run python -m pytest
uv run python -m black hello.py
```

### 기존 방식과 비교

```bash
# 기존 pip/venv 방식 (매번 활성화 필요)
source .venv/bin/activate   # macOS/Linux
.venv\Scripts\activate      # Windows
python hello.py
deactivate

# uv 방식 (활성화 불필요)
uv run hello.py
```

---

## 6. 팀원과 환경 맞추기 — uv sync

팀원이 내 코드를 받아서 실행하려면 같은 패키지를 설치해야 합니다. `uv sync`가 이 역할을 합니다.

```bash
# 팀원이 내 프로젝트를 git clone한 후
git clone https://github.com/내-계정/my-first-project.git
cd my-first-project

# 이 한 줄로 모든 패키지 설치 완료
uv sync
```

`uv.lock` 파일을 보고 정확히 같은 버전의 패키지들을 설치합니다. 이제 "내 컴퓨터에서는 됐는데 네 컴퓨터에서는 왜 안 돼?" 문제가 사라집니다.

> **팁:** `uv.lock` 파일은 반드시 git에 커밋하세요. 이 파일이 있어야 팀원들이 `uv sync`로 동일한 환경을 재현할 수 있습니다.

### .gitignore에 추가할 항목

```gitignore
# .gitignore
.venv/           ← 가상환경 폴더는 git에 올리지 않음
__pycache__/
*.pyc
```

`uv.lock`과 `pyproject.toml`은 **반드시 git에 포함**시키고, `.venv/` 폴더는 **반드시 제외**합니다.

---

## 7. 패키지 제거와 목록 확인

### 패키지 제거

```bash
uv remove requests
```

`pyproject.toml`에서 `requests`가 삭제되고, `uv.lock`도 자동으로 업데이트됩니다.

### 설치된 패키지 목록 확인

```bash
uv pip list
```

출력:
```
Package            Version
------------------ ----------
certifi            2024.12.14
charset-normalizer 3.4.0
idna               3.10
requests           2.32.3
urllib3            2.2.3
```

### 패키지 정보 확인

```bash
uv pip show requests
```

---

## 8. 기존 requirements.txt 프로젝트 마이그레이션

`pip`으로 관리하던 프로젝트를 `uv`로 전환하는 방법입니다.

### 기존 프로젝트 구조 (pip 방식)

```
old-project/
├── requirements.txt
├── app.py
└── README.md
```

`requirements.txt` 내용 예시:
```
flask==3.0.0
requests==2.31.0
python-dotenv==1.0.0
```

### 마이그레이션 단계

**1단계:** 프로젝트 폴더에서 `uv init` 실행 (기존 파일은 유지됨)

```bash
cd old-project
uv init --no-workspace
```

`--no-workspace` 옵션은 이미 파일이 있는 폴더에서 초기화할 때 충돌을 방지합니다.

**2단계:** `requirements.txt`의 패키지들을 `uv add`로 추가

```bash
uv add flask==3.0.0 requests==2.31.0 python-dotenv==1.0.0
```

또는 한 번에 마이그레이션:

```bash
uv add $(cat requirements.txt | grep -v '^#' | grep -v '^$' | tr '\n' ' ')
```

**3단계:** 잘 작동하는지 확인

```bash
uv run python app.py
```

**4단계:** 기존 `requirements.txt`는 보관하거나 삭제

```bash
# requirements.txt를 아직 유지하고 싶다면 uv로 다시 생성 가능
uv pip freeze > requirements.txt
```

### 마이그레이션 후 구조

```
old-project/
├── pyproject.toml   ← 새로 생성됨
├── uv.lock          ← 새로 생성됨
├── .venv/           ← 새로 생성됨
├── requirements.txt ← 선택적으로 유지 가능
├── app.py
└── README.md
```

---

## 따라 하기 실습

### 실습 1 — 날씨 정보 출력 프로그램 만들기

이 실습에서는 `httpx` 패키지를 사용해 공개 API에서 데이터를 받아오는 프로그램을 만듭니다.

**1단계:** 새 프로젝트 만들기

```bash
uv init weather-checker
cd weather-checker
```

**2단계:** 필요한 패키지 추가

```bash
uv add httpx
```

**3단계:** `weather.py` 파일 작성

```python
# weather.py
import httpx


def get_ip_location():
    """현재 IP 주소의 위치 정보를 가져옵니다."""
    response = httpx.get("https://ipapi.co/json/")
    response.raise_for_status()
    return response.json()


def main():
    print("현재 위치 정보를 가져오는 중...")
    
    location = get_ip_location()
    
    print(f"\n--- 현재 위치 ---")
    print(f"도시: {location.get('city', '알 수 없음')}")
    print(f"국가: {location.get('country_name', '알 수 없음')}")
    print(f"시간대: {location.get('timezone', '알 수 없음')}")
    print(f"IP 주소: {location.get('ip', '알 수 없음')}")


if __name__ == "__main__":
    main()
```

**4단계:** 실행

```bash
uv run weather.py
```

예상 출력:
```
현재 위치 정보를 가져오는 중...

--- 현재 위치 ---
도시: Seoul
국가: South Korea
시간대: Asia/Seoul
IP 주소: 123.456.789.000
```

**5단계:** 프로젝트 구조 확인

```bash
ls -la
```

```
drwxr-xr-x  .venv/
-rw-r--r--  .python-version
-rw-r--r--  pyproject.toml
-rw-r--r--  README.md
-rw-r--r--  uv.lock
-rw-r--r--  weather.py
```

---

### 실습 2 — 텍스트 분석 도구 만들기

이 실습은 실습 1에서 배운 내용 위에 **개발용 패키지**와 **여러 패키지 사용**을 추가합니다.

**1단계:** 새 프로젝트 만들기

```bash
uv init text-analyzer
cd text-analyzer
```

**2단계:** 일반 패키지와 개발용 패키지 구분해서 추가

```bash
# 프로그램 실행에 필요한 패키지
uv add rich

# 개발할 때만 필요한 패키지 (테스트, 코드 검사)
uv add --dev pytest ruff
```

**3단계:** `analyzer.py` 파일 작성

```python
# analyzer.py
from rich.console import Console
from rich.table import Table

console = Console()


def analyze_text(text: str) -> dict:
    """텍스트의 기본 통계를 분석합니다."""
    words = text.split()
    sentences = [s.strip() for s in text.split('.') if s.strip()]
    
    return {
        "글자 수 (공백 포함)": len(text),
        "글자 수 (공백 제외)": len(text.replace(" ", "")),
        "단어 수": len(words),
        "문장 수": len(sentences),
        "평균 단어 길이": round(sum(len(w) for w in words) / len(words), 1) if words else 0,
    }


def print_analysis(text: str):
    """분석 결과를 예쁘게 출력합니다."""
    result = analyze_text(text)
    
    table = Table(title="텍스트 분석 결과", show_header=True)
    table.add_column("항목", style="cyan")
    table.add_column("값", style="green", justify="right")
    
    for key, value in result.items():
        table.add_row(key, str(value))
    
    console.print(table)


def main():
    sample_text = """
    Python은 읽기 쉽고 배우기 쉬운 프로그래밍 언어입니다.
    다양한 분야에서 사용되며 초보자에게도 적합합니다.
    데이터 분석, 웹 개발, 인공지능 등 여러 용도로 활용됩니다.
    """.strip()
    
    console.print("\n[bold yellow]분석할 텍스트:[/bold yellow]")
    console.print(sample_text)
    console.print()
    
    print_analysis(sample_text)


if __name__ == "__main__":
    main()
```

**4단계:** 실행

```bash
uv run analyzer.py
```

**5단계:** `pyproject.toml` 확인 — 일반 패키지와 개발 패키지가 구분되어 있는지 봅니다.

```toml
[project]
name = "text-analyzer"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "rich>=13.9.4",
]

[dependency-groups]
dev = [
    "pytest>=8.3.4",
    "ruff>=0.8.6",
]
```

---

### 실습 3 — 기존 pip 프로젝트 마이그레이션

이 실습은 `pip`으로 만든 가상의 기존 프로젝트를 `uv`로 전환합니다.

**1단계:** 기존 프로젝트 흉내내기

```bash
mkdir legacy-project
cd legacy-project
```

`requirements.txt` 파일 만들기:

```
requests>=2.28.0
python-dotenv>=1.0.0
click>=8.0.0
```

`main.py` 파일 만들기:

```python
# main.py
import click
import requests
from dotenv import load_dotenv
import os

load_dotenv()


@click.command()
@click.argument('url')
def fetch(url):
    """URL의 내용을 가져와서 상태 코드를 출력합니다."""
    try:
        response = requests.get(url, timeout=5)
        click.echo(f"상태 코드: {response.status_code}")
        click.echo(f"콘텐츠 타입: {response.headers.get('content-type', '알 수 없음')}")
        click.echo(f"응답 크기: {len(response.content)} bytes")
    except requests.exceptions.ConnectionError:
        click.echo("오류: 연결할 수 없습니다.", err=True)
    except requests.exceptions.Timeout:
        click.echo("오류: 시간이 초과되었습니다.", err=True)


if __name__ == "__main__":
    fetch()
```

**2단계:** `uv init`으로 마이그레이션 시작

```bash
uv init
```

**3단계:** `requirements.txt`에서 패키지 불러오기

```bash
uv add requests python-dotenv click
```

**4단계:** 실행 테스트

```bash
uv run main.py https://httpbin.org/get
```

출력:
```
상태 코드: 200
콘텐츠 타입: application/json
응답 크기: 408 bytes
```

**5단계:** `requirements.txt` 업데이트 (원하는 경우)

```bash
uv pip freeze > requirements.txt
```

마이그레이션 완료입니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|------|---------------------|-----------|
| `pip install`을 그냥 씀 | 패키지는 설치됐지만 `pyproject.toml`에 반영 안 됨 | `uv add 패키지명`을 사용할 것 |
| `uv.lock`을 `.gitignore`에 추가함 | 팀원이 `uv sync` 실행 시 다른 버전 설치될 수 있음 | `uv.lock`은 git에 포함시켜야 함 |
| `.venv`를 git에 올림 | 레포지토리 용량이 크게 늘어남 | `.gitignore`에 `.venv/` 추가 |
| 터미널 재시작 안 함 | `uv: command not found` | 터미널을 완전히 닫고 다시 열기 |
| `uv init` 없이 `uv add` 실행 | `error: No `pyproject.toml` found` | 먼저 `uv init` 실행 후 `uv add` 사용 |
| Python 파일을 `python hello.py`로 실행 | 가상환경 밖에서 실행되어 패키지 못 찾음 | `uv run hello.py`로 실행 |
| `uv.lock` 파일을 직접 수정함 | 패키지 충돌 또는 설치 실패 | `uv lock --upgrade` 또는 `uv sync`로 재생성 |
| `uv remove` 대신 파일에서 직접 삭제 | `uv.lock`과 `pyproject.toml` 불일치 | `uv remove 패키지명`을 사용할 것 |

### 자주 만나는 오류 메시지 상세 설명

**오류 1:**
```
error: No `pyproject.toml` found in `/Users/사용자/프로젝트` or any parent directory
```
→ `uv init`을 먼저 실행하지 않은 경우입니다. `uv init`을 실행하세요.

**오류 2:**
```
ModuleNotFoundError: No module named 'requests'
```
→ `python hello.py`처럼 가상환경 밖에서 실행했거나 `uv add requests`를 하지 않은 경우입니다. `uv add requests` 후 `uv run hello.py`로 실행하세요.

**오류 3:**
```
error: Distribution `requests` requires a different Python version
```
→ Python 버전이 맞지 않는 경우입니다. `uv python install 3.12`로 필요한 버전을 설치하세요.

---

## 확인 체크리스트

아래 항목을 스스로 확인해 보세요.

- [ ] `uv --version`을 실행하면 버전 번호가 출력된다
- [ ] `uv init 프로젝트명`으로 새 프로젝트를 만들 수 있다
- [ ] `uv add requests` 후 `pyproject.toml`에 `requests`가 추가된 것을 확인했다
- [ ] `uv.lock` 파일이 자동으로 생성된 것을 확인했다
- [ ] `uv run 파일명.py`로 스크립트를 실행할 수 있다
- [ ] `uv add --dev pytest`로 개발 전용 패키지를 따로 추가했다
- [ ] `uv remove 패키지명`으로 패키지를 제거하면 `pyproject.toml`에서도 삭제된다
- [ ] `.venv/` 폴더를 `.gitignore`에 추가했다
- [ ] `uv.lock` 파일을 git에 커밋해야 한다는 것을 이해했다
- [ ] `pip install` 대신 `uv add`를, `python 파일.py` 대신 `uv run 파일.py`를 쓴다

---

## 한 번 더 생각해 보기

1. **팀 협업 상황을 상상해 보세요.** 내가 `uv add pandas`를 실행하고 코드를 작성한 뒤 git에 올렸습니다. 팀원은 내 코드를 `git clone`한 후 `uv sync`를 실행했습니다. 팀원은 왜 `uv add pandas`를 따로 실행하지 않아도 `pandas`가 설치될까요? 어떤 파일 덕분인지 설명해 보세요.

2. **`pip`과 `uv`의 철학 차이를 생각해 보세요.** `pip install requests`는 현재 Python 환경(글로벌 또는 활성화된 venv)에 패키지를 설치합니다. `uv add requests`는 프로젝트 폴더의 `pyproject.toml`을 업데이트하고 `.venv`에 설치합니다. 이 차이가 "내 컴퓨터에서는 됐는데 서버에서는 왜 안 되지?" 문제를 어떻게 줄여주는지 설명해 보세요.

3. **`pyproject.toml`과 `uv.lock`의 역할 차이를 정리해 보세요.** 요리에 비유하면 `pyproject.toml`은 레시피(재료 목록)이고 `uv.lock`은 장보기 영수증(정확한 브랜드와 수량)입니다. 만약 `uv.lock` 없이 `pyproject.toml`만 있다면 팀원들의 환경에서 어떤 문제가 생길 수 있을까요?

---

## 다음 장

다음 장에서는 `uv`로 구성한 Python 환경을 GitHub Actions CI/CD 파이프라인에 연결해서 코드를 자동으로 테스트하는 방법을 배웁니다.