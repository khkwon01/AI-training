## 이 장에서 배우는 것

- AI 컨텍스트 윈도우가 무엇인지, 왜 대형 코드베이스에서 중요한지 이해한다
- 컨텍스트 한계에 부딪혔을 때 나타나는 증상을 알아본다
- 대형 코드베이스에서 AI에게 효과적으로 정보를 전달하는 전략을 익힌다
- 파일 요약, 청킹, 핵심 정보 추출 기법을 실습한다
- AI와 협업할 때 컨텍스트를 절약하는 습관을 만든다

---

## 먼저 쉬운 설명

AI와 대화할 때 AI는 **한 번에 볼 수 있는 글자 수에 한계**가 있습니다. 이것을 "컨텍스트 윈도우"라고 부릅니다.

비유로 생각해 보면 이렇습니다. 여러분이 누군가에게 길을 물어볼 때, 그 사람이 한 번에 기억할 수 있는 정보는 제한적입니다. 서울 전체 지도를 통째로 보여주면서 "여기서 거기 어떻게 가요?"라고 묻는 것보다, 현재 위치와 목적지만 간단히 알려주는 게 훨씬 빠르고 정확한 답을 얻을 수 있죠.

실제 프로젝트에서는 파일이 수십, 수백 개에 달합니다. 이 모든 코드를 AI에게 한꺼번에 보여주면 AI가 "기억이 꽉 찼어요"라는 상황이 됩니다. 그러면 앞에서 나눈 대화 내용을 잊어버리거나, 엉뚱한 답을 내놓기 시작합니다.

이 장에서는 **큰 코드베이스에서도 AI와 스마트하게 협업하는 방법**을 배웁니다.

---

## 1. 컨텍스트 윈도우란 무엇인가

AI 모델은 대화를 "토큰"이라는 단위로 처리합니다. 영어 단어 하나, 한국어 글자 1~2개 정도가 토큰 하나에 해당합니다. Claude Sonnet 기준으로 약 20만 토큰(약 15만 단어)을 한 번에 처리할 수 있습니다.

문제는 대화가 길어질수록, 또는 파일을 많이 붙여 넣을수록 이 공간이 빠르게 채워진다는 점입니다.

```python
# 예시: 프로젝트 파일 크기 확인하기
# estimate_tokens.py

import os

def count_chars_in_project(root_dir: str) -> dict:
    """프로젝트 폴더의 파일별 글자 수를 반환한다."""
    result = {}
    for dirpath, _, filenames in os.walk(root_dir):
        for filename in filenames:
            if filename.endswith(".py"):
                filepath = os.path.join(dirpath, filename)
                with open(filepath, "r", encoding="utf-8") as f:
                    content = f.read()
                # 대략적인 토큰 수: 글자 수 / 2 (한국어 기준)
                estimated_tokens = len(content) // 2
                result[filepath] = estimated_tokens
    return result


if __name__ == "__main__":
    project_path = "."  # 현재 폴더 기준
    sizes = count_chars_in_project(project_path)
    total = sum(sizes.values())

    for path, tokens in sorted(sizes.items(), key=lambda x: -x[1]):
        print(f"{tokens:>8,} 토큰  {path}")

    print(f"\n합계: {total:,} 토큰")
    print(f"Claude 컨텍스트의 약 {total / 200000 * 100:.1f}% 사용")
```

실행하면 이렇게 출력됩니다:

```
   5,200 토큰  ./src/api/user_service.py
   3,800 토큰  ./src/db/models.py
   2,100 토큰  ./tests/test_user.py

합계: 11,100 토큰
Claude 컨텍스트의 약 5.6% 사용
```

> **핵심 개념**: 컨텍스트가 꽉 차면 AI는 오래된 대화를 잊거나, 답변 품질이 떨어집니다. 미리 파악하고 관리하는 것이 중요합니다.

---

## 2. 컨텍스트 한계에 부딪혔을 때 나타나는 증상

컨텍스트가 가득 찼을 때 AI의 답변에는 패턴이 있습니다. 이 증상을 알아두면 "아, 지금 컨텍스트가 문제구나"라고 빠르게 판단할 수 있습니다.

