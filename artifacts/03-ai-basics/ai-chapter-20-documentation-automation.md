## 이 장에서 배우는 것

- AI에게 README.md 초안을 요청하는 프롬프트를 작성하는 방법
- Python 함수에 docstring을 AI로 자동 추가하는 방법
- GitHub Actions를 이용해 문서를 자동으로 갱신하는 기본 워크플로우
- 좋은 문서화 프롬프트와 나쁜 프롬프트의 차이
- 문서화 자동화 체크리스트로 품질 확인하는 방법

---

## 먼저 쉬운 설명

코드를 잘 짜도 문서가 없으면 다른 사람(또는 한 달 뒤의 나)이 이해하지 못합니다.

그런데 문서 쓰는 것은 귀찮습니다. 코드는 바뀌는데 문서는 업데이트를 잊어버립니다. README가 엉터리인 프로젝트, 함수에 설명이 전혀 없는 코드… 개발자라면 누구나 겪어본 경험입니다.

AI는 이 문제를 많이 해결해 줍니다. 코드를 보여주고 "문서 만들어줘"라고 하면 바로 초안이 나옵니다. 물론 100% 완벽하지는 않지만, 빈 문서에서 시작하는 것보다 훨씬 빠릅니다.

이 장에서는 AI를 문서화 도구로 쓰는 실용적인 방법을 배웁니다.

---

## 1. README.md 자동 생성 — AI에게 어떻게 요청할까

README.md는 프로젝트의 첫 인상입니다. AI에게 요청할 때는 **맥락**을 충분히 줄수록 결과가 좋습니다.

### 나쁜 프롬프트 예시

```
README 만들어줘
```

AI는 어떤 프로젝트인지, 누가 쓰는지, 어떤 기능이 있는지 전혀 모릅니다.

### 좋은 프롬프트 예시

```
아래 정보를 바탕으로 GitHub README.md를 작성해줘.

프로젝트 이름: task-cli
언어: Python 3.11
기능:
  - 할 일(task)을 명령줄에서 추가/삭제/조회
  - JSON 파일로 데이터 저장
  - 완료 표시 기능

대상 사용자: Python 초보자
설치 방법: pip install task-cli
실행 예시: task add "장보기"

포함할 섹션: 프로젝트 설명, 설치, 사용법, 예시 출력, 라이선스
마크다운 형식으로 작성하고, 코드 블록은 bash와 python으로 구분해줘.
```

### AI가 생성한 README.md 예시 (일부)

```markdown
# task-cli

명령줄에서 할 일을 관리하는 간단한 Python CLI 도구입니다.

## 설치

```bash
pip install task-cli
```

## 사용법

```bash
# 할 일 추가
task add "장보기"

# 목록 조회
task list

# 완료 표시
task done 1

# 삭제
task delete 1
```

## 예시 출력

```
[1] [ ] 장보기
[2] [x] 운동하기
```
```

> **팁:** AI가 생성한 README는 반드시 직접 읽고 수정하세요. 없는 기능을 있다고 쓰거나, 실제 명령어와 다르게 쓰는 경우가 있습니다.

---

## 2. docstring 자동 추가 — 함수 설명을 AI가 써준다

Python에서 함수 바로 아래에 `"""..."""`로 쓰는 설명을 **docstring**이라고 합니다. 도구(`help()`, IDE 자동완성)가 이 내용을 읽어서 보여줍니다.

### docstring이 없는 함수

```python
# task_manager.py

def add_task(tasks, title):
    task = {"id": len(tasks) + 1, "title": title, "done": False}
    tasks.append(task)
    return tasks
```

`help(add_task)`를 실행하면 아무 설명도 없습니다.

### AI에게 docstring 추가 요청

```
아래 Python 함수에 Google 스타일 docstring을 추가해줘.
매개변수 타입과 반환값도 설명해줘. 한국어로 작성해줘.

def add_task(tasks, title):
    task = {"id": len(tasks) + 1, "title": title, "done": False}
    tasks.append(task)
    return tasks
```

### AI가 추가한 docstring

```python
def add_task(tasks: list, title: str) -> list:
    """할 일 목록에 새로운 작업을 추가합니다.

    Args:
        tasks (list): 기존 할 일 목록. 각 항목은 dict 형태입니다.
        title (str): 추가할 작업의 제목.

    Returns:
        list: 새 작업이 추가된 할 일 목록.

    Example:
        >>> tasks = []
        >>> add_task(tasks, "장보기")
        [{'id': 1, 'title': '장보기', 'done': False}]
    """
    task = {"id": len(tasks) + 1, "title": title, "done": False}
    tasks.append(task)
    return tasks
```

### 여러 함수를 한 번에 처리하기

파일 전체를 붙여넣고 이렇게 요청하면 됩니다.

```
아래 Python 파일의 모든 함수에 Google 스타일 docstring을 추가해줘.
기존 코드 로직은 절대 바꾸지 말고, docstring만 추가해줘.

[파일 내용 붙여넣기]
```

