## 이 장에서 배우는 것

- GitHub Copilot이 무엇인지, 왜 팀에서 쓰면 좋은지 이해한다
- VS Code에서 Copilot 확장을 설치하고 로그인하는 방법을 익힌다
- 팀 전체가 같은 설정을 공유하는 `.vscode/extensions.json` 파일을 만든다
- `.github/copilot-instructions.md`로 Copilot에게 우리 팀의 코딩 규칙을 알려주는 방법을 배운다
- Copilot 제안을 수락·거절·수정하는 기본 조작을 익힌다

---

## 먼저 쉬운 설명

코드를 작성하다 보면 "이 다음에 뭘 써야 하지?" 하고 멈추는 순간이 많습니다. GitHub Copilot은 그 순간마다 옆에서 코드를 제안해 주는 **AI 짝 프로그래머**입니다.

혼자 쓸 때도 편리하지만, 팀에서 쓰면 더 강력합니다. 설정 파일 하나를 저장소에 넣어두면 팀원 모두가 동일한 Copilot 환경을 갖게 됩니다. 신입 팀원이 저장소를 클론하는 순간 "이 확장 설치할게요?" 팝업이 뜨고, Copilot은 우리 팀의 컨벤션을 이미 알고 제안을 해 줍니다.

이 장에서는 그 팀 설정을 처음부터 직접 만들어 봅니다.

---

## 1. GitHub Copilot 확장 설치하기

### VS Code에서 설치하는 방법

VS Code 왼쪽 사이드바의 **Extensions** 아이콘(네모 네 개)을 클릭하고 `GitHub Copilot`을 검색합니다.

설치해야 할 확장은 두 가지입니다.

| 확장 이름 | 역할 |
|---|---|
| `GitHub Copilot` | 코드 자동완성 |
| `GitHub Copilot Chat` | 채팅으로 질문하기 |

설치 후 VS Code 우측 하단에 Copilot 아이콘이 나타납니다. 아이콘을 클릭하면 GitHub 계정으로 로그인하라는 창이 열립니다.

> **Copilot 유료 여부**: 개인 계정은 월 구독이 필요합니다. 학생이라면 GitHub Education Pack을 통해 무료로 사용할 수 있습니다. 회사 계정이라면 관리자가 GitHub Copilot Business를 활성화해 줘야 합니다.

---

## 2. 팀 공유 확장 목록 만들기

### `.vscode/extensions.json`

이 파일을 저장소에 넣으면, 팀원이 저장소를 클론했을 때 VS Code가 자동으로 "이 확장들을 설치할까요?" 팝업을 띄워줍니다.

```
your-project/
└── .vscode/
    └── extensions.json   ← 여기에 만든다
```

```json
{
  "recommendations": [
    "github.copilot",
    "github.copilot-chat",
    "ms-python.python",
    "ms-python.ruff",
    "eamodio.gitlens"
  ]
}
```

각 값은 VS Code 마켓플레이스의 **확장 ID**입니다. 확장 상세 페이지 오른쪽에서 확인할 수 있습니다.

### 확인 방법

파일을 저장한 뒤 VS Code 명령 팔레트(`Cmd+Shift+P` / `Ctrl+Shift+P`)에서 `Show Recommended Extensions`를 실행하면 목록이 보입니다.

---

## 3. Copilot에게 팀 규칙 알려주기

### `.github/copilot-instructions.md`

Copilot이 코드를 제안할 때 우리 팀의 컨벤션을 반영하도록 지시할 수 있습니다. 이 파일을 저장소 루트의 `.github/` 폴더 안에 만듭니다.

```
your-project/
├── .github/
│   └── copilot-instructions.md   ← 여기에 만든다
└── .vscode/
    └── extensions.json
```

아래는 Python 팀을 위한 예시입니다.

```markdown
# 팀 코딩 규칙 (Copilot 지침)

## 언어 및 프레임워크
- Python 3.11 이상을 사용한다
- 웹 API는 FastAPI로 작성한다
- 데이터 검증은 Pydantic v2를 사용한다

## 코드 스타일
- 함수와 변수 이름은 snake_case를 사용한다
- 클래스 이름은 PascalCase를 사용한다
- 타입 힌트를 반드시 작성한다
- docstring은 한국어로 작성한다

## 금지 사항
- print() 디버깅 대신 logging 모듈을 사용한다
- bare except는 사용하지 않는다

## 예시 패턴
함수를 만들 때 항상 아래 형태를 따른다:

```python
def 함수이름(인자: 타입) -> 반환타입:
    """한 줄 설명."""
    ...
