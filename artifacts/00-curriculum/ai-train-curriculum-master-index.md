# AI-training 전체 커리큘럼 마스터 인덱스

## 목표

코딩을 처음 시작하는 초보자가 Python 기초부터 AI 협업, GitHub, AWS 배포까지 단계별로 따라갈 수 있는 전체 학습 경로입니다.

---

## 전체 학습 경로

```
Part 1. 환경 준비 (01-python)
  ↓
Part 2. Python 기초 (01-python)
  ↓
Part 3. 코드 저장소 관리 (02-github)
  ↓
Part 4. AI 도구 연결 (03-ai-basics)
  ↓
Part 5. AI와 협업 코딩 (04-vibe-coding)
  ↓
Part 6. AWS 배포 (05-aws-deploy)
  ↓
Part 7. 실전 서비스 만들기 (06-demo-services)
```

---

## Part 1. 환경 준비

VS Code와 Python을 설치하고 코딩할 준비를 한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [VS Code 설치와 기본 사용](../01-python/ai-train-python-vscode-chapter-01.md) | VS Code 설치, 폴더 열기, 터미널, Python 확장 |
| 2 | [Python 설치와 VS Code 설정](../01-python/ai-train-python-vscode-chapter-02-python-setup.md) | Python 설치, 인터프리터 선택, venv 연결 |
| 실습 | [환경 설정 실습](../01-python/ai-train-python-practice-setup-01.md) | 설치 확인 및 첫 실행 |

---

## Part 2. Python 기초

Python 언어의 핵심 개념을 순서대로 익힌다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: print와 실행](../01-python/ai-train-python-chapter-01-print-and-run.md) | print, 파일 실행, 따옴표 |
| 2 | [Chapter 02: 변수](../01-python/ai-train-python-chapter-02-variables.md) | 변수, 이름 짓기, 값 변경 |
| 3 | [Chapter 03: 자료형](../01-python/ai-train-python-chapter-03-data-types.md) | int, float, str, bool, f-string |
| 4 | [Chapter 11: 조건문](../01-python/ai-train-python-chapter-11-conditions.md) | if/elif/else, 비교 연산자, 논리 연산자 |
| 5 | [Chapter 12: 반복문](../01-python/ai-train-python-chapter-12-loops.md) | for, while, range, break/continue |
| 6 | [Chapter 13: 함수](../01-python/ai-train-python-chapter-13-functions.md) | def, 매개변수, return, 기본값 |
| 7 | [Chapter 14: class와 object](../01-python/ai-train-python-chapter-14-class-and-object.md) | class, __init__, self, 메서드 |
| 8 | [Chapter 15: module과 import](../01-python/ai-train-python-chapter-15-module-and-import.md) | import, from, 표준 라이브러리, __name__ |
| 9 | [Chapter 16: 터미널/실행버튼/디버그 비교](../01-python/ai-train-python-chapter-16-run-debug-compare.md) | 3가지 실행 방법, 중단점, F10 단계 실행 |
| 9 | [Chapter 04: 파일과 JSON](../01-python/ai-train-python-chapter-04-files-and-json.md) | 파일 읽기/쓰기, JSON 저장 |
| 10 | [Chapter 05: 디버깅](../01-python/ai-train-python-chapter-05-debugging.md) | 오류 메시지 읽기, print 디버깅 |
| 11 | [Chapter 06: 프로젝트 구조](../01-python/ai-train-python-chapter-06-project-structure.md) | 폴더 구조, 이름 규칙, import |
| 12 | [Chapter 07: 실행 환경](../01-python/ai-train-python-chapter-07-environment.md) | 환경변수, dependency, path |
| 13 | [Chapter 08: 예외 처리](../01-python/ai-train-python-chapter-08-exceptions.md) | try/except, 예외 종류 |
| 14 | [Chapter 09: venv와 pip](../01-python/ai-train-python-chapter-09-venv-and-pip.md) | venv, pip, requirements.txt |
| 15 | [Chapter 10: comprehension과 datetime](../01-python/ai-train-python-chapter-10-comprehension-datetime.md) | 리스트 컴프리헨션, 날짜 다루기 |

