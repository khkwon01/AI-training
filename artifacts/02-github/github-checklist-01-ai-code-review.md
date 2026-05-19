## 이 장에서 배우는 것

- AI가 생성한 코드를 리뷰할 때 무엇을 확인해야 하는지 이해한다
- GitHub Pull Request에서 코드 리뷰 체크리스트 템플릿을 작성하는 방법을 배운다
- 체크리스트를 `.github` 폴더에 설정해 팀 전체가 자동으로 사용하게 만든다
- AI 코드의 흔한 문제 패턴(보안, 예외 처리, 테스트 누락)을 직접 확인할 수 있다

---

## 먼저 쉬운 설명

요즘은 Claude나 Copilot 같은 AI 도구로 코드를 빠르게 만들 수 있습니다. 그런데 AI가 만든 코드는 **겉으로 보면 그럴듯하지만 실제로 실행하면 문제가 생기는 경우**가 종종 있습니다.

예를 들어 이런 상황이 자주 일어납니다:

- AI가 만든 코드에 `try/except`가 없어서 오류가 나도 프로그램이 아무 말 없이 멈춘다
- API 키가 코드 안에 그냥 들어가 있어서 GitHub에 올리는 순간 보안 사고가 생긴다
- 테스트 코드가 없어서 나중에 어디서 문제가 생겼는지 찾기가 너무 힘들다

이런 문제를 막으려면 **코드를 GitHub에 합치기 전에 체계적으로 확인하는 습관**이 필요합니다. 그 도구가 바로 **코드 리뷰 체크리스트 템플릿**입니다.

체크리스트를 `.github/pull_request_template.md` 파일에 저장해 두면, Pull Request를 열 때마다 자동으로 체크리스트가 나타납니다. 팀원 모두가 같은 기준으로 AI 코드를 검토할 수 있습니다.

---

## 1. GitHub Pull Request 템플릿이란?

`.github/pull_request_template.md` 파일을 만들면 GitHub가 PR을 열 때 그 내용을 본문에 자동으로 채워줍니다. 개발자가 직접 체크리스트를 입력할 필요 없이, 열자마자 확인 항목이 나타납니다.

### 파일 위치 구조

```
my-project/
├── .github/
│   └── pull_request_template.md   ← 이 파일을 만든다
├── src/
│   └── main.py
└── README.md
```

### 기본 템플릿 파일 예시 (`.github/pull_request_template.md`)

```markdown
## 변경 내용 요약

<!-- 이 PR에서 무엇을 바꿨는지 한두 줄로 설명하세요 -->

## AI 코드 리뷰 체크리스트

### 기능 동작
- [ ] 코드가 의도한 대로 실제로 동작하는지 직접 실행해 확인했다
- [ ] 엣지 케이스(빈 값, 아주 큰 숫자, 특수문자 등)를 테스트했다

### 보안
- [ ] API 키, 비밀번호, 토큰이 코드에 하드코딩되어 있지 않다
- [ ] 사용자 입력값을 그대로 SQL이나 명령어에 사용하지 않는다

### 에러 처리
- [ ] 예외(Exception)가 발생할 수 있는 곳에 `try/except`가 있다
- [ ] 에러 메시지가 사용자에게 도움이 되는 내용을 담고 있다

### 테스트
- [ ] 새로운 함수에 대한 테스트 코드를 작성했다
- [ ] 기존 테스트가 모두 통과한다

### 코드 품질
- [ ] 함수 이름과 변수 이름이 역할을 잘 설명하고 있다
- [ ] 중복된 코드가 없다
```

---

## 2. AI 코드에서 자주 빠지는 보안 문제 확인하기

AI는 빠르게 코드를 만들어 주지만, 보안에 취약한 패턴을 그냥 생성하기도 합니다. 리뷰 체크리스트에서 가장 중요한 항목 중 하나가 **보안 확인**입니다.

### AI가 자주 생성하는 위험한 코드 패턴

```python
# 위험한 예시 — API 키가 코드에 직접 들어가 있음
import openai

openai.api_key = "sk-proj-abc123xyz456"  # ← 절대 이렇게 하면 안 됨

def ask_ai(question):
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": question}]
    )
    return response.choices[0].message.content
```

### 올바른 방법 — 환경 변수 사용

```python
# 안전한 예시 — 환경 변수에서 키를 읽어옴
import os
import openai

openai.api_key = os.environ.get("OPENAI_API_KEY")  # ← 이렇게 해야 함

if not openai.api_key:
    raise ValueError("OPENAI_API_KEY 환경 변수가 설정되지 않았습니다")

def ask_ai(question):
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": question}]
    )
    return response.choices[0].message.content
```

### `.env` 파일과 `.gitignore` 설정

```bash
# .env 파일 (절대 GitHub에 올리면 안 됨)
OPENAI_API_KEY=sk-proj-abc123xyz456

# .gitignore 파일에 반드시 추가
.env
*.env
```