```python
# 증상을 유발하는 상황 예시
# bad_context_usage.py

# ❌ 나쁜 방법: 관련 없는 파일까지 모두 붙여 넣기
with open("settings.py") as f:
    settings_code = f.read()

with open("utils.py") as f:
    utils_code = f.read()

with open("models.py") as f:
    models_code = f.read()

with open("views.py") as f:
    views_code = f.read()

# AI에게 보내는 메시지
message = f"""
다음은 전체 프로젝트 코드입니다:

{settings_code}
{utils_code}
{models_code}
{views_code}

views.py의 get_user 함수에서 버그를 찾아주세요.
"""
# 결과: AI가 엉뚱한 파일을 참조하거나, 처음 대화 내용을 잊어버림
```

**컨텍스트 한계 증상 목록:**

| 증상 | 실제 AI 답변 예시 |
|------|-----------------|
| 앞서 정의한 함수 이름을 틀림 | "앞서 말씀하신 `process_data` 함수에서..." (실제 함수명은 `handle_data`) |
| 같은 질문을 반복해서 물어봄 | "혹시 사용하시는 Python 버전이 어떻게 되시나요?" (세 번째 답변에서) |
| 이전 코드 수정 사항을 반영 못 함 | 이미 고쳤다고 말한 버그를 다시 고치라고 함 |
| 관계없는 파일 내용을 언급 | `get_user` 버그를 물었는데 `settings.py` 내용을 분석 |

---

## 3. 핵심 정보만 추출하는 청킹 전략

"청킹(Chunking)"은 큰 정보를 작고 의미 있는 조각으로 나누는 기술입니다. AI에게 전체 코드를 넘기는 대신, 지금 작업에 필요한 부분만 정밀하게 추출해서 전달합니다.

```python
# smart_context.py
# AI에게 보낼 핵심 정보를 추출하는 유틸리티

import ast
import textwrap
from pathlib import Path


def extract_function_signatures(filepath: str) -> str:
    """파이썬 파일에서 함수 시그니처와 docstring만 추출한다."""
    source = Path(filepath).read_text(encoding="utf-8")
    tree = ast.parse(source)
    
    summaries = []
    for node in ast.walk(tree):
        if isinstance(node, (ast.FunctionDef, ast.AsyncFunctionDef)):
            # 함수 시그니처 재구성
            args = [arg.arg for arg in node.args.args]
            signature = f"def {node.name}({', '.join(args)}):"
            
            # docstring이 있으면 포함
            docstring = ast.get_docstring(node)
            if docstring:
                short_doc = docstring.split("\n")[0]  # 첫 줄만
                summaries.append(f"  {signature}\n      # {short_doc}")
            else:
                summaries.append(f"  {signature}")
    
    return "\n".join(summaries)


def extract_class_structure(filepath: str) -> str:
    """클래스 이름과 메서드 목록만 추출한다."""
    source = Path(filepath).read_text(encoding="utf-8")
    tree = ast.parse(source)
    
    output = []
    for node in ast.walk(tree):
        if isinstance(node, ast.ClassDef):
            output.append(f"class {node.name}:")
            for item in node.body:
                if isinstance(item, (ast.FunctionDef, ast.AsyncFunctionDef)):
                    args = [a.arg for a in item.args.args if a.arg != "self"]
                    output.append(f"    def {item.name}({', '.join(args)})")
    
    return "\n".join(output)


# 사용 예시
if __name__ == "__main__":
    # 전체 코드 대신 구조만 AI에게 전달
    signatures = extract_function_signatures("src/user_service.py")
    print("=== AI에게 전달할 컨텍스트 ===")
    print(signatures)
```

출력 예시:

```
=== AI에게 전달할 컨텍스트 ===
  def create_user(username, email, password):
      # 새 사용자를 생성하고 DB에 저장한다
  def get_user_by_id(user_id):
      # ID로 사용자를 조회한다
  def update_user_email(user_id, new_email):
      # 사용자 이메일을 업데이트한다
```

전체 파일(300줄) 대신 **핵심 구조 6줄**만 AI에게 전달했습니다. 토큰 사용량이 약 95% 줄어듭니다.

---

## 4. 프롬프트에 컨텍스트 지도 만들기

대형 프로젝트에서는 AI에게 "현재 상황 지도"를 먼저 알려주면 훨씬 정확한 답을 얻을 수 있습니다. 매번 파일을 붙여 넣는 것보다 **프로젝트 구조를 한 번에 요약**해 두는 것이 효율적입니다.