```
```

### 실제 효과 확인

`copilot-instructions.md` 설정 전후를 비교해 봅시다.

**설정 전** — Copilot이 제안하는 함수:

```python
# copilot-instructions.md 없을 때
def get_user(id):
    user = db.query("SELECT * FROM users WHERE id = " + str(id))
    print("찾은 유저:", user)
    return user
```

**설정 후** — 우리 팀 규칙이 반영된 제안:

```python
# copilot-instructions.md 적용 후
def get_user(user_id: int) -> User | None:
    """ID로 사용자를 조회한다."""
    return db.query(User).filter(User.id == user_id).first()
```

타입 힌트, snake_case, 한국어 docstring이 자동으로 붙어 나옵니다.

---

## 4. Copilot 제안 수락·거절·수정하기

Copilot이 코드를 제안하면 회색 글씨로 미리 보기가 나타납니다. 이때 사용할 수 있는 단축키입니다.

| 동작 | Mac | Windows/Linux |
|---|---|---|
| 제안 수락 | `Tab` | `Tab` |
| 제안 거절 | `Esc` | `Esc` |
| 다음 제안 보기 | `Option+]` | `Alt+]` |
| 이전 제안 보기 | `Option+[` | `Alt+[` |
| 여러 제안 한번에 보기 | `Ctrl+Enter` | `Ctrl+Enter` |

### 부분 수락

제안 전체가 마음에 들지 않을 때는 **단어 단위**로 수락할 수 있습니다.

- Mac: `Cmd+→`
- Windows: `Ctrl+→`

예를 들어 Copilot이 `get_user_by_email_and_status`를 제안했는데 `get_user_by_email`까지만 필요하다면, `Cmd+→`를 두 번 눌러 단어 두 개만 수락합니다.

---

## 5. VS Code 워크스페이스 설정 공유하기

`.vscode/settings.json`에 팀 공통 설정을 저장하면, 저장소를 클론한 모든 팀원이 같은 편집기 동작을 갖게 됩니다.

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "charliermarsh.ruff",
  "python.defaultInterpreterPath": ".venv/bin/python",
  "github.copilot.enable": {
    "*": true,
    "markdown": true,
    "plaintext": false
  },
  "github.copilot.editor.enableAutoCompletions": true
}
```

`github.copilot.enable` 항목의 의미:

- `"*": true` — 모든 파일 형식에서 Copilot 활성화
- `"markdown": true` — 마크다운 파일에서도 활성화
- `"plaintext": false` — 일반 텍스트 파일에서는 비활성화

> **주의**: `.vscode/settings.json`에는 팀 공통 설정만 넣습니다. API 키나 개인 경로처럼 사람마다 다른 값은 넣지 않습니다. 개인 설정은 VS Code의 **User Settings**에서 관리합니다.

---

## 따라 하기 실습

### 실습 1 — 팀 공유 확장 목록 파일 만들기

**목표**: 저장소에 `.vscode/extensions.json`을 추가하고 GitHub에 푸시한다.

```bash
# 1. 폴더 만들기
mkdir -p .vscode

# 2. 파일 만들기 (VS Code에서 직접 열어도 됩니다)
code .vscode/extensions.json
```

아래 내용을 붙여넣고 저장합니다.

```json
{
  "recommendations": [
    "github.copilot",
    "github.copilot-chat",
    "ms-python.python",
    "ms-python.ruff"
  ]
}
```

```bash
# 3. 커밋하고 푸시
git add .vscode/extensions.json
git commit -m "chore: add recommended VS Code extensions for team"
git push origin main
```

**확인**: GitHub 저장소 페이지에서 `.vscode/extensions.json` 파일이 보이면 성공입니다.

---

### 실습 2 — Copilot 지침 파일 작성하기

**목표**: `.github/copilot-instructions.md`를 만들어 팀 규칙 세 가지를 적는다.

```bash
# 1. .github 폴더 만들기 (이미 있으면 건너뜁니다)
mkdir -p .github

# 2. 파일 만들기
code .github/copilot-instructions.md
```

아래 내용을 팀 상황에 맞게 수정해서 저장합니다.

```markdown
# 팀 코딩 지침

## 언어
- Python 3.11 이상

## 스타일
- 함수 이름은 snake_case
- 타입 힌트 필수
- docstring은 한국어로 작성