---

## 3. 예외 처리 확인하기

AI가 생성한 코드는 흔히 **예외 처리가 없거나 너무 광범위**합니다. 체크리스트에서 이 부분을 꼭 확인해야 합니다.

### AI가 자주 만드는 문제 있는 패턴

```python
# 문제 있는 예시 1 — 예외 처리가 전혀 없음
import requests

def get_user_data(user_id):
    response = requests.get(f"https://api.example.com/users/{user_id}")
    data = response.json()          # 요청 실패하면 여기서 프로그램이 죽는다
    return data["name"]             # "name" 키가 없으면 KeyError 발생

# 문제 있는 예시 2 — 너무 넓은 예외 처리
def get_user_data(user_id):
    try:
        response = requests.get(f"https://api.example.com/users/{user_id}")
        return response.json()["name"]
    except Exception:
        pass                        # 오류가 생겨도 아무것도 모름 (침묵하는 실패)
```

### 좋은 예외 처리 패턴

```python
# 좋은 예시 — 구체적인 예외를 처리하고 의미 있는 메시지를 남김
import requests
import logging

logger = logging.getLogger(__name__)

def get_user_data(user_id: int) -> str:
    """사용자 ID로 이름을 가져온다."""
    try:
        response = requests.get(
            f"https://api.example.com/users/{user_id}",
            timeout=5
        )
        response.raise_for_status()   # 4xx, 5xx 응답이면 예외 발생
        data = response.json()
        return data["name"]

    except requests.exceptions.Timeout:
        logger.error("사용자 데이터 요청 시간 초과: user_id=%s", user_id)
        raise
    except requests.exceptions.HTTPError as e:
        logger.error("API 오류 발생: %s", e)
        raise
    except KeyError:
        logger.error("응답에 'name' 필드가 없음: user_id=%s", user_id)
        raise ValueError(f"사용자 {user_id}의 데이터 형식이 올바르지 않습니다")
```

---

## 4. 테스트 코드 확인하기

AI는 테스트 코드를 따로 작성하지 않는 경우가 많습니다. 리뷰 체크리스트에서 **새 기능에 테스트가 있는지** 반드시 확인해야 합니다.

### 테스트가 없는 AI 생성 코드 예시

```python
# src/calculator.py — AI가 만들어 준 코드, 테스트 없음
def divide(a, b):
    return a / b  # b가 0이면 ZeroDivisionError 발생하는데 테스트가 없음
```

### 리뷰 후 추가해야 하는 테스트 코드

```python
# tests/test_calculator.py — 리뷰어가 요청해서 추가된 테스트
import pytest
from src.calculator import divide

def test_divide_정상_케이스():
    assert divide(10, 2) == 5.0

def test_divide_소수점():
    assert divide(7, 2) == 3.5

def test_divide_0으로_나누기():
    with pytest.raises(ZeroDivisionError):
        divide(10, 0)

def test_divide_음수():
    assert divide(-10, 2) == -5.0
```

```bash
# 테스트 실행 명령어
pytest tests/test_calculator.py -v

# 출력 예시
tests/test_calculator.py::test_divide_정상_케이스 PASSED
tests/test_calculator.py::test_divide_소수점 PASSED
tests/test_calculator.py::test_divide_0으로_나누기 PASSED
tests/test_calculator.py::test_divide_음수 PASSED
```

---

## 따라 하기 실습

### 실습 1 — `.github` 폴더와 PR 템플릿 파일 만들기

프로젝트 루트에서 아래 명령어를 실행합니다.

```bash
# 1. .github 폴더 만들기
mkdir -p .github

# 2. PR 템플릿 파일 만들기
touch .github/pull_request_template.md
```

만들어진 파일에 아래 내용을 붙여넣습니다:

```markdown
## 변경 내용 요약

<!-- 무엇을 바꿨는지 설명하세요 -->

## AI 코드 리뷰 체크리스트

### 보안
- [ ] 하드코딩된 비밀값(API 키, 비밀번호)이 없다
- [ ] `.env` 파일이 `.gitignore`에 등록되어 있다

### 에러 처리
- [ ] 네트워크 요청, 파일 읽기 등 실패할 수 있는 곳에 `try/except`가 있다
- [ ] `except Exception: pass` 패턴을 사용하지 않았다

### 테스트
- [ ] 새로운 함수에 대한 테스트를 작성했다
- [ ] `pytest`를 실행해 모든 테스트가 통과하는 것을 확인했다

### 코드 품질
- [ ] 함수 하나가 하나의 역할만 한다
- [ ] 변수 이름이 충분히 설명적이다
```

---

### 실습 2 — 새 브랜치를 만들고 PR 올려서 체크리스트 확인하기

