# ai-train AI 도구 학습 지도

## 목표

AI 도구를 올바르게 이해하고 활용해서, AI와 협업하면서도 스스로 검증하고 판단하는 개발 능력을 기른다.

---

## 이 파트의 핵심 질문

> "AI가 다 해준다면 나는 뭘 배워야 할까?"

이 파트는 그 질문에 대한 답이다. AI를 도구로 쓰면서 실력을 쌓는 방법을 배운다.

---

## 권장 학습 순서

### 0단계. 준비 확인

- Python 기초(Part 2)를 어느 정도 익혔는가
- VS Code가 설치되어 있는가
- GitHub 계정이 있는가 (Copilot 사용 시 필요)

---

### 1단계. AI 활용의 기본 원칙

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 1 | [Chapter 01: AI 워크플로우와 검증](../03-ai-basics/ai-train-ai-chapter-01-workflow-and-verification.md) | AI 활용 원칙, 생성→읽기→수정→확인 루프 |
| 2 | [Chapter 08: 직접 배울 것 vs AI 위임](../03-ai-basics/ai-train-ai-chapter-08-teach-vs-delegate.md) | 경계선 판단 기준, 에러 직접 읽기 |

**이 단계 마치면**: AI를 쓸 때 어디까지 믿고 어디서 검증해야 하는지 판단할 수 있다.

---

### 2단계. VS Code에 AI 도구 연결

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 3 | [Chapter 03: VS Code AI 도구 설치](../03-ai-basics/ai-train-ai-chapter-03-vscode-ai-setup.md) | Copilot, Codeium, Claude 설치 및 기본 사용 |
| 4 | [Chapter 07: Claude Code 입문](../03-ai-basics/ai-train-ai-chapter-07-claude-code-intro.md) | Claude Code 설치, 파일 직접 수정, 안전한 사용 |
| 5 | [Chapter 09: AI 도구 선택 가이드](../03-ai-basics/ai-train-ai-chapter-09-tool-selection-guide.md) | Claude vs Copilot vs Cursor 비교, 상황별 선택법 |

**이 단계 마치면**: 상황에 맞는 AI 도구를 골라 VS Code에서 사용할 수 있다.

---

### 3단계. 프롬프트와 코드 품질

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 6 | [Chapter 04: 좋은 프롬프트 작성법](../03-ai-basics/ai-train-ai-chapter-04-prompt-writing.md) | 역할/목표/제약/형식 4요소, 코딩/설명/검토 패턴 |
| 7 | [Chapter 02: AI 초안 수정](../03-ai-basics/ai-train-ai-chapter-02-draft-correction.md) | AI 코드 읽고 한 줄씩 이해하며 수정하기 |

**이 단계 마치면**: 원하는 결과를 얻는 프롬프트를 작성하고, 받은 코드를 직접 이해할 수 있다.

---

### 4단계. AI 코드 검증과 보안

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 8 | [Chapter 05: AI 코드 검증 체크리스트](../03-ai-basics/ai-train-ai-chapter-05-verification-checklist.md) | 정상/엣지 케이스 테스트, assert, 자주 나타나는 문제 |
| 9 | [Chapter 06: AI 환각 감지 체크리스트](../03-ai-basics/ai-train-ai-chapter-06-hallucination-checklist.md) | 존재하지 않는 함수, 잘못된 API, 공식 문서 대조 |
| 10 | [Chapter 10: AI 보안 기초](../03-ai-basics/ai-train-ai-chapter-10-ai-security-basics.md) | Prompt Injection, 사용자 입력 검증, 안전한 AI 사용 |

**이 단계 마치면**: AI가 만든 코드의 함정을 찾아내고 안전하게 배포할 수 있다.

---

### 실습

| 파일 | 내용 |
|------|------|
| [초안 수정 랩](../03-ai-basics/ai-train-ai-lab-01-draft-correction.md) | AI 초안을 받아 단계별로 수정하는 실습 |

---

## 학습 팁

- **Chapter 08을 먼저 읽어도 좋다**: "뭘 직접 배워야 하는지"를 알면 나머지 챕터를 대하는 태도가 달라진다
- **프롬프트는 연습이 필요하다**: 같은 요청도 구체적일수록 더 좋은 결과가 나온다. 여러 번 시도해보자
- **환각은 자신감 있게 온다**: AI가 확신 있게 말한다고 맞는 게 아니다. 항상 실행해서 확인하자

## 자주 막히는 지점

| 상황 | 확인할 것 |
|------|----------|
| AI 코드가 실행이 안 됨 | 존재하지 않는 함수/메서드인지 공식 문서 확인 |
| AI 설명이 코드와 다름 | 코드를 직접 실행해서 결과 확인 |
| Copilot 제안이 안 뜸 | VS Code 하단 Copilot 아이콘 활성화 상태 확인 |
| Claude Code가 잘못 수정 | `git diff`로 변경 내용 확인 후 `git restore` |