**연습 자료**

| 파일 | 내용 |
|------|------|
| [venv/pip 실습](../01-python/ai-train-python-practice-venv-pip-01.md) | venv 만들기, 패키지 설치 실습 |
| [퀵 체크 1](../01-python/ai-train-python-quick-check-01.md) | Chapter 01~05 핵심 확인 문제 |
| [퀵 체크 2](../01-python/ai-train-python-quick-check-02.md) | Chapter 06~10 핵심 확인 문제 |
| [퀵 체크 3](../01-python/ai-train-python-quick-check-03.md) | Chapter 11~15 핵심 확인 문제 |
| [퀵 체크 1~2 정답](../01-python/ai-train-python-quick-check-answers.md) | 퀵 체크 1~2 정답 및 해설 |
| [퀵 체크 3 정답](../01-python/ai-train-python-quick-check-03-answers.md) | 퀵 체크 3 정답 및 해설 |
| [누적 실습 01](../01-python/ai-train-python-practice-cumulative-01.md) | Chapter 01~10 종합 실습 |
| [누적 실습 01 정답](../01-python/ai-train-python-practice-cumulative-01-answers.md) | 누적 실습 01 정답 |
| [누적 실습 02](../01-python/ai-train-python-practice-cumulative-02.md) | Chapter 11~15 연결 실습 (학생 성적 관리) |
| [조건문+반복문+함수 실습](../01-python/ai-train-python-practice-conditions-loops-functions.md) | 5단계 점진적 문제 세트 (소수, FizzBuzz, 통계 등) |
| [class+module 실습](../01-python/ai-train-python-practice-class-and-module.md) | 도서관·은행 시스템 구현 + 모듈 분리 연습 |

---

## Part 3. 코드 저장소 관리 (GitHub)

코드를 GitHub에 저장하고 VS Code와 연결한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: Git과 GitHub 시작](../02-github/ai-train-github-chapter-01-git-and-github.md) | Git 설치, GitHub 계정, clone |
| 2 | [Chapter 02: add, commit, push](../02-github/ai-train-github-chapter-02-add-commit-push.md) | 변경 기록하고 올리기 |
| 3 | [Chapter 03: AI review 워크플로우](../02-github/ai-train-github-chapter-03-ai-review-workflow.md) | PR, AI review, 협업 흐름 |
| 4 | [Chapter 04: VS Code와 GitHub 연결](../02-github/ai-train-github-chapter-04-vscode-github-connect.md) | Source Control UI, 계정 연결 |
| 5 | [Chapter 05: branch와 Pull Request](../02-github/ai-train-github-chapter-05-branch-and-pr.md) | branch 생성, PR 만들기, merge |
| 6 | [Chapter 06: merge conflict 해결](../02-github/ai-train-github-chapter-06-merge-conflict.md) | conflict 표시 읽기, VS Code로 해결 |
| 7 | [Chapter 07: issue 트래킹](../02-github/ai-train-github-chapter-07-issue-tracking.md) | issue 생성, branch 연결, 자동 close |
| 8 | [Chapter 08: PR 리뷰 가이드](../02-github/ai-train-github-chapter-08-pr-review-guide.md) | Files changed 읽기, 줄 코멘트, AI 코드 체크리스트 |

**실습 자료**

| 파일 | 내용 |
|------|------|
| [브라우저 따라하기](../02-github/ai-train-github-walkthrough-01-browser.md) | Issue → PR 흐름 실습 |
| [워크북](../02-github/ai-train-github-workbook-01.md) | GitHub 실습 워크북 |
| [AI review 랩](../02-github/ai-train-github-lab-01-ai-review.md) | AI review 실습 |
| [화면 참조 노트](../02-github/ai-train-github-cue-note-01-screen.md) | 화면 위치 빠른 참조 |

---

## Part 4. AI 도구 연결