실습 1에서 만든 파일을 GitHub에 올려서 실제로 체크리스트가 나타나는지 확인합니다.

```bash
# 1. 새 브랜치 만들기
git checkout -b add-pr-template

# 2. 파일 스테이징
git add .github/pull_request_template.md

# 3. 커밋
git commit -m "chore: AI 코드 리뷰 체크리스트 PR 템플릿 추가"

# 4. 브랜치 푸시
git push origin add-pr-template
```

GitHub 웹사이트에서 이 브랜치로 Pull Request를 열면, 작성해 둔 체크리스트가 본문에 자동으로 나타납니다.

---

### 실습 3 — AI 생성 코드를 체크리스트로 직접 검토하기

아래 AI가 생성했다고 가정한 코드를 보고, 실습 1에서 만든 체크리스트를 기준으로 문제를 찾아봅니다.

```python
# ai_generated_code.py — 문제를 찾아보세요
import requests

DB_PASSWORD = "super_secret_123"   # 힌트 1

def fetch_weather(city):
    url = f"http://api.weather.com/v1/current?city={city}&key=abc999"  # 힌트 2
    result = requests.get(url)
    return result.json()["temperature"]   # 힌트 3
```

이 코드에서 찾아야 할 문제:

1. `DB_PASSWORD`가 코드에 직접 들어 있다 → 환경 변수로 이동
2. API 키 `abc999`가 URL에 하드코딩되어 있다 → 환경 변수로 이동
3. 네트워크 요청 실패 시 예외 처리가 없다 → `try/except` 추가
4. `"temperature"` 키가 없을 때 `KeyError`가 발생한다 → `KeyError` 처리 추가
5. 테스트 코드가 없다 → `tests/test_weather.py` 작성

---

## 자주 하는 실수

| 실수 | 발생하는 에러 메시지 | 해결 방법 |
|------|-------------------|---------| 
| `.github` 폴더 이름을 `github`로 만듦 | PR 본문에 체크리스트가 나타나지 않음 | 폴더 이름을 `.github`(점 포함)로 변경 |
| 템플릿 파일을 `main` 브랜치에 올리지 않음 | PR을 열어도 체크리스트가 안 나옴 | `main` 브랜치에 먼저 머지한 후 새 PR 열기 |
| `except Exception: pass`로 모든 예외를 무시 | 오류가 나도 프로그램이 아무 말 없이 진행됨 | 구체적인 예외 타입을 잡고 `logging.error()`로 기록 |
| `.env` 파일을 `.gitignore`에 추가 안 함 | `git push` 후 GitHub에 비밀 키가 노출 | `.gitignore`에 `.env` 추가 후 `git rm --cached .env` 실행 |
| `response.raise_for_status()` 없이 `response.json()` 호출 | `JSONDecodeError: Expecting value: line 1 column 1 (char 0)` | `response.raise_for_status()`를 먼저 호출해 HTTP 에러를 잡기 |
| 테스트 파일 이름이 `test_`로 시작하지 않음 | `pytest` 실행 시 테스트가 0개로 표시됨 | 파일 이름을 `test_기능이름.py` 형식으로 변경 |

---

## 확인 체크리스트

이 장을 마쳤다면 아래 항목을 스스로 확인해 보세요.

- [ ] `.github/pull_request_template.md` 파일이 무엇인지 설명할 수 있다
- [ ] 이 파일이 `main` 브랜치에 있어야 PR 템플릿이 동작한다는 것을 안다
- [ ] AI 코드에서 하드코딩된 비밀값을 찾아 환경 변수로 바꿀 수 있다
- [ ] `except Exception: pass`가 왜 나쁜지 설명할 수 있다
- [ ] 새 함수에 대한 `pytest` 테스트를 하나 이상 작성할 수 있다
- [ ] 실습 2의 브랜치를 만들고 PR을 열어 체크리스트가 나타나는 것을 확인했다

---

## 한 번 더 생각해 보기

1. AI가 생성한 코드와 사람이 직접 쓴 코드는 리뷰할 때 무엇이 다를까요? AI 코드에서 특히 더 신경 써야 하는 부분이 있다면 무엇인지 생각해 보세요.

2. 팀원이 체크리스트의 모든 항목에 체크를 했는데, 나중에 배포 후 보안 문제가 발생했습니다. 체크리스트만으로는 충분하지 않을 때 어떤 추가적인 방어선을 만들 수 있을까요?

3. 프로젝트가 커지면 체크리스트 항목도 늘어납니다. 체크리스트가 너무 길어지면 사람들이 대충 체크하게 되는 문제가 생깁니다. 이 문제를 어떻게 해결할 수 있을까요?

---

## 다음 장

다음 장에서는 GitHub Actions를 사용해 PR이 열릴 때 자동으로 보안 스캔과 테스트를 실행하는 CI 파이프라인을 만드는 방법을 배웁니다.