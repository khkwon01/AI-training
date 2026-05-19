# 실전 서비스 학습 지도

## 목표

앞에서 배운 Python, GitHub, AI, AWS를 모두 연결해서 실제로 동작하는 서비스를 처음부터 배포까지 완성한다.

---

## 이 파트가 다른 파트와 다른 점

- Part 1~6은 각 기술을 따로 배웠다
- **이 파트는 모두 연결**한다

```
Python 코드 작성
    + GitHub 버전 관리
    + AI 도구로 기능 추가
    + 테스트로 검증
    + AWS Lambda 배포
    + GitHub Actions로 자동화
= 실제 서비스
```

---

## 권장 학습 순서

### 0단계. 준비 확인

이 파트를 시작하기 전에 필요한 것:
- Python으로 함수를 작성하고 파일로 저장할 수 있는가
- GitHub에 저장소를 만들고 push할 수 있는가
- AWS Lambda에 함수를 배포하고 URL로 호출할 수 있는가
- AI 도구(Copilot 또는 Claude)를 사용할 수 있는가

---

### 1단계. 터미널 서비스 만들기

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 1 | [Chapter 01: 서비스 처음부터 만들기](../06-demo-services/service-chapter-01-getting-started.md) | 메모 서비스 5개 함수(추가·조회·삭제·저장·검색) + 메뉴 루프 |
| - | [기획 워크북](../06-demo-services/service-workbook-01-planning.md) | 기능 정의, 구조 설계 워크북 |

**이 단계 마치면**: 터미널에서 동작하는 메모 서비스를 완성하고 GitHub에 올릴 수 있다.

---

### 2단계. 테스트 추가

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 2 | [Chapter 03: 서비스에 테스트 추가하기](../06-demo-services/service-chapter-03-testing.md) | assert, unittest, setUp, 테스트 파일 분리 |
| - | [구현 워크북](../06-demo-services/service-workbook-02-implementation.md) | 단계별 구현 워크북 |

**이 단계 마치면**: 코드가 기대대로 동작하는지 자동으로 확인하는 테스트를 작성할 수 있다.

---

### 3단계. Lambda API로 전환

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 3 | [Chapter 02: Lambda에 배포하기](../06-demo-services/service-chapter-02-deploy-to-lambda.md) | 터미널 서비스 → Lambda API 변환, URL로 테스트 |
| - | [Lambda 기초 개선판](../06-demo-services/service-chapter-02b-lambda-basics-refined.md) | Lambda 핵심 개념 집중 설명 (인지 부하 낮춤) |

**이 단계 마치면**: 메모 서비스를 URL로 호출할 수 있는 Lambda API로 만들 수 있다.

---

### 4단계. 자동 배포 설정

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 4 | [Chapter 04: GitHub Actions로 자동 배포](../06-demo-services/service-chapter-04-github-actions.md) | workflow 파일, GitHub Secrets, push 시 Lambda 자동 배포 |

**이 단계 마치면**: `git push`만 하면 Lambda가 자동으로 업데이트된다.

---

### 예제 코드 시리즈 (기능별 단계적 추가)

| 파일 | 추가된 기능 |
|------|-----------|
| [service-code-01](../06-demo-services/service-code-01.md) | 기본 함수 구조 |
| [service-code-02](../06-demo-services/service-code-02.md) | 리스트 저장 |
| [service-code-03](../06-demo-services/service-code-03.md) | 목록 출력 |
| [service-code-04](../06-demo-services/service-code-04.md) | 삭제 기능 |
| [service-code-05](../06-demo-services/service-code-05.md) | JSON 파일 저장 |
| [service-code-06](../06-demo-services/service-code-06.md) | 검색 기능 |

코드 01부터 순서대로 읽으면 기능이 하나씩 추가되는 과정을 볼 수 있다.

---

### 운영 참고 자료

| 파일 | 내용 |
|------|------|
| [진행 체크리스트](../06-demo-services/service-checklist-01-progress.md) | 단계별 완료 확인 |
| [구현 순서 노트](../06-demo-services/service-note-01-sequence.md) | 기능 구현 순서 참조 |

---

## 학습 팁

- **Chapter 01을 꼭 터미널에서 완성하자**: Lambda로 바로 가지 말고, 로컬에서 동작하는 서비스를 먼저 완성한다
- **테스트를 먼저 작성하는 습관**: 기능을 만들기 전에 "어떻게 동작해야 하는가"를 테스트로 먼저 쓰면 실수가 줄어든다
- **AI와 협업하자**: 새 기능을 추가할 때 AI에게 프롬프트로 초안을 받고, 검증하고, 테스트를 통과시킨다
- **commit을 자주**: 기능 하나 추가할 때마다 commit한다. 문제가 생겨도 되돌릴 수 있다

## 이 파트를 마치면

```
✅ 로컬에서 동작하는 Python 서비스
✅ GitHub에 코드 버전 관리
✅ 테스트로 기능 검증
✅ Lambda에 배포된 API
✅ GitHub Actions로 자동 배포
✅ CloudWatch로 로그 모니터링
```

이 모든 것을 혼자 완성할 수 있다.