VS Code에 AI를 연결하고 AI 활용 기본 습관을 익힌다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: AI 워크플로우와 검증](../03-ai-basics/ai-train-ai-chapter-01-workflow-and-verification.md) | AI 활용 원칙, 검증 습관 |
| 2 | [Chapter 02: AI 초안 수정](../03-ai-basics/ai-train-ai-chapter-02-draft-correction.md) | AI 코드 읽고 수정하기 |
| 3 | [Chapter 03: VS Code AI 도구 설치](../03-ai-basics/ai-train-ai-chapter-03-vscode-ai-setup.md) | Copilot, Codeium, Claude 연결 |
| 4 | [Chapter 04: 좋은 프롬프트 작성법](../03-ai-basics/ai-train-ai-chapter-04-prompt-writing.md) | 역할/목표/제약/형식, 패턴별 프롬프트 |
| 5 | [Chapter 05: AI 코드 검증 체크리스트](../03-ai-basics/ai-train-ai-chapter-05-verification-checklist.md) | 엣지 케이스 테스트, assert, 자주 나타나는 문제 패턴 |
| 6 | [Chapter 06: AI 환각 감지 체크리스트](../03-ai-basics/ai-train-ai-chapter-06-hallucination-checklist.md) | 환각 유형, 공식 문서 대조, assert 검증 |
| 7 | [Chapter 07: Claude Code 입문](../03-ai-basics/ai-train-ai-chapter-07-claude-code-intro.md) | 설치, 파일 직접 수정, 일반 채팅과 차이, 안전한 사용 |
| 8 | [Chapter 08: 직접 배울 것 vs AI 위임](../03-ai-basics/ai-train-ai-chapter-08-teach-vs-delegate.md) | 경계선 판단 기준, 에러 직접 읽기, AI 코드 주석 달기 |
| 9 | [Chapter 09: AI 도구 선택 가이드](../03-ai-basics/ai-train-ai-chapter-09-tool-selection-guide.md) | Claude vs Copilot vs Cursor 비교, 상황별 선택법 |
| 10 | [Chapter 10: AI 보안 기초](../03-ai-basics/ai-train-ai-chapter-10-ai-security-basics.md) | Prompt Injection, 사용자 입력 검증, 안전한 AI 사용 |

**실습**

| 파일 | 내용 |
|------|------|
| [초안 수정 랩](../03-ai-basics/ai-train-ai-lab-01-draft-correction.md) | AI 초안 수정 실습 |

---

## Part 5. AI와 협업 코딩 (Vibe Coding)

AI와 함께 기능을 만드는 방법을 익힌다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: Vibe coding 이해](../04-vibe-coding/ai-train-vibe-chapter-01-vibe-coding.md) | vibe coding 개념, 기본 원칙 |
| 2 | [Chapter 02: AI 코딩 루프](../04-vibe-coding/ai-train-vibe-chapter-02-ai-coding-loop.md) | 요청→확인→수정 루프 실습 |
| 3 | [Chapter 03: AI 코드 수정 루프](../04-vibe-coding/ai-train-vibe-chapter-03-repair-loop.md) | 오류→분석→수정→검증 반복 패턴 |
| 4 | [Chapter 04: 구현부터 배포까지 전체 워크플로우](../04-vibe-coding/ai-train-vibe-chapter-04-full-workflow.md) | issue→branch→AI코딩→PR→Lambda배포 전체 실습 |

**실습 자료**

| 파일 | 내용 |
|------|------|
| [프롬프트 세트 01](../04-vibe-coding/ai-train-vibe-prompt-set-01.md) | 기획·기능 분해 프롬프트 |
| [프롬프트 세트 02](../04-vibe-coding/ai-train-vibe-prompt-set-02.md) | 구현·리뷰·디버깅 프롬프트 |
| [워크북](../04-vibe-coding/ai-train-vibe-workbook-01.md) | Vibe coding 실습 워크북 |

---

## Part 6. AWS 배포

