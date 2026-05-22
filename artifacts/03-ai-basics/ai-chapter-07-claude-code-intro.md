# Chapter 07: Claude Code 입문

## 이 장에서 배우는 것

- Claude Code가 무엇인지, 일반 AI 채팅 및 VS Code AI 확장과 어떻게 다른지
- 설치부터 첫 실행까지 단계별로 따라 하기
- Claude Code로 파일을 직접 수정하는 방식 이해하기
- CLAUDE.md로 프로젝트 컨텍스트를 유지하는 방법
- 일반 AI 채팅과 Claude Code의 핵심 차이 3가지
- 자주 쓰는 요청 패턴과 안전한 사용 습관

---

## 먼저 쉬운 설명

지금까지 배운 AI 도구들을 떠올려보자.

- **GitHub Copilot**: 코드를 입력하는 동안 자동완성 제안
- **ChatGPT / Claude.ai**: 브라우저에서 대화형 질문과 답변
- **VS Code AI 채팅**: VS Code 안에 채팅 패널을 열어 질문

그리고 이제 **Claude Code**가 등장한다.

```
일반 AI 채팅:
  "이 함수를 고쳐줘" → AI가 코드를 텍스트로 보여줌 → 내가 직접 복사·붙여넣기

Claude Code:
  "이 함수를 고쳐줘" → AI가 파일을 직접 읽고, 직접 수정
```

이것을 **Agentic AI**라고 한다. AI가 단순히 답을 보여주는 것이 아니라, 실제로 파일을 열고 쓰고, 터미널 명령을 실행하는 **행위자(agent)** 역할을 한다.

---

## 1. 왜 Claude Code인가 — VS Code AI 채팅과 무엇이 다를까

### VS Code에서 AI를 쓰는 일반적인 방법

많은 사람들이 VS Code에서 AI를 이렇게 쓴다.

1. VS Code를 열고 GitHub Copilot Chat 또는 Continue.dev 같은 확장을 설치
2. 사이드바에 채팅 패널이 생김
3. 패널에 "이 함수 설명해줘" 또는 "버그를 찾아줘"라고 입력
4. AI가 텍스트로 답변을 보여줌
5. 수정이 필요하면 내가 직접 편집기에 붙여넣거나, 제안 버튼을 눌러 적용

이 방식은 편하지만, **AI는 내가 허락한 영역만 봄**. 열린 파일이나 선택한 영역만 컨텍스트로 전달된다. 프로젝트 전체 구조를 AI가 파악하려면 파일을 일일이 열어서 붙여넣어야 한다.

### Claude Code가 다른 점

Claude Code는 터미널에서 실행되는 CLI(Command Line Interface) 도구다.

```
VS Code AI 채팅:
  - VS Code가 켜져 있어야 함
  - 열린 파일이나 선택 영역만 컨텍스트로 전달
  - AI가 제안한 내용을 내가 직접 적용
  - 터미널 명령 실행 불가

Claude Code:
  - 터미널에서 독립 실행
  - 프로젝트 폴더 전체를 자유롭게 탐색
  - 파일을 직접 읽고, 직접 수정
  - 터미널 명령(테스트 실행, 빌드 등)도 실행 가능
```

예를 들어 "이 프로젝트에서 오류가 발생하는 이유를 찾아줘"라고 요청하면:
- VS Code AI 채팅: 열린 파일들만 보고 추측해서 답변
- Claude Code: 관련 파일들을 스스로 탐색하고, 테스트를 실행하고, 실제 오류를 재현해서 원인 파악

**간단히 말하면:** VS Code AI 채팅은 "조언을 주는 동료"이고, Claude Code는 "직접 작업하는 동료"다.

---

## 2. 핵심 차이 3가지

일반 AI 채팅(Claude.ai, ChatGPT 브라우저 버전)과 Claude Code의 결정적 차이는 세 가지다.

### 차이 1: 파일을 직접 수정한다

