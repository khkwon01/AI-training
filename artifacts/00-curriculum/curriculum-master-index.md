# AI-training 전체 커리큘럼 마스터 인덱스

## 목표

코딩을 처음 시작하는 초보자가 Python 기초부터 AI 협업, GitHub, AWS 배포까지 단계별로 따라갈 수 있는 전체 학습 경로입니다.

---

## 전체 학습 경로

```
Part 1. 환경 준비 (01-python)
  ↓
Part 2. Python 기초 (01-python, Chapter 01~16)
  ↓
Part 2+. Python 심화 (01-python, Chapter 17~) ← 선택 수강
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

> **기초 과정**: Part 1 → Part 2 → Part 3 → Part 4 → Part 5 → Part 6 → Part 7  
> **심화 선택**: Part 2+는 Part 2 완료 후 Python을 더 깊이 배우고 싶은 사람이 이수한다

---

## Part 1. 환경 준비

VS Code와 Python을 설치하고 코딩할 준비를 한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [VS Code 설치와 기본 사용](../01-python/python-vscode-chapter-01.md) | VS Code 설치, 폴더 열기, 터미널, Python 확장 |
| 2 | [Python 설치와 VS Code 설정](../01-python/python-vscode-chapter-02-python-setup.md) | Python 설치, 인터프리터 선택, venv 연결 |
| 실습 | [환경 설정 실습](../01-python/python-practice-setup-01.md) | 설치 확인 및 첫 실행 |
| 3 | [VS Code Chapter 03: ruff 린터](../01-python/python-vscode-chapter-03-ruff-linter.md) | ruff 설치·설정, 코드 자동 교정 |
| 참조 | [Python 치트시트](../01-python/python-quick-ref-cheat-sheet.md) | 기초 문법 빠른 참조 |

---

## Part 2. Python 기초 (Chapter 01~16)

Python 언어의 핵심 개념을 순서대로 익힌다. **이 파트를 마치면 Part 3으로 넘어간다.**

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: print와 실행](../01-python/python-chapter-01-print-and-run.md) | print, 파일 실행, 따옴표 |
| 2 | [Chapter 02: 변수](../01-python/python-chapter-02-variables.md) | 변수, 이름 짓기, 값 변경 |
| 3 | [Chapter 03: 자료형](../01-python/python-chapter-03-data-types.md) | int, float, str, bool, f-string |
| 4 | [Chapter 04: 파일과 JSON](../01-python/python-chapter-04-files-and-json.md) | 파일 읽기/쓰기, JSON 저장 |
| 5 | [Chapter 05: 디버깅](../01-python/python-chapter-05-debugging.md) | 오류 메시지 읽기, print 디버깅 |
| 6 | [Chapter 06: 프로젝트 구조](../01-python/python-chapter-06-project-structure.md) | 폴더 구조, 이름 규칙, import |
| 7 | [Chapter 07: 실행 환경](../01-python/python-chapter-07-environment.md) | 환경변수, dependency, path |
| 8 | [Chapter 08: 예외 처리](../01-python/python-chapter-08-exceptions.md) | try/except, 예외 종류 |
| 9 | [Chapter 09: venv와 pip](../01-python/python-chapter-09-venv-and-pip.md) | venv, pip, requirements.txt |
| 10 | [Chapter 10: comprehension과 datetime](../01-python/python-chapter-10-comprehension-datetime.md) | 리스트 컴프리헨션, 날짜 다루기 |
| 11 | [Chapter 11: 조건문](../01-python/python-chapter-11-conditions.md) | if/elif/else, 비교 연산자, 논리 연산자 |
| 12 | [Chapter 12: 반복문](../01-python/python-chapter-12-loops.md) | for, while, range, break/continue |
| 13 | [Chapter 13: 함수](../01-python/python-chapter-13-functions.md) | def, 매개변수, return, 기본값 |
| 14 | [Chapter 14: class와 object](../01-python/python-chapter-14-class-and-object.md) | class, __init__, self, 메서드 |
| 15 | [Chapter 15: module과 import](../01-python/python-chapter-15-module-and-import.md) | import, from, 표준 라이브러리, __name__ |
| 16 | [Chapter 16: 터미널/실행버튼/디버그 비교](../01-python/python-chapter-16-run-debug-compare.md) | 3가지 실행 방법, 중단점, F10 단계 실행 |

**기초 연습 자료**

| 파일 | 내용 |
|------|------|
| [venv/pip 실습](../01-python/python-practice-venv-pip-01.md) | venv 만들기, 패키지 설치 실습 |
| [퀵 체크 1](../01-python/python-quick-check-01.md) | Chapter 01~05 핵심 확인 문제 |
| [퀵 체크 2](../01-python/python-quick-check-02.md) | Chapter 06~10 핵심 확인 문제 |
| [퀵 체크 3](../01-python/python-quick-check-03.md) | Chapter 11~15 핵심 확인 문제 |
| [퀵 체크 1~2 정답](../01-python/python-quick-check-answers.md) | 퀵 체크 1~2 정답 및 해설 |
| [퀵 체크 3 정답](../01-python/python-quick-check-03-answers.md) | 퀵 체크 3 정답 및 해설 |
| [누적 실습 01](../01-python/python-practice-cumulative-01.md) | Chapter 01~10 종합 실습 |
| [누적 실습 01 정답](../01-python/python-practice-cumulative-01-answers.md) | 누적 실습 01 정답 |
| [누적 실습 02](../01-python/python-practice-cumulative-02.md) | Chapter 11~15 연결 실습 (학생 성적 관리) |
| [조건문+반복문+함수 실습](../01-python/python-practice-conditions-loops-functions.md) | 5단계 점진적 문제 세트 (소수, FizzBuzz, 통계 등) |
| [AI와 첫 코딩 연결 실습](../01-python/python-practice-ai-first-coding.md) | Python 기초 마무리 — AI에게 코드 요청하고 에러 전달하는 첫 루프 경험 |

---

## Part 2+. Python 심화 (Chapter 17~) ← 선택 수강

> **선택 과정**: Part 2를 마친 후 Python을 더 깊이 익히고 싶은 사람이 수강한다.  
> 기초 과정(Part 3~7)을 먼저 진행해도 무방하다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 17 | [Chapter 17: CLI와 argparse](../01-python/python-chapter-17-cli-argparse.md) | argparse, sys.argv, 명령줄 도구 만들기 |
| 18 | [Chapter 18: pytest 테스팅](../01-python/python-chapter-18-pytest.md) | pytest, 픽스처, 파라미터화 테스트 |
| 19 | [Chapter 19: 타입 힌트와 docstring](../01-python/python-chapter-19-type-hints-docstring.md) | type hints, docstring, mypy |
| 20 | [Chapter 20: 실전 에러 패턴](../01-python/python-chapter-20-error-patterns.md) | 실무 예외 처리 패턴 |
| 21 | [Chapter 21: 팀 venv 관리](../01-python/python-chapter-21-venv-team.md) | 팀 환경 관리, pyproject.toml |
| 22 | [Chapter 22: pydantic 데이터 검증](../01-python/python-chapter-22-pydantic.md) | pydantic, 데이터 모델, 검증 |
| 23 | [Chapter 23: async 기초](../01-python/python-chapter-23-async-basics.md) | async/await, httpx, API 호출 |
| 24 | [Chapter 24: 패키징](../01-python/python-chapter-24-packaging.md) | pyproject.toml, 패키지 배포 |
| 25 | [Chapter 25: logging](../01-python/python-chapter-25-logging.md) | logging 모듈, 프로덕션 로깅 |
| 26 | [Chapter 26: requests HTTP](../01-python/python-chapter-26-requests.md) | requests, HTTP 패턴 |
| 27 | [Chapter 27: 설정 관리](../01-python/python-chapter-27-config-management.md) | 환경변수, .env, 설정 패턴 |
| 28 | [Chapter 28: 문자열 템플릿](../01-python/python-chapter-28-string-templates.md) | f-string, jinja2, 포맷 패턴 |
| 29 | [Chapter 29: datetime과 timezone](../01-python/python-chapter-29-datetime-timezone.md) | datetime, timezone, 날짜 연산 |
| 30 | [Chapter 30: pathlib 파일 시스템](../01-python/python-chapter-30-pathlib.md) | pathlib, 파일/디렉토리 다루기 |
| 31 | [Chapter 31: generators와 iterators](../01-python/python-chapter-31-generators.md) | generator, iterator, yield |
| 32 | [Chapter 32: context managers](../01-python/python-chapter-32-context-managers.md) | with, __enter__/__exit__ |
| 33 | [Chapter 33: decorators](../01-python/python-chapter-33-decorators.md) | @decorator, functools, 실전 패턴 |
| 34 | [Chapter 34: dataclasses](../01-python/python-chapter-34-dataclasses.md) | @dataclass, field, 비교/정렬 |

**심화 연습 자료**

| 파일 | 내용 |
|------|------|
| [class+module 실습](../01-python/python-practice-class-and-module.md) | 도서관·은행 시스템 구현 + 모듈 분리 연습 |

---

## Part 3. 코드 저장소 관리 (GitHub)

코드를 GitHub에 저장하고 VS Code와 연결한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: Git과 GitHub 시작](../02-github/github-chapter-01-git-and-github.md) | Git 설치, GitHub 계정, clone |
| 2 | [Chapter 02: add, commit, push](../02-github/github-chapter-02-add-commit-push.md) | 변경 기록하고 올리기 |
| 3 | [Chapter 03: AI review 워크플로우](../02-github/github-chapter-03-ai-review-workflow.md) | PR, AI review, 협업 흐름 |
| 4 | [Chapter 04: VS Code와 GitHub 연결](../02-github/github-chapter-04-vscode-github-connect.md) | Source Control UI, 계정 연결 |
| 5 | [Chapter 05: branch와 Pull Request](../02-github/github-chapter-05-branch-and-pr.md) | branch 생성, PR 만들기, merge |
| 6 | [Chapter 06: merge conflict 해결](../02-github/github-chapter-06-merge-conflict.md) | conflict 표시 읽기, VS Code로 해결 |
| 7 | [Chapter 07: issue 트래킹](../02-github/github-chapter-07-issue-tracking.md) | issue 생성, branch 연결, 자동 close |
| 8 | [Chapter 08: PR 리뷰 가이드](../02-github/github-chapter-08-pr-review-guide.md) | Files changed 읽기, 줄 코멘트, AI 코드 체크리스트 |

**실습 자료**

| 파일 | 내용 |
|------|------|
| [브라우저 따라하기](../02-github/github-walkthrough-01-browser.md) | Issue → PR 흐름 실습 |
| [branch 실습 워크시트](../02-github/github-practice-01-branch-workflow.md) | branch 생성~merge 단계별 실습 |
| [Chapter 09: CI 파이프라인](../02-github/github-chapter-09-ci-pipeline.md) | GitHub Actions CI |
| [Chapter 10: 저장소 설정](../02-github/github-chapter-10-repo-settings.md) | branch protection |
| [Chapter 11: commit 컨벤션](../02-github/github-chapter-11-commit-conventions.md) | conventional commits |
| [Chapter 12: Actions Secrets](../02-github/github-chapter-12-actions-secrets.md) | Secrets 관리 |
| [Chapter 13: solo vs 팀 워크플로우](../02-github/github-chapter-13-workflow-solo-vs-team.md) | Git 전략 비교 |
| [Chapter 14: Copilot Workspace](../02-github/github-chapter-14-copilot-workspace.md) | Copilot 팀 설정 |
| [Chapter 18: Claude Code로 PR 만들기](../02-github/github-chapter-18-claude-code-pr-workflow.md) | Claude Code 터미널 → 브랜치 → diff 검토 → PR 생성 실전 흐름 |
| [Chapter 19: 보안 스캐닝](../02-github/github-chapter-19-security-scanning.md) | GitHub Advanced Security, CodeQL, 코드 스캐닝 자동화 |
| [워크북](../02-github/github-workbook-01.md) | GitHub 실습 워크북 |
| [AI review 랩](../02-github/github-lab-01-ai-review.md) | AI review 실습 |
| [화면 참조 노트](../02-github/github-cue-note-01-screen.md) | 화면 위치 빠른 참조 |
| [명령어 빠른 참조](../02-github/github-quick-ref-commands.md) | 매일 쓰는 Git 명령어 + PR 체크리스트 |

---

## Part 4. AI 도구 연결

VS Code에 AI를 연결하고 AI 활용 기본 습관을 익힌다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: AI 워크플로우와 검증](../03-ai-basics/ai-chapter-01-workflow-and-verification.md) | AI 활용 원칙, 검증 습관 |
| 2 | [Chapter 02: AI 초안 수정](../03-ai-basics/ai-chapter-02-draft-correction.md) | AI 코드 읽고 수정하기 |
| 3 | [Chapter 03: VS Code AI 도구 설치](../03-ai-basics/ai-chapter-03-vscode-ai-setup.md) | Copilot, Codeium, Claude 연결 |
| 4 | [Chapter 04: 좋은 프롬프트 작성법](../03-ai-basics/ai-chapter-04-prompt-writing.md) | 역할/목표/제약/형식, 패턴별 프롬프트 |
| 5 | [Chapter 05: AI 코드 검증 체크리스트](../03-ai-basics/ai-chapter-05-verification-checklist.md) | 엣지 케이스 테스트, assert, 자주 나타나는 문제 패턴 |
| 6 | [Chapter 06: AI 환각 감지 체크리스트](../03-ai-basics/ai-chapter-06-hallucination-checklist.md) | 환각 유형, 공식 문서 대조, assert 검증 |
| 7 | [Chapter 07: Claude Code 입문](../03-ai-basics/ai-chapter-07-claude-code-intro.md) | 설치, 파일 직접 수정, 일반 채팅과 차이, 안전한 사용 |
| 8 | [Chapter 08: 직접 배울 것 vs AI 위임](../03-ai-basics/ai-chapter-08-teach-vs-delegate.md) | 경계선 판단 기준, 에러 직접 읽기, AI 코드 주석 달기 |
| 9 | [Chapter 09: AI 도구 선택 가이드](../03-ai-basics/ai-chapter-09-tool-selection-guide.md) | Claude vs Copilot vs Cursor 비교, 상황별 선택법 |
| 10 | [Chapter 10: AI 보안 기초](../03-ai-basics/ai-chapter-10-ai-security-basics.md) | Prompt Injection, 사용자 입력 검증, 안전한 AI 사용 |
| 11 | [Chapter 11: 역할별 프롬프트 패턴](../03-ai-basics/ai-chapter-11-role-prompt-patterns.md) | Planner·Implementer·Reviewer 역할 프롬프트 실전 패턴 |

**실습**

| 파일 | 내용 |
|------|------|
| [Chapter 12: AI 페어 프로그래밍](../03-ai-basics/ai-chapter-12-pair-programming.md) | AI 코딩 워크플로우 |
| [Chapter 13: AI 코드리뷰 자동화](../03-ai-basics/ai-chapter-13-ai-code-review-automation.md) | Actions + AI 리뷰 |
| [Chapter 14: AI 디버깅](../03-ai-basics/ai-chapter-14-ai-debugging.md) | AI로 에러 수정 |
| [Chapter 15: 모델 선택 가이드](../03-ai-basics/ai-chapter-15-model-selection.md) | 태스크별 모델 |
| [Chapter 16: 아키텍처 프롬프트](../03-ai-basics/ai-chapter-16-architecture-prompts.md) | 설계 프롬프트 |
| [Chapter 17: 컨텍스트 관리](../03-ai-basics/ai-chapter-17-context-management.md) | 대형 코드베이스 |
| [Chapter 18: AI 메모리 패턴](../03-ai-basics/ai-chapter-18-memory-patterns.md) | 세션 메모리 |
| [Chapter 19: AI 버그 수정](../03-ai-basics/ai-chapter-19-fixing-ai-bugs.md) | AI 버그 읽고 고치기 |
| [Chapter 20: AI 문서화 자동화](../03-ai-basics/ai-chapter-20-documentation-automation.md) | README, docstring, API 문서 자동 생성 |
| [Chapter 22: AI 코드 품질 루브릭](../03-ai-basics/ai-chapter-22-code-quality-rubric.md) | AI 코드 평가 5기준 체크리스트 |
| [초안 수정 랩](../03-ai-basics/ai-lab-01-draft-correction.md) | AI 초안 수정 실습 |

---

## Part 5. AI와 협업 코딩 (Vibe Coding)

AI와 함께 기능을 만드는 방법을 익힌다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: Vibe coding 이해](../04-vibe-coding/vibe-chapter-01-vibe-coding.md) | vibe coding 개념, 기본 원칙 |
| 2 | [Chapter 02: AI 코딩 루프](../04-vibe-coding/vibe-chapter-02-ai-coding-loop.md) | 요청→확인→수정 루프 실습 |
| 3 | [Chapter 03: AI 코드 수정 루프](../04-vibe-coding/vibe-chapter-03-repair-loop.md) | 오류→분석→수정→검증 반복 패턴 |
| 4 | [Chapter 04: 구현부터 배포까지 전체 워크플로우](../04-vibe-coding/vibe-chapter-04-full-workflow.md) | issue→branch→AI코딩→PR→Lambda배포 전체 실습 |
| 5 | [Chapter 05: 프로젝트 기반 Vibe Coding](../04-vibe-coding/vibe-chapter-05-project-based-vibe-coding.md) | AI와 함께 미니 서비스 처음부터 끝까지 만들기 |
| 6 | [Chapter 06: 실전 프롬프트 패턴](../04-vibe-coding/vibe-chapter-06-advanced-prompts.md) | 도구별 프롬프트 최적화, 개선 3단계 |
| 7 | [Chapter 07: AI 도구로 서비스 전체 만들기](../04-vibe-coding/vibe-chapter-07-full-service-with-ai.md) | 기획→코딩→GitHub→Lambda배포 완성 시나리오 |
| 8 | [Chapter 08: CLAUDE.md와 Agentic Coding 안전 사용법](../04-vibe-coding/vibe-chapter-08-claude-md-and-agentic-safety.md) | CLAUDE.md 작성 실습, AI 자율 수정 위험성, diff 검토 체크리스트 |

**실습 자료**

| 파일 | 내용 |
|------|------|
| [프롬프트 세트 01](../04-vibe-coding/vibe-prompt-set-01.md) | 기획·기능 분해 프롬프트 |
| [프롬프트 세트 02](../04-vibe-coding/vibe-prompt-set-02.md) | 구현·리뷰·디버깅 프롬프트 |
| [워크북](../04-vibe-coding/vibe-workbook-01.md) | Vibe coding 실습 워크북 |
| [통합 워크시트](../04-vibe-coding/vibe-worksheet-02-full-integration.md) | GitHub+AWS+DynamoDB 전체 연결 통합 워크시트 |

---

## Part 6. AWS 배포

Python 코드를 AWS Lambda에 올리고 검증한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: 배포 개념과 검증](../05-aws-deploy/aws-chapter-01-validation.md) | 배포란 무엇인가, 확인 방법 |
| 2 | [Chapter 02: AWS 계정과 Lambda](../05-aws-deploy/aws-chapter-02-account-and-lambda-setup.md) | 계정 생성, IAM, Lambda 첫 배포 |
| 3 | [Chapter 03: API Gateway로 URL 만들기](../05-aws-deploy/aws-chapter-03-api-gateway.md) | Function URL, API Gateway, curl 테스트 |
| 4 | [Chapter 04: 로깅과 모니터링](../05-aws-deploy/aws-chapter-04-logging-and-monitoring.md) | CloudWatch Logs, 로그 패턴, 오류 추적 |
| 5 | [Chapter 05: 환경 변수와 보안 설정](../05-aws-deploy/aws-chapter-05-env-and-security.md) | os.environ, Secrets Manager, IAM 최소 권한 |
| 6 | [Chapter 06: 배포 런북과 운영 체크리스트](../05-aws-deploy/aws-chapter-06-deployment-runbook.md) | 배포 전후 체크리스트, 롤백, 주간 운영 점검 |
| 7 | [Chapter 07: DynamoDB로 데이터 영구 저장](../05-aws-deploy/aws-chapter-07-dynamodb.md) | DynamoDB 생성, boto3 CRUD, Lambda 재시작 후에도 데이터 유지 |

**참고 자료**

| 파일 | 내용 |
|------|------|
| [Chapter 08: S3 정적 호스팅](../05-aws-deploy/aws-chapter-08-s3-static-hosting.md) | S3 웹호스팅 |
| [Chapter 09: Lambda Layers](../05-aws-deploy/aws-chapter-09-lambda-layers.md) | 의존성 Layers |
| [Chapter 10: API Gateway 고급](../05-aws-deploy/aws-chapter-10-api-gateway-advanced.md) | 고급 라우팅 |
| [Chapter 11: 비용 최적화](../05-aws-deploy/aws-chapter-11-cost-optimization.md) | 서버리스 비용 |
| [Chapter 12: SQS 기초](../05-aws-deploy/aws-chapter-12-sqs-basics.md) | 메시지 큐 |
| [Chapter 13: CloudFormation](../05-aws-deploy/aws-chapter-13-cloudformation.md) | IaC 기초 |
| [Chapter 18: Function URL 우선 배포 가이드](../05-aws-deploy/aws-chapter-18-function-url-guide.md) | Function URL vs API Gateway 비교, 초보자 첫 배포 권장 방법 |
| [검증 실습](../05-aws-deploy/aws-practice-01-validation.md) | Lambda 테스트 및 로그 확인 |
| [검증 한 페이지 요약](../05-aws-deploy/aws-quick-ref-01-validation.md) | 배포 후 체크리스트 |
| [콘솔 참조 노트](../05-aws-deploy/aws-cue-note-01-console.md) | AWS 콘솔 빠른 참조 |

---

## Part 7. 실전 서비스 만들기

앞에서 배운 모든 것을 연결해서 작동하는 서비스를 완성한다.

| 순서 | 파일 | 내용 |
|------|------|------|
| 1 | [Chapter 01: 서비스 처음부터 만들기](../06-demo-services/service-chapter-01-getting-started.md) | 메모 서비스 5개 함수 + 메뉴 루프 |
| 2 | [Chapter 02: Lambda에 배포하기](../06-demo-services/service-chapter-02-deploy-to-lambda.md) | 터미널 서비스 → API 변환, URL 테스트 |
| 3 | [Chapter 03: 서비스에 테스트 추가하기](../06-demo-services/service-chapter-03-testing.md) | assert, unittest, 테스트 파일 분리 |
| 4 | [Chapter 04: GitHub Actions로 자동 배포](../06-demo-services/service-chapter-04-github-actions.md) | workflow 파일, Secrets, push 시 Lambda 자동 배포 |
| 5 | [Chapter 05: 완성형 서비스 통합 실습](../06-demo-services/service-chapter-05-full-integration.md) | DynamoDB+Lambda+GitHub Actions+CloudWatch 전체 연결 |
| 6 | [Chapter 06: Issue부터 배포까지](../06-demo-services/service-chapter-06-issue-to-deploy.md) | Issue→AI 코딩→PR→자동 배포 전체 실전 흐름 |
| 2 | [Chapter 07: 아키텍처 개요](../06-demo-services/service-chapter-07-architecture.md) | 풀스택 구조 |
| [Chapter 08: REST API 설계](../06-demo-services/service-chapter-08-rest-api-design.md) | REST 원칙 |
| [Chapter 09: AI 리팩토링](../06-demo-services/service-chapter-09-ai-refactoring.md) | 코드 품질 개선 |
| [Chapter 10: 마이크로서비스 vs 모놀리스](../06-demo-services/service-chapter-10-microservices-vs-monolith.md) | 아키텍처 선택 |
| [Chapter 11: E2E 테스팅](../06-demo-services/service-chapter-11-e2e-testing.md) | 통합 테스트 |
| [Chapter 12: 헬스 체크](../06-demo-services/service-chapter-12-health-check.md) | 모니터링 |
| [Chapter 13: DB 설계](../06-demo-services/service-chapter-13-database-design.md) | 데이터베이스 기초 |
| [Chapter 14: Feature Flag](../06-demo-services/service-chapter-14-feature-flags.md) | 기능 플래그 |
| [Chapter 15: API 버저닝](../06-demo-services/service-chapter-15-api-versioning.md) | 버전 관리 전략 |
| [Chapter 16: 캐싱 전략](../06-demo-services/service-chapter-16-caching-strategy.md) | Lambda 캐싱, cold start 최적화 |
| [기획 워크북](../06-demo-services/service-workbook-01-planning.md) | 기능 정의, 구조 설계 |
| 3 | [구현 워크북](../06-demo-services/service-workbook-02-implementation.md) | 단계별 구현 가이드 |

**예제 코드 시리즈**

| 파일 | 추가 기능 |
|------|---------|
| [service-code-01](../06-demo-services/service-code-01.md) | 기본 함수 구조 |
| [service-code-02](../06-demo-services/service-code-02.md) | 리스트 저장 |
| [service-code-03](../06-demo-services/service-code-03.md) | 목록 출력 |
| [service-code-04](../06-demo-services/service-code-04.md) | 삭제 기능 |
| [service-code-05](../06-demo-services/service-code-05.md) | JSON 파일 저장 |
| [service-code-06](../06-demo-services/service-code-06.md) | 검색 기능 |

---

## 설계 문서

### 커리큘럼 전체 설계

| 파일 | 내용 |
|------|------|
| [커리큘럼 청사진](curriculum-blueprint.md) | 전체 교육 목표, 대상, 원칙 |
| [E2E 통합 튜토리얼](curriculum-e2e-tutorial.md) | Python 서비스 → GitHub PR → AI 리뷰 → AWS Lambda 배포 전체 흐름 |

### 영역별 학습 지도

각 파트의 상세 순서, 학습 팁, 자주 막히는 지점을 담은 안내 문서다.

| 파일 | 해당 파트 | 내용 |
|------|----------|------|
| [Python 학습 지도](curriculum-python-map.md) | Part 1~2 | VS Code 설정 + Python 챕터 상세 순서 |
| [GitHub 학습 지도](curriculum-github-map.md) | Part 3 | Git 기초 → VS Code 연결 → branch/PR → AI 리뷰 |
| [AI 도구 학습 지도](curriculum-ai-map.md) | Part 4 | AI 원칙 → 도구 설치 → 프롬프트 → 검증/보안 |
| [Vibe Coding 학습 지도](curriculum-vibe-coding-map.md) | Part 5 | vibe coding 개념 → 루프 → 실전 워크플로우 |
| [AWS 배포 학습 지도](curriculum-aws-map.md) | Part 6 | 배포 개념 → Lambda → URL → 보안 → 운영 |
| [실전 서비스 학습 지도](curriculum-demo-services-map.md) | Part 7 | 터미널 서비스 → 테스트 → Lambda → 자동 배포 |