> **중요:** "기존 코드 로직은 절대 바꾸지 말고"라는 문장을 꼭 넣으세요. 없으면 AI가 코드를 "개선"하려다 버그를 만들 수 있습니다.

---

## 3. API 문서 자동 생성 — pydoc과 AI 프롬프트

### Python 내장 도구: pydoc

docstring이 잘 작성된 파일은 `pydoc` 명령으로 바로 HTML 문서를 만들 수 있습니다.

```bash
# HTML 파일로 저장
python -m pydoc -w task_manager

# 브라우저에서 확인 (포트 8080)
python -m pydoc -p 8080
```

실행하면 `task_manager.html` 파일이 생깁니다.

### AI로 API 명세서 만들기

REST API가 있을 때 AI에게 이렇게 요청합니다.

```
아래 Flask 라우터 코드를 보고 API 명세서를 마크다운 표 형식으로 만들어줘.
각 엔드포인트의 메서드, URL, 설명, 요청 예시, 응답 예시를 포함해줘.

@app.route('/tasks', methods=['GET'])
def get_tasks():
    return jsonify(tasks)

@app.route('/tasks', methods=['POST'])
def create_task():
    data = request.json
    new_task = add_task(tasks, data['title'])
    return jsonify(new_task), 201

@app.route('/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    ...
```

### AI가 생성한 API 명세서 예시

```markdown
## API 엔드포인트

| 메서드 | URL | 설명 |
|--------|-----|------|
| GET | /tasks | 모든 할 일 목록 조회 |
| POST | /tasks | 새 할 일 생성 |
| DELETE | /tasks/{id} | 특정 할 일 삭제 |

### GET /tasks

**응답 예시**
```json
[
  {"id": 1, "title": "장보기", "done": false},
  {"id": 2, "title": "운동하기", "done": true}
]
```
```

---

## 4. GitHub Actions로 문서 자동 갱신

코드가 바뀔 때마다 수동으로 문서를 업데이트하는 것은 금방 잊어버립니다. GitHub Actions를 쓰면 코드를 push할 때 자동으로 문서를 생성할 수 있습니다.

### 기본 워크플로우 구조

```
내 컴퓨터에서 코드 push
    ↓
GitHub Actions 실행
    ↓
pydoc으로 HTML 문서 생성
    ↓
docs/ 폴더에 자동 커밋
```

### 워크플로우 파일 예시

`.github/workflows/docs.yml` 파일을 만듭니다.

```yaml
name: 문서 자동 생성

on:
  push:
    branches:
      - main
    paths:
      - '**.py'          # Python 파일이 바뀔 때만 실행

jobs:
  generate-docs:
    runs-on: ubuntu-latest

    steps:
      - name: 코드 가져오기
        uses: actions/checkout@v4

      - name: Python 설정
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: 의존성 설치
        run: pip install -r requirements.txt

      - name: pydoc HTML 문서 생성
        run: |
          mkdir -p docs
          python -m pydoc -w task_manager
          mv task_manager.html docs/

      - name: 문서 커밋 및 push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add docs/
          git diff --staged --quiet || git commit -m "docs: 자동 문서 갱신"
          git push
```

> **설명:**
> - `paths: - '**.py'` — Python 파일이 바뀔 때만 워크플로우가 실행됩니다. 불필요한 실행을 줄입니다.
> - `git diff --staged --quiet ||` — 변경 사항이 없을 때 빈 커밋을 만들지 않습니다.

### AI에게 워크플로우 만들어 달라고 요청하기

```
GitHub Actions 워크플로우 파일을 만들어줘.

조건:
- main 브랜치에 .py 파일이 push될 때 실행
- pydoc으로 HTML 문서를 생성해서 docs/ 폴더에 저장
- 변경된 문서를 자동으로 커밋하고 push
- Python 3.11 사용

YAML 형식으로 작성해줘.
```

---

## 따라 하기 실습

### 실습 1 — 내 함수에 docstring 추가하기

아래 파일을 `task_manager.py`로 저장합니다.

```python
# task_manager.py

def load_tasks(filepath):
    import json
    try:
        with open(filepath, 'r', encoding='utf-8') as f:
            return json.load(f)
    except FileNotFoundError:
        return []

def save_tasks(tasks, filepath):
    import json
    with open(filepath, 'w', encoding='utf-8') as f:
        json.dump(tasks, f, ensure_ascii=False, indent=2)

def complete_task(tasks, task_id):
    for task in tasks:
        if task['id'] == task_id:
            task['done'] = True
            return True
    return False
```

Claude 또는 ChatGPT에 이 파일을 붙여넣고 다음 프롬프트를 사용합니다.

```
아래 Python 파일의 모든 함수에 Google 스타일 docstring을 추가해줘.
한국어로 작성하고, 기존 코드 로직은 절대 바꾸지 마.
타입 힌트도 함수 시그니처에 추가해줘.

[파일 내용 붙여넣기]
```

결과를 파일에 붙여넣은 뒤 확인합니다.