| 일반 AI 채팅 | Claude Code |
|-------------|------------|
| 코드를 텍스트로 보여줌 | 파일을 직접 열어서 수정 |
| 내가 복사해서 붙여넣어야 함 | 변경 확인 후 바로 저장 |
| 여러 파일을 동시에 바꾸기 어려움 | 여러 파일을 한 번에 수정 가능 |

```
예시: "add_memo 함수에 빈 문자열 검증을 추가해줘"

일반 AI 채팅:
  → "아래와 같이 수정하세요: ..." (코드 블록 텍스트)
  → 내가 memo_service.py를 열고 직접 붙여넣기

Claude Code:
  → memo_service.py를 직접 열어서 수정
  → 변경 내용을 diff로 보여줌
  → y 입력 시 저장 완료
```

### 차이 2: 터미널 명령을 실행한다

Claude Code는 파일 수정에 그치지 않는다. 테스트를 실행하고, 결과를 확인하고, 오류가 있으면 다시 수정한다.

```
예시: "테스트를 실행해서 오류가 없는지 확인해줘"

일반 AI 채팅:
  → "python3 -m unittest ...을 실행해보세요" (명령을 알려줌)
  → 내가 직접 터미널을 열어서 실행

Claude Code:
  → python3 -m unittest ... 를 직접 실행
  → 오류가 있으면 원인을 분석해서 수정
  → 다시 테스트 실행해서 통과 확인
```

### 차이 3: 프로젝트 전체를 파악한다

브라우저 AI 채팅에 코드를 붙여넣으면 그 내용만 안다. Claude Code는 프로젝트 폴더 구조, 파일 내용, 설정 파일을 스스로 탐색한다.

```
예시: "이 프로젝트에서 memo를 삭제하는 로직이 어디 있어?"

일반 AI 채팅:
  → "코드를 붙여넣어 주시면 확인해드리겠습니다"

Claude Code:
  → 폴더를 스스로 탐색해서 관련 파일을 찾음
  → memo_service.py의 delete_memo 함수를 바로 확인
  → 필요하면 연관 파일도 함께 분석
```

---

## 3. 설치 방법 단계별

### 전제 조건 확인

Claude Code를 설치하기 전에 아래를 확인한다.

**Node.js 버전 확인:**
```bash
node --version
# v18.0.0 이상이어야 함
```

Node.js가 없거나 버전이 낮으면 https://nodejs.org 에서 LTS 버전을 설치한다.

**npm 버전 확인:**
```bash
npm --version
# 8.0.0 이상이면 충분
```

**Anthropic 계정:** https://claude.ai 에서 계정을 만들어 두거나, 기존 계정이 있으면 준비한다.

---

### 설치

```bash
npm install -g @anthropic-ai/claude-code
```

`-g` 옵션은 전역(global) 설치를 의미한다. 어느 폴더에서든 `claude` 명령으로 실행할 수 있게 된다.

설치 확인:
```bash
claude --version
```

버전 번호가 출력되면 설치 성공이다.

---

### 인증 방법

Claude Code를 처음 실행하면 Anthropic 계정 인증이 필요하다. 두 가지 방법이 있다.

**방법 A: 브라우저 로그인 (권장, 가장 간단)**

```bash
claude
```

처음 실행 시 아래와 같은 메시지가 나온다:

```
Welcome to Claude Code!
Let's get you set up. Please visit the following URL to authenticate:

https://claude.ai/oauth/authorize?...

Waiting for authentication...
```

브라우저에서 URL을 열면 claude.ai 로그인 화면이 나온다. 로그인 후 "허용" 버튼을 클릭하면 터미널에서 자동으로 인증 완료 메시지가 뜬다.

**방법 B: API 키 직접 사용**

Anthropic 유료 계정(API 플랜)이 있다면 API 키를 환경 변수로 설정할 수 있다.

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
claude
```

일반 claude.ai 유료 구독(Pro/Max) 사용자는 방법 A가 더 간단하다.

---

## 4. 첫 실행: 터미널에서 대화 시작하기

### 프로젝트 폴더에서 실행

Claude Code는 현재 폴더를 기준으로 파일을 탐색한다. 항상 작업할 프로젝트 폴더로 먼저 이동한 다음 실행한다.

```bash
cd /path/to/my-project
claude
```

### 대화 프롬프트

실행하면 아래와 같은 화면이 나온다:

```
Claude Code  (version x.x.x)
Working directory: /path/to/my-project