```python
# generate_context_map.py
# 프로젝트 컨텍스트 지도를 자동 생성한다

import os
from pathlib import Path


def generate_project_map(root: str, max_depth: int = 3) -> str:
    """프로젝트 폴더 구조를 텍스트로 요약한다."""
    lines = [f"프로젝트 루트: {root}\n"]
    
    ignore_dirs = {".git", "__pycache__", ".venv", "node_modules", ".pytest_cache"}
    
    for dirpath, dirnames, filenames in os.walk(root):
        # 무시할 폴더 제외
        dirnames[:] = [d for d in dirnames if d not in ignore_dirs]
        
        depth = dirpath.replace(root, "").count(os.sep)
        if depth > max_depth:
            continue
        
        indent = "  " * depth
        folder_name = os.path.basename(dirpath) or root
        lines.append(f"{indent}📁 {folder_name}/")
        
        for filename in sorted(filenames):
            if filename.endswith((".py", ".md", ".yaml", ".toml")):
                sub_indent = "  " * (depth + 1)
                lines.append(f"{sub_indent}📄 {filename}")
    
    return "\n".join(lines)


def create_ai_context_prompt(task: str, relevant_files: list[str]) -> str:
    """AI에게 보낼 최적화된 컨텍스트 프롬프트를 만든다."""
    project_map = generate_project_map(".")
    
    file_contents = []
    for filepath in relevant_files:
        content = Path(filepath).read_text(encoding="utf-8")
        # 파일이 200줄 이상이면 앞 50줄만 포함
        lines = content.split("\n")
        if len(lines) > 200:
            preview = "\n".join(lines[:50])
            file_contents.append(
                f"### {filepath} (처음 50줄, 전체 {len(lines)}줄)\n```python\n{preview}\n```"
            )
        else:
            file_contents.append(f"### {filepath}\n```python\n{content}\n```")
    
    prompt = f"""## 프로젝트 구조
{project_map}

## 관련 파일
{''.join(file_contents)}

## 작업 요청
{task}
"""
    return prompt


# 사용 예시
if __name__ == "__main__":
    prompt = create_ai_context_prompt(
        task="user_service.py의 create_user 함수에서 이메일 중복 체크가 빠진 버그를 수정해 주세요.",
        relevant_files=["src/user_service.py", "src/db/models.py"]
    )
    
    token_estimate = len(prompt) // 2
    print(f"생성된 프롬프트 크기: 약 {token_estimate:,} 토큰")
    print(prompt[:500] + "...")  # 미리보기
```

---

## 5. 대화 세션 리셋과 요약 전략

긴 대화가 이어질수록 AI가 초반 내용을 잊어버립니다. 이를 방지하는 가장 좋은 방법은 **주기적으로 대화를 요약해서 새 세션에 전달**하는 것입니다.

```python
# session_summary.py
# 현재 작업 상태를 요약해서 새 세션에 전달할 수 있는 형태로 저장한다

from dataclasses import dataclass, field
from datetime import datetime
import json
from pathlib import Path


@dataclass
class WorkSession:
    """AI와의 작업 세션 상태를 추적한다."""
    project_name: str
    goal: str
    completed_tasks: list[str] = field(default_factory=list)
    current_task: str = ""
    known_issues: list[str] = field(default_factory=list)
    key_decisions: list[str] = field(default_factory=list)
    modified_files: list[str] = field(default_factory=list)

    def add_completed(self, task: str) -> None:
        self.completed_tasks.append(task)

    def to_context_prompt(self) -> str:
        """새 AI 세션을 시작할 때 붙여 넣을 요약 프롬프트를 생성한다."""
        completed = "\n".join(f"  - {t}" for t in self.completed_tasks)
        issues = "\n".join(f"  - {i}" for i in self.known_issues)
        decisions = "\n".join(f"  - {d}" for d in self.key_decisions)
        files = "\n".join(f"  - {f}" for f in self.modified_files)

        return f"""## 이전 세션 요약 ({datetime.now().strftime('%Y-%m-%d')})

**프로젝트**: {self.project_name}
**전체 목표**: {self.goal}
**현재 작업**: {self.current_task}

**완료된 작업**:
{completed}

**알려진 이슈**:
{issues}

**주요 결정 사항**:
{decisions}

**수정된 파일**:
{files}

위 내용을 바탕으로 현재 작업을 계속 도와주세요.
"""

    def save(self, filepath: str = "ai_session.json") -> None:
        Path(filepath).write_text(
            json.dumps(self.__dict__, ensure_ascii=False, indent=2),
            encoding="utf-8"
        )

    @classmethod
    def load(cls, filepath: str = "ai_session.json") -> "WorkSession":
        data = json.loads(Path(filepath).read_text(encoding="utf-8"))
        return cls(**data)


# 사용 예시
if __name__ == "__main__":
    session = WorkSession(
        project_name="쇼핑몰 백엔드 API",
        goal="사용자 인증 시스템 구현",
        current_task="JWT 토큰 갱신 로직 구현",
    )
    session.add_completed("회원가입 API 완성")
    session.add_completed("로그인 API 완성")
    session.known_issues.append("refresh_token 만료 시 500 에러 발생")
    session.key_decisions.append("토큰 저장소로 Redis 사용하기로 결정")
    session.modified_files.extend(["src/auth/jwt_handler.py", "src/api/auth_routes.py"])

    # 저장
    session.save()

    # 새 세션 시작 시 로드하여 프롬프트 생성
    loaded = WorkSession.load()
    print(loaded.to_context_prompt())
```