```bash
python -c "import task_manager; help(task_manager.load_tasks)"
```

정상이라면 docstring 내용이 출력됩니다.

---

### 실습 2 — README.md 초안 만들기

실습 1에서 만든 `task_manager.py`를 기반으로 README를 만들어 봅니다.

다음 프롬프트를 사용합니다.

```
아래 Python 모듈을 기반으로 GitHub README.md를 작성해줘.

모듈 이름: task_manager
기능: JSON 파일 기반 할 일(task) 관리
주요 함수: load_tasks, save_tasks, complete_task
사용 환경: Python 3.11, 별도 패키지 불필요

포함할 내용:
1. 프로젝트 소개 (2-3줄)
2. 설치 방법
3. 사용 예시 (코드 블록 포함)
4. 함수 목록 표 (함수명, 매개변수, 설명)
5. 라이선스: MIT

한국어로 작성해줘.
```

생성된 내용을 `README.md` 파일로 저장하고, 실제 코드와 맞지 않는 부분을 직접 수정합니다.

---

### 실습 3 — GitHub Actions 워크플로우 설정

1. 프로젝트 루트에 `.github/workflows/` 폴더를 만듭니다.

```bash
mkdir -p .github/workflows
```

2. AI에게 요청해서 받은 워크플로우 내용을 `.github/workflows/docs.yml`로 저장합니다.

3. 파일을 커밋하고 push합니다.

```bash
git add .github/workflows/docs.yml
git commit -m "ci: 문서 자동 생성 워크플로우 추가"
git push origin main
```

4. GitHub 저장소의 **Actions** 탭에서 워크플로우가 실행되는지 확인합니다.

실패했다면 로그에서 오류 메시지를 복사해서 AI에게 붙여넣고 물어봅니다.

```
GitHub Actions에서 아래 오류가 났어. 원인과 해결 방법을 알려줘.

[오류 메시지 붙여넣기]
```

---

## 자주 하는 실수

| 실수 | 오류 또는 증상 | 해결 방법 |
|------|---------------|-----------|
| AI가 없는 기능을 README에 썼는데 그냥 사용 | 사용자가 존재하지 않는 명령어 실행 | AI 결과물은 반드시 실제 코드와 대조해서 검토 |
| docstring 요청 시 "코드 바꾸지 마" 생략 | 함수 로직이 바뀌거나 변수명이 변경됨 | 프롬프트에 "기존 코드 로직은 절대 바꾸지 마" 명시 |
| `python -m pydoc -w task_manager` 실행 위치 오류 | `No module named task_manager` | `task_manager.py` 파일이 있는 폴더에서 실행 |
| GitHub Actions에서 push 권한 없음 | `remote: Permission to ... denied` | 저장소 Settings → Actions → General에서 "Read and write permissions" 활성화 |
| 빈 커밋이 계속 생성됨 | `nothing to commit` 오류 없이 빈 커밋 반복 | `git diff --staged --quiet \|\|` 조건을 워크플로우에 추가 |
| pydoc HTML에 한글이 깨짐 | 브라우저에서 문자가 ???로 표시 | `# -*- coding: utf-8 -*-` 또는 파일 인코딩을 UTF-8로 저장 |

---

## 확인 체크리스트

- [ ] AI에게 README를 요청할 때 프로젝트 이름, 기능, 대상 사용자를 프롬프트에 포함했다
- [ ] AI가 생성한 README를 읽고 실제 코드와 맞지 않는 부분을 수정했다
- [ ] `task_manager.py`의 모든 함수에 Google 스타일 docstring이 추가되었다
- [ ] `help(함수명)` 또는 `python -m pydoc 모듈명`으로 docstring이 보이는 것을 확인했다
- [ ] `.github/workflows/docs.yml` 파일이 올바른 경로에 저장되어 있다
- [ ] GitHub Actions 탭에서 워크플로우가 성공적으로 실행된 것을 확인했다
- [ ] AI 결과물을 그냥 쓰지 않고 한 번 이상 직접 읽어 보았다

---

## 한 번 더 생각해 보기

1. AI가 만든 README.md에 실제로 없는 기능이 포함되어 있었던 경험을 해보셨나요? 왜 이런 일이 생기는지, 그리고 어떻게 방지할 수 있을지 생각해 보세요.

2. docstring이 없는 함수와 있는 함수의 차이를 `help()` 명령으로 직접 비교해 보면 어떤 느낌인가요? 실제로 팀 프로젝트에서 docstring이 있으면 어떤 점이 편해질까요?

3. GitHub Actions 워크플로우가 실패했을 때 오류 메시지를 AI에게 그대로 붙여넣어서 해결하는 방식에 대해 어떻게 생각하나요? 이 방법의 한계는 무엇일까요?

---

## 다음 장

다음 장에서는 AI를 활용해 코드 리뷰를 자동화하고, PR에 자동으로 피드백을 달아주는 GitHub Actions 워크플로우를 만드는 방법을 배웁니다.