Python 코드를 AWS Lambda에 올리고 검증한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: 배포 개념과 검증](../05-aws-deploy/ai-train-aws-chapter-01-validation.md) | 배포란 무엇인가, 확인 방법 |
| 2 | [Chapter 02: AWS 계정과 Lambda](../05-aws-deploy/ai-train-aws-chapter-02-account-and-lambda-setup.md) | 계정 생성, IAM, Lambda 첫 배포 |
| 3 | [Chapter 03: API Gateway로 URL 만들기](../05-aws-deploy/ai-train-aws-chapter-03-api-gateway.md) | Function URL, API Gateway, curl 테스트 |
| 4 | [Chapter 04: 로깅과 모니터링](../05-aws-deploy/ai-train-aws-chapter-04-logging-and-monitoring.md) | CloudWatch Logs, 로그 패턴, 오류 추적 |
| 5 | [Chapter 05: 환경 변수와 보안 설정](../05-aws-deploy/ai-train-aws-chapter-05-env-and-security.md) | os.environ, Secrets Manager, IAM 최소 권한 |
| 6 | [Chapter 06: 배포 런북과 운영 체크리스트](../05-aws-deploy/ai-train-aws-chapter-06-deployment-runbook.md) | 배포 전후 체크리스트, 롤백, 주간 운영 점검 |

**참고 자료**

| 파일 | 내용 |
|------|------|
| [검증 실습](../05-aws-deploy/ai-train-aws-practice-01-validation.md) | Lambda 테스트 및 로그 확인 |
| [검증 한 페이지 요약](../05-aws-deploy/ai-train-aws-quick-ref-01-validation.md) | 배포 후 체크리스트 |
| [콘솔 참조 노트](../05-aws-deploy/ai-train-aws-cue-note-01-console.md) | AWS 콘솔 빠른 참조 |

---

## Part 7. 실전 서비스 만들기

앞에서 배운 모든 것을 연결해서 작동하는 서비스를 완성한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: 서비스 처음부터 만들기](../06-demo-services/ai-train-service-chapter-01-getting-started.md) | 메모 서비스 5개 함수 + 메뉴 루프 |
| 2 | [Chapter 02: Lambda에 배포하기](../06-demo-services/ai-train-service-chapter-02-deploy-to-lambda.md) | 터미널 서비스 → API 변환, URL 테스트 |
| 3 | [Chapter 03: 서비스에 테스트 추가하기](../06-demo-services/ai-train-service-chapter-03-testing.md) | assert, unittest, 테스트 파일 분리 |
| 4 | [Chapter 04: GitHub Actions로 자동 배포](../06-demo-services/ai-train-service-chapter-04-github-actions.md) | workflow 파일, Secrets, push 시 Lambda 자동 배포 |
| 2 | [기획 워크북](../06-demo-services/ai-train-service-workbook-01-planning.md) | 기능 정의, 구조 설계 |
| 3 | [구현 워크북](../06-demo-services/ai-train-service-workbook-02-implementation.md) | 단계별 구현 가이드 |

**예제 코드 시리즈**

| 파일 | 추가 기능 |
|------|---------|
| [service-code-01](../06-demo-services/ai-train-service-code-01.md) | 기본 함수 구조 |
| [service-code-02](../06-demo-services/ai-train-service-code-02.md) | 리스트 저장 |
| [service-code-03](../06-demo-services/ai-train-service-code-03.md) | 목록 출력 |
| [service-code-04](../06-demo-services/ai-train-service-code-04.md) | 삭제 기능 |
| [service-code-05](../06-demo-services/ai-train-service-code-05.md) | JSON 파일 저장 |
| [service-code-06](../06-demo-services/ai-train-service-code-06.md) | 검색 기능 |

---

## 설계 문서

| 파일 | 내용 |
|------|------|
| [커리큘럼 청사진](ai-train-curriculum-blueprint.md) | 전체 교육 목표와 원칙 |
| [Python 학습 지도](ai-train-curriculum-python-map.md) | Python 챕터 상세 안내 |