출력 예시:

```
## 이전 세션 요약 (2026-05-18)

**프로젝트**: 쇼핑몰 백엔드 API
**전체 목표**: 사용자 인증 시스템 구현
**현재 작업**: JWT 토큰 갱신 로직 구현

**완료된 작업**:
  - 회원가입 API 완성
  - 로그인 API 완성

**알려진 이슈**:
  - refresh_token 만료 시 500 에러 발생

**주요 결정 사항**:
  - 토큰 저장소로 Redis 사용하기로 결정

**수정된 파일**:
  - src/auth/jwt_handler.py
  - src/api/auth_routes.py
```

---

## 따라 하기 실습

### 실습 1: 내 프로젝트 컨텍스트 비용 측정하기

**목표**: 현재 프로젝트가 AI 컨텍스트를 얼마나 차지하는지 파악한다.

1. `estimate_tokens.py` 파일을 프로젝트 루트에 만든다.
2. 아래 코드를 붙여 넣고 실행한다.

```python
# estimate_tokens.py
import os

def scan_project(root="."):
    total = 0
    files = []
    for dirpath, dirnames, filenames in os.walk(root):
        dirnames[:] = [d for d in dirnames
                       if d not in {".git", "__pycache__", ".venv", "node_modules"}]
        for name in filenames:
            if name.endswith((".py", ".js", ".ts", ".md")):
                path = os.path.join(dirpath, name)
                try:
                    with open(path, encoding="utf-8") as f:
                        size = len(f.read()) // 2  # 대략적인 토큰 추정
                    files.append((size, path))
                    total += size
                except Exception:
                    pass
    return total, sorted(files, reverse=True)

total, files = scan_project()
print(f"전체 토큰 추정: {total:,}")
print(f"컨텍스트 점유율: {total / 200_000 * 100:.1f}%\n")
print("상위 5개 파일:")
for size, path in files[:5]:
    print(f"  {size:>6,}토큰  {path}")
```

3. 출력 결과를 보고 "이 파일들이 정말 AI 작업에 모두 필요한가?" 스스로 질문해 본다.

---

### 실습 2: 컨텍스트 지도 파일 만들기

**목표**: 프로젝트 구조를 한눈에 보여주는 `CONTEXT.md` 파일을 만든다.

1. 실습 1 결과를 바탕으로 가장 중요한 파일 3~5개를 고른다.
2. `generate_context_map.py`를 실행해 프로젝트 맵을 생성한다.
3. 생성된 내용을 `CONTEXT.md`에 저장한다.

```python
# generate_context_map.py (실습용 단순화 버전)
import os
from pathlib import Path

def build_map(root=".", max_depth=2):
    lines = ["# 프로젝트 컨텍스트 지도\n"]
    ignore = {".git", "__pycache__", ".venv", "node_modules"}
    
    for dirpath, dirnames, filenames in os.walk(root):
        dirnames[:] = [d for d in dirnames if d not in ignore]
        depth = dirpath.replace(root, "").count(os.sep)
        if depth > max_depth:
            continue
        indent = "  " * depth
        folder = os.path.basename(dirpath) or "."
        lines.append(f"{indent}- **{folder}/**")
        for f in sorted(filenames):
            if f.endswith((".py", ".md", ".yaml", ".toml", ".env.example")):
                lines.append(f"{indent}  - `{f}`")
    
    return "\n".join(lines)

content = build_map()
Path("CONTEXT.md").write_text(content, encoding="utf-8")
print("CONTEXT.md 생성 완료!")
print(content[:300])
```