## 금지
- print() 사용 금지, logging 사용
```

```bash
# 3. 커밋하고 푸시
git add .github/copilot-instructions.md
git commit -m "docs: add Copilot instructions for team conventions"
git push origin main
```

---

### 실습 3 — Copilot 제안을 받아 함수 완성하기

**목표**: Copilot의 도움으로 함수 하나를 완성하고, 수락·거절을 직접 경험한다.

`main.py` 파일을 열고 아래 주석만 입력합니다. (직접 타이핑해야 Copilot이 동작합니다.)

```python
# 사용자 이름과 나이를 받아서 인사말을 반환하는 함수
def greet_user(
```

여기서 멈추면 Copilot이 나머지 코드를 회색 글씨로 제안합니다.

1. 제안이 마음에 들면 `Tab`을 눌러 수락합니다.
2. 다른 제안을 보려면 `Option+]` (`Alt+]`)를 누릅니다.
3. `Ctrl+Enter`를 누르면 여러 제안을 한 화면에서 비교할 수 있습니다.

가장 마음에 드는 제안을 수락한 뒤 저장합니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 또는 증상 | 해결 방법 |
|---|---|---|
| Copilot 아이콘에 빨간 점이 표시됨 | `GitHub Copilot could not connect` | GitHub에 로그인됐는지 확인. VS Code에서 `Accounts` 아이콘 클릭 후 로그인 상태 확인 |
| Copilot 제안이 전혀 안 나옴 | 회색 텍스트 없음 | `settings.json`의 `github.copilot.enable`이 `false`인지 확인. 또는 파일 형식이 `plaintext`로 인식된 경우 |
| `extensions.json` 파일을 만들었는데 팝업이 안 뜸 | 아무 반응 없음 | 저장소를 한 번 닫고 다시 열면 팝업이 나타납니다 (`File > Close Folder` 후 재오픈) |
| `copilot-instructions.md`가 반영 안 됨 | Copilot이 여전히 예전 스타일로 제안 | 파일 경로가 `.github/copilot-instructions.md`인지 확인 (`.github` 아래에 있어야 함) |
| 제안이 너무 길어서 부분만 쓰고 싶음 | — | `Tab` 대신 `Cmd+→` (Mac) / `Ctrl+→` (Windows)로 단어 단위 수락 |
| `.vscode/settings.json`에 개인 API 키를 넣고 푸시함 | 팀원에게 내 키가 노출됨 | 즉시 키를 폐기하고 재발급. 개인 정보는 절대 이 파일에 넣지 않는다 |

---

## 확인 체크리스트

- [ ] VS Code에 `GitHub Copilot`과 `GitHub Copilot Chat` 확장이 설치되어 있다
- [ ] VS Code 우측 하단 Copilot 아이콘에 빨간 점이 없다 (정상 연결 상태)
- [ ] 저장소에 `.vscode/extensions.json` 파일이 있고 `github.copilot`이 포함되어 있다
- [ ] 저장소에 `.github/copilot-instructions.md` 파일이 있고 팀 규칙 세 가지 이상이 적혀 있다
- [ ] `Tab`으로 제안 수락, `Esc`로 거절을 직접 해 봤다
- [ ] `Ctrl+Enter`로 여러 제안을 한 화면에서 비교해 봤다
- [ ] `.vscode/settings.json`에 개인 비밀 정보가 포함되지 않았다
- [ ] 두 파일 모두 `git push`로 팀 저장소에 올라가 있다

---

## 한 번 더 생각해 보기

1. `copilot-instructions.md`에 "절대로 SQL을 직접 작성하지 말고 ORM을 사용하라"고 적으면 Copilot이 항상 그 규칙을 따를까요? Copilot의 지침 파일은 **강제** 규칙인가요, **권고** 사항인가요? 차이가 무엇인지 생각해 보세요.

2. `.vscode/settings.json`을 저장소에 커밋하면 편리하지만, 팀원마다 운영체제나 Python 경로가 다를 수 있습니다. 이런 경우 어떤 설정은 커밋하고 어떤 설정은 `.gitignore`에 넣는 것이 좋을지 기준을 세워 보세요.

3. Copilot이 제안한 코드를 `Tab`으로 바로 수락하면 편리합니다. 하지만 수락 전에 어떤 것을 확인해야 할까요? 제안 코드를 그대로 믿으면 안 되는 상황을 하나 상상해 보세요.

---

## 다음 장

다음 장에서는 GitHub Actions를 사용해 Pull Request가 열릴 때마다 Copilot이 자동으로 코드 리뷰를 남기는 CI 파이프라인을 구성합니다.