> _
```

`>` 다음에 자연어로 요청을 입력한다.

```
> 이 프로젝트의 파일 목록을 보여주고 각 파일이 무엇을 하는지 설명해줘
```

Enter를 누르면 Claude Code가 폴더를 탐색하고, 각 파일의 역할을 설명한다.

### 한 번만 실행하는 방법

대화 모드 대신 명령 한 줄로 실행하고 바로 종료하려면 `-p` 옵션을 쓴다.

```bash
claude -p "이 프로젝트의 파일 목록을 보여줘"
```

자동화 스크립트나 빠른 확인 용도로 유용하다.

---

## 5. 파일 직접 수정: 실제로 어떻게 동작하는가

### 수정 요청 흐름

"이 파일을 수정해줘"라고 요청하면 Claude Code는 다음 순서로 동작한다.

```
1. 파일 읽기
   → memo_service.py를 열어서 현재 내용을 분석

2. 변경 계획 수립
   → 무엇을 어떻게 바꿀지 결정

3. 변경 내용 미리 보기 (diff 형식)
   → 삭제할 줄은 빨간색(또는 - 표시)
   → 추가할 줄은 초록색(또는 + 표시)

4. 승인 요청
   → "Proceed? (y/n)"

5. 승인 시 파일 저장
   → 실제 파일에 변경사항 적용
```

### diff 읽는 방법

```diff
  def add_memo(memos, text):
-     memos.append(text)
+     if not text or not text.strip():
+         return False
+     memos.append(text.strip())
+     return True
```

- `-`로 시작하는 줄: 삭제되는 내용
- `+`로 시작하는 줄: 추가되는 내용
- 공백으로 시작하는 줄: 변경 없이 유지되는 내용 (맥락 표시)

이 diff는 "기존에 memos.append(text)만 하던 것을, 빈 문자열 검증 후 처리하도록 바꾼다"는 의미다.

### y / n 판단 기준

```
y를 누르기 전에 확인할 것:
  □ 내가 요청한 내용과 일치하는가
  □ 예상치 못한 다른 변경이 포함되어 있지 않은가
  □ 삭제되는 줄이 있다면, 정말 필요 없는 코드인가
```

의심스러우면 `n`을 누르고 요청을 더 구체적으로 다시 입력한다.

---

## 6. CLAUDE.md — 프로젝트 컨텍스트 유지하기

### CLAUDE.md란

프로젝트 폴더 최상단에 `CLAUDE.md` 파일을 만들면, Claude Code는 실행될 때마다 이 파일을 자동으로 읽는다. 프로젝트에 대한 중요한 정보를 여기에 기록해두면, 매번 설명할 필요가 없다.

### 왜 필요한가

```
CLAUDE.md 없을 때:
  > 이 프로젝트는 Python 3.11을 쓰고, 테스트는 pytest를 써.
    requirements.txt에 의존성이 있고, main.py가 진입점이야.
    그리고 함수 이름은 snake_case로 써야 해. 지금 add_user 함수 좀 추가해줘.

CLAUDE.md 있을 때:
  > add_user 함수 추가해줘
  (나머지 정보는 CLAUDE.md에서 Claude Code가 자동으로 읽음)
```

### CLAUDE.md 작성 예시

```markdown
# 프로젝트: Memo Service

## 기술 스택
- Python 3.11
- 테스트: unittest (표준 라이브러리)
- 실행: python3 main.py

## 프로젝트 구조
- memo_service.py: 메모 CRUD 핵심 로직
- test_memo_service.py: 단위 테스트
- main.py: CLI 진입점