4. 생성된 `CONTEXT.md`를 열어 내용을 확인한다. 이 파일이 바로 새 AI 세션을 시작할 때 첫 번째로 붙여 넣을 내용이 된다.

---

### 실습 3: 세션 요약 파일로 작업 이어가기

**목표**: 오늘 한 AI 작업을 `ai_session.json`에 저장하고, 내일 새 세션에서 이어받는 흐름을 체험한다.

1. `session_summary.py`의 `WorkSession` 클래스를 복사해서 저장한다.
2. 현재 진행 중인 실제 프로젝트나 과제의 상태를 입력한다.

```python
# 내 작업 상태 기록
from session_summary import WorkSession

session = WorkSession(
    project_name="여기에 내 프로젝트 이름",
    goal="여기에 전체 목표",
    current_task="지금 하고 있는 작업",
)
session.add_completed("오늘 완료한 작업 1")
session.add_completed("오늘 완료한 작업 2")
session.known_issues.append("아직 해결 못 한 문제")
session.modified_files.append("수정한 파일 경로")

session.save("my_session.json")
print("저장 완료!")
print(session.to_context_prompt())
```

3. 저장된 `my_session.json`을 열어 내용이 맞는지 확인한다.
4. `to_context_prompt()` 출력 내용을 실제 AI 채팅 창에 붙여 넣고, 이전 대화 없이도 맥락을 이해하는지 테스트해 본다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| 전체 파일을 모두 붙여 넣기 | AI가 앞 대화를 잊어버림, 엉뚱한 파일 언급 | 필요한 함수 10~30줄만 잘라서 전달 |
| 너무 긴 대화를 이어감 | "이전에 말씀하신 `xxx` 함수는..." 에서 틀린 이름 사용 | 30개 이상 메시지 → 새 세션 + 요약 붙여넣기 |
| `open()` 인코딩 미지정 | `UnicodeDecodeError: 'cp949' codec can't decode` | `open(path, encoding="utf-8")` 명시 |
| `ast.parse()` 에 잘못된 문자열 전달 | `SyntaxError: invalid syntax` | 파일이 실제 유효한 파이썬 코드인지 먼저 확인 |
| 요약 없이 파일만 전달 | AI가 "어떤 부분을 봐야 하나요?"라고 되물음 | 파일 + 구체적인 질문 + 원하는 출력 형식 함께 전달 |
| `.git` 폴더 스캔 | 프로그램이 느려지거나 수천 개 파일 출력 | `os.walk` 시 `ignore_dirs`에 `.git` 포함 |

---

## 확인 체크리스트

- [ ] 컨텍스트 윈도우의 개념과 토큰 한계를 설명할 수 있다
- [ ] `estimate_tokens.py`를 실행해서 내 프로젝트의 토큰 사용량을 확인했다
- [ ] 전체 파일 대신 필요한 함수만 추출하는 방법을 이해한다
- [ ] `CONTEXT.md` 파일을 직접 생성했다
- [ ] `WorkSession` 클래스로 작업 상태를 저장하고 불러올 수 있다
- [ ] 컨텍스트가 가득 찼을 때 나타나는 증상 3가지를 말할 수 있다
- [ ] `UnicodeDecodeError`가 발생했을 때 해결 방법을 안다

---

## 한 번 더 생각해 보기

1. **청킹 전략의 한계**: 함수 시그니처만 추출하면 토큰은 줄어들지만, AI가 함수 내부 구현을 모르게 됩니다. 어떤 상황에서는 전체 코드가 필요할까요? 그 기준을 어떻게 정할 수 있을까요?

2. **요약의 정확성**: `WorkSession`에 저장하는 내용은 사람이 직접 작성합니다. 만약 요약이 잘못되었다면 AI도 잘못된 방향으로 작업할 수 있습니다. 요약의 정확성을 높이기 위해 어떤 내용을 반드시 포함해야 할까요?

3. **자동화 가능성**: `estimate_tokens.py`와 `generate_context_map.py`는 매번 손으로 실행해야 합니다. 이 과정을 자동화하려면 어떻게 하면 좋을까요? (힌트: `git commit` 훅이나 CI 파이프라인을 생각해 보세요)

---

## 다음 장

다음 장에서는 AI가 생성한 코드를 GitHub Actions로 자동 검증하는 CI 파이프라인을 구성해, 컨텍스트 관리와 품질 보증을 하나의 워크플로로 연결하는 방법을 배웁니다.