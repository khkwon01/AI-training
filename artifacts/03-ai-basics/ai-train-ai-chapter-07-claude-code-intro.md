# ai-train AI Chapter 07: Claude Code 입문

## 이 장에서 배우는 것

- Claude Code가 무엇인지, 일반 AI 채팅과 어떻게 다른지
- VS Code에서 Claude Code를 설치하고 사용하는 방법
- Claude Code로 코드 작성, 설명, 수정을 요청하는 방법
- Claude Code가 파일을 직접 수정하는 방식 이해하기
- 초보자가 Claude Code를 안전하게 쓰는 습관

---

## 먼저 쉬운 설명

지금까지 배운 AI 도구들을 떠올려보자.

- **Copilot**: 코딩 중 자동완성 제안
- **ChatGPT/Claude.ai**: 브라우저에서 대화형 질문

**Claude Code**는 다르다. 터미널에서 실행되고, 실제 파일을 직접 읽고 수정할 수 있다.

```
일반 AI 채팅: "이렇게 고쳐줘" → 코드를 텍스트로 보여줌 → 내가 직접 복붙
Claude Code: "이렇게 고쳐줘" → 파일을 직접 수정
```

이것을 **Agentic AI**라고 한다. AI가 스스로 파일을 읽고, 쓰고, 명령을 실행한다.

---

## 1. Claude Code 설치

### 전제 조건

- Node.js 18 이상 설치
- Anthropic 계정 (claude.ai)

### 설치

```bash
npm install -g @anthropic-ai/claude-code
```

### 실행

프로젝트 폴더에서:

```bash
claude
```

처음 실행하면 Anthropic 계정으로 로그인하라는 안내가 나온다.

---

## 2. 기본 사용법

### 대화 시작

Claude Code를 실행하면 대화창이 열린다.

```
> _
```

여기에 자연어로 요청한다.

```
> memo.py 파일에서 빈 문자열을 추가하려 할 때 False를 반환하는 add_memo 함수를 만들어줘
```

Claude Code는:
1. 현재 폴더의 파일들을 읽는다
2. 요청을 분석한다
3. 파일을 직접 수정하거나 새 파일을 만든다
4. 무엇을 했는지 설명한다

### 파일 내용 기반 질문

```
> memo.py 파일에서 add_memo 함수가 어떻게 동작하는지 설명해줘
```

Claude Code가 파일을 읽고 설명한다. 브라우저로 복붙하지 않아도 된다.

---

## 3. 자주 쓰는 요청 패턴

### 코드 작성

```
> search_memo(memos, keyword) 함수를 memo_service.py에 추가해줘.
  대소문자 구분 없이 검색해야 해.
```

### 코드 설명

```
> memo_service.py 전체를 읽고 각 함수가 하는 일을 간단히 설명해줘
```

### 버그 찾기

```
> test_memo_service.py를 실행했더니 오류가 났어. 원인을 찾아줘
```

### 리팩토링

```
> delete_memo 함수를 더 읽기 쉽게 개선해줘. 기능은 그대로 유지해야 해.
```

### 테스트 작성

```
> add_memo 함수에 대한 unittest 테스트를 test_memo_service.py에 추가해줘
```

---

## 4. Claude Code가 파일을 수정할 때

Claude Code는 파일을 수정하기 전에 무엇을 바꿀지 먼저 보여준다.

```
I'll modify memo_service.py to add input validation:

  def add_memo(memos, text):
-     memos.append(text)
+     if not text or not text.strip():
+         return False
+     memos.append(text.strip())
+     return True

Proceed? (y/n)
```

**`y`를 누르기 전에 반드시 읽는다.** 원하지 않는 변경이면 `n`을 누른다.

---

## 5. 일반 AI 채팅과 비교

| 항목 | Claude.ai (채팅) | Claude Code |
|------|----------------|------------|
| 실행 위치 | 브라우저 | 터미널 |
| 파일 접근 | 직접 불가 (복붙 필요) | 직접 읽고 수정 |
| 컨텍스트 | 대화 내용만 | 프로젝트 파일 전체 |
| 속도 | 즉시 | 파일 처리 시간 있음 |
| 적합한 상황 | 개념 설명, 짧은 질문 | 파일 수정, 코드 리팩토링 |

---

## 6. 안전하게 쓰는 습관

Claude Code는 파일을 직접 수정하므로 주의가 필요하다.

```
□ 요청 전에 git commit으로 현재 상태를 저장한다
□ Claude Code가 보여주는 변경사항을 반드시 읽고 승인한다
□ 중요한 파일은 백업하거나 branch를 만들고 작업한다
□ 한 번에 너무 많은 것을 요청하지 않는다 (작게 쪼개기)
□ 변경 후 테스트를 실행해서 기존 기능이 깨지지 않았는지 확인한다
```

---

## 7. `/` 명령어

Claude Code 안에서 슬래시 명령을 쓸 수 있다.

| 명령 | 의미 |
|------|------|
| `/help` | 사용 가능한 명령 목록 |
| `/clear` | 대화 내용 초기화 |
| `/exit` | Claude Code 종료 |
| `/diff` | 현재 변경사항 확인 |

---

## 8. 따라 하기 실습

### 실습 1. Claude Code 설치 및 첫 대화

```bash
npm install -g @anthropic-ai/claude-code
cd memo-service
claude
```

첫 번째 요청:
```
> 이 프로젝트의 파일 목록을 보여주고 각 파일이 무엇을 하는지 설명해줘
```

### 실습 2. 함수 추가 요청

```
> memo_service.py에 메모 개수를 반환하는 count_memos(memos) 함수를 추가해줘
```

변경사항을 보고 `y`로 승인. 파일에 실제로 추가됐는지 확인.

### 실습 3. 테스트 요청

```
> count_memos 함수에 대한 간단한 unittest 테스트를 test_memo_service.py에 추가해줘
```

테스트 파일 확인 후 실행:
```bash
python3 -m unittest test_memo_service.py -v
```

### 실습 4. git으로 변경 기록하기

Claude Code로 만든 변경사항을 commit한다.

```bash
git add memo_service.py test_memo_service.py
git commit -m "feat: count_memos 함수 추가 (Claude Code 사용)"
```

---

## 확인 체크리스트

- [ ] Claude Code를 설치하고 터미널에서 실행할 수 있는가
- [ ] 파일 수정 전에 변경사항을 읽고 판단하는 습관이 있는가
- [ ] Claude Code와 브라우저 AI 채팅의 차이를 설명할 수 있는가
- [ ] 요청 전에 git commit으로 상태를 저장하는가

---

## 한 번 더 생각해 보기

1. Claude Code가 파일을 직접 수정할 수 있다면, 요청을 잘못 쓰면 어떤 위험이 있을까?
2. 브라우저 채팅과 Claude Code를 각각 언제 쓰면 가장 효율적일까?
3. Claude Code의 응답을 그대로 믿지 않아야 하는 이유는 무엇인가?

---

## 참고 자료

- Claude Code 공식 문서 — https://docs.anthropic.com/en/docs/claude-code
- Claude Code 설치 가이드 — https://docs.anthropic.com/en/docs/claude-code/getting-started