## 코딩 규칙
- 함수 이름: snake_case
- 반환값: 성공 시 True/결과, 실패 시 False/None
- 빈 문자열 입력은 항상 거부

## 실행 방법
테스트 실행: python3 -m unittest test_memo_service.py -v
프로그램 실행: python3 main.py
```

### 어디에 만드는가

```
my-project/
├── CLAUDE.md          ← 여기 (프로젝트 루트)
├── memo_service.py
├── test_memo_service.py
└── main.py
```

`claude` 명령을 실행하는 폴더와 같은 위치에 두면 자동으로 인식된다.

### CLAUDE.md에 넣으면 좋은 내용

```
□ 프로젝트 목적과 간단한 설명
□ 사용하는 언어와 주요 프레임워크
□ 파일/폴더 구조 설명
□ 코딩 컨벤션 (네이밍 규칙, 들여쓰기 등)
□ 테스트 실행 명령
□ 빌드/실행 명령
□ 특별히 주의할 사항 (레거시 코드, 변경 금지 영역 등)
```

---

## 7. 자주 쓰는 Claude Code 패턴

### 패턴 1: 파일 수정 요청

```
> memo_service.py의 add_memo 함수에 빈 문자열 검증을 추가해줘.
  빈 문자열이나 공백만 있는 문자열이 들어오면 False를 반환해야 해.
```

팁: 무엇을 원하는지(빈 문자열 거부)와 어떤 결과를 원하는지(False 반환)를 함께 명시하면 정확도가 올라간다.

---

### 패턴 2: 버그 찾기

```
> test_memo_service.py를 실행했더니 AssertionError가 났어.
  오류 메시지는 "False is not true"야. 원인을 찾아서 고쳐줘.
```

또는 오류 메시지를 직접 붙여넣는 방법도 유효하다.

```
> 아래 오류가 발생했어. 원인을 찾아줘:

  Traceback (most recent call last):
    File "test_memo_service.py", line 12, in test_add_memo
      self.assertTrue(add_memo(memos, ""))
  AssertionError: False is not true
```

---

### 패턴 3: 새 기능 추가

```
> memo_service.py에 search_memo(memos, keyword) 함수를 추가해줘.
  keyword가 포함된 메모 목록을 반환하고, 대소문자를 구분하지 않아야 해.
  추가 후에 test_memo_service.py에 테스트도 작성해줘.
```

한 번의 요청으로 구현과 테스트를 함께 요청할 수 있다.

---

### 패턴 4: 코드 설명

```
> memo_service.py 전체를 읽고 각 함수가 무엇을 하는지 설명해줘.
  처음 보는 사람도 이해할 수 있도록 쉽게 설명해줘.
```

```
> delete_memo 함수에서 enumerate를 쓰는 이유가 뭐야?
```

---

### 패턴 5: 리팩토링

```
> delete_memo 함수를 더 읽기 쉽게 개선해줘. 기능은 그대로 유지해야 해.
```

```
> memo_service.py에서 중복된 코드가 있으면 찾아서 함수로 추출해줘.
```

---

### 패턴 6: 프로젝트 전체 파악

```
> 이 프로젝트의 구조를 설명해줘. 어떤 파일이 어떤 역할을 하는지 알고 싶어.
```

```
> 이 프로젝트에서 메모를 삭제하는 로직이 어디서 시작해서 어디서 끝나는지 흐름을 설명해줘.
```

---

## 8. 안전한 사용법

Claude Code는 파일을 직접 수정하므로, 실수를 되돌리기 어려울 수 있다. 다음 습관을 들이면 안전하게 쓸 수 있다.

### 습관 1: 요청 전 git commit

파일을 수정하기 전에 항상 현재 상태를 git에 저장한다.

```bash
git add .
git commit -m "Claude Code 작업 전 현재 상태 저장"
```

실수로 원하지 않는 변경이 생겨도 `git checkout` 또는 `git restore`로 되돌릴 수 있다.

### 습관 2: diff를 반드시 읽고 승인

```
□ Claude Code가 보여주는 변경사항을 한 줄씩 읽는다
□ 내가 요청하지 않은 변경이 포함되어 있지 않은지 확인
□ 삭제되는 코드(-로 표시)가 정말 필요 없는 내용인지 확인
□ 확신이 없으면 n을 누르고 요청을 다시 작성
```

### 습관 3: 브랜치에서 작업

큰 작업을 할 때는 별도 브랜치를 만들어서 작업한다.

```bash
git checkout -b claude-code-experiment
# 이 브랜치에서 Claude Code로 작업
# 만족스러우면 main에 merge
# 마음에 안 들면 브랜치 삭제
```

### 습관 4: 작게 쪼개서 요청

```
나쁜 요청:
  > 이 서비스 전체를 리팩토링하고, 테스트도 추가하고,
    오류 처리도 개선하고, 문서도 작성해줘.

좋은 요청:
  > add_memo 함수에만 오류 처리를 추가해줘. (확인 후 다음 단계 진행)
  > 다음으로 delete_memo 함수에 오류 처리를 추가해줘.
```

한 번에 너무 많은 변경을 요청하면 무엇이 바뀌었는지 파악하기 어렵다.

### 습관 5: 변경 후 테스트 실행

파일 수정이 끝나면 기존 테스트가 여전히 통과하는지 확인한다.

```bash
python3 -m unittest test_memo_service.py -v
```

또는 Claude Code 안에서 직접 요청할 수도 있다.

```
> 방금 변경사항을 테스트해줘. 기존 테스트가 모두 통과하는지 확인해.
```

### 위험한 명령 주의

Claude Code가 아래와 같은 명령을 실행하려 할 때는 신중히 검토한다.

```
주의가 필요한 명령:
  □ rm, rmdir — 파일/폴더 삭제
  □ git reset --hard — 변경사항 강제 초기화
  □ DROP TABLE / DELETE FROM — 데이터베이스 데이터 삭제
  □ pip uninstall / npm uninstall — 패키지 삭제
```

"정말 이 작업이 필요한가?"를 한 번 더 생각하고 승인한다.

---

## 9. `/` 슬래시 명령어

Claude Code 대화창 안에서 슬래시(/)로 시작하는 명령을 쓸 수 있다.

| 명령 | 의미 |
|------|------|
| `/help` | 사용 가능한 명령 목록 보기 |
| `/clear` | 대화 내용 초기화 (컨텍스트 리셋) |
| `/exit` | Claude Code 종료 |
| `/diff` | 이번 세션에서 수정한 내용 확인 |
| `/undo` | 마지막 변경사항 취소 |

### 언제 `/clear`를 쓰는가

대화가 길어지면 Claude Code가 이전 대화 내용을 컨텍스트로 계속 유지하는데, 이것이 혼란을 줄 수 있다. 새로운 작업을 시작할 때는 `/clear`로 초기화하면 더 정확한 응답을 받을 수 있다.

---

## 10. 일반 AI 채팅과 비교 정리

| 항목 | Claude.ai 브라우저 | VS Code AI 채팅 | Claude Code |
|------|------------------|---------------|------------|
| 실행 위치 | 브라우저 | VS Code 내 | 터미널 |
| 파일 접근 | 직접 불가 (복붙 필요) | 열린 파일만 | 폴더 전체 자유롭게 탐색 |
| 파일 수정 | 직접 불가 | 제안 후 내가 적용 | 직접 수정 |
| 터미널 명령 실행 | 불가 | 불가 | 가능 |
| 프로젝트 컨텍스트 | 없음 (매번 설명) | 제한적 | CLAUDE.md로 유지 |
| 적합한 상황 | 개념 설명, 짧은 질문 | 빠른 코드 제안 | 파일 수정, 리팩토링, 버그 수정 |

---

## 11. 따라 하기 실습

### 실습 1: 설치 확인

**목표:** Claude Code가 제대로 설치되었는지 확인한다.

```bash
# 설치
npm install -g @anthropic-ai/claude-code

# 버전 확인
claude --version
```

예상 출력:
```
@anthropic-ai/claude-code/x.x.x ...
```

버전 번호가 출력되면 설치 성공이다.

**로그인 확인:**

```bash
cd memo-service  # 또는 아무 프로젝트 폴더
claude
```

첫 실행 시 브라우저 인증 화면이 열린다. 로그인 후 터미널로 돌아오면:

```
> _
```

프롬프트가 나타나면 준비 완료다.

첫 번째 요청으로 아래를 입력해 보자:

```
> 현재 폴더에 어떤 파일이 있는지 알려줘
```

---

### 실습 2: 간단한 파일 수정 요청

**목표:** Claude Code가 파일을 직접 수정하는 과정을 경험한다.

실습용 파일 `memo_service.py`가 있는 폴더에서 Claude Code를 실행한다.

```bash
cd memo-service
claude
```

요청 입력:

```
> memo_service.py에 메모 개수를 반환하는 count_memos(memos) 함수를 추가해줘
```

Claude Code가 diff를 보여줄 것이다:

```diff
+ def count_memos(memos):
+     return len(memos)
```

**diff 읽기 체크:**
- `+`가 붙은 줄이 추가되는 코드다
- 내가 요청한 것(count_memos 함수)과 일치하는가?

일치하면 `y`를 입력해서 승인한다.

파일에 실제로 반영되었는지 확인:

```bash
grep -n "count_memos" memo_service.py
```

---

### 실습 3: diff로 검토하기

**목표:** 변경 내용을 git diff로 확인하는 습관을 익힌다.

실습 2에서 변경이 완료된 후, 터미널에서 확인한다.

```bash
# Claude Code에서 나가기 (Ctrl+C 또는 /exit)

# git에 추가되지 않은 변경 확인
git diff memo_service.py
```

출력 예시:

```diff
diff --git a/memo_service.py b/memo_service.py
index abc1234..def5678 100644
--- a/memo_service.py
+++ b/memo_service.py
@@ -15,3 +15,7 @@ def delete_memo(memos, index):
         return False
     memos.pop(index)
     return True
+
+def count_memos(memos):
+    return len(memos)
```

변경 내용이 마음에 들면 commit한다:

```bash
git add memo_service.py
git commit -m "feat: count_memos 함수 추가 (Claude Code 사용)"
```

변경 내용이 마음에 들지 않으면 되돌린다:

```bash
git restore memo_service.py
```

---

## 확인 체크리스트

- [ ] Claude Code를 설치하고 `claude --version`으로 확인할 수 있는가
- [ ] 터미널에서 `claude`를 입력해 대화를 시작할 수 있는가
- [ ] Claude Code와 브라우저 AI 채팅의 핵심 차이 3가지를 설명할 수 있는가
- [ ] diff 출력에서 `+`와 `-`가 무엇을 의미하는지 아는가
- [ ] 파일 수정 전에 git commit하는 이유를 설명할 수 있는가
- [ ] CLAUDE.md가 무엇이고, 어디에 두는지 아는가
- [ ] 실습 2에서 수정된 파일을 `git diff`로 확인할 수 있는가

---

## 한 번 더 생각해 보기

1. Claude Code가 파일을 직접 수정할 수 있다면, 요청을 모호하게 쓰면 어떤 일이 생길까?
2. VS Code AI 채팅이 더 적합한 상황과 Claude Code가 더 적합한 상황을 각각 하나씩 떠올려보자.
3. CLAUDE.md에 "이 파일은 절대 수정하지 마세요"라고 써두면 Claude Code가 지킬까? 왜 그럴까, 왜 아닐까?
4. 요청 하나에 여러 파일을 동시에 수정하도록 시켰을 때, 리뷰를 어떻게 하면 효율적일까?

---

## 참고 자료

- Claude Code 공식 문서 — https://docs.anthropic.com/en/docs/claude-code
- Claude Code 설치 가이드 — https://docs.anthropic.com/en/docs/claude-code/getting-started
- Claude Code GitHub — https://github.com/anthropics/claude-code
