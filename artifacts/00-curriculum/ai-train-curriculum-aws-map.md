# ai-train AWS 배포 학습 지도

## 목표

Python 코드를 AWS Lambda에 올려서 인터넷에서 접근 가능한 API로 만들고, 안전하게 운영하는 방법을 익힌다.

---

## 왜 AWS를 배우는가

내 컴퓨터에서만 실행되는 프로그램은 다른 사람이 사용할 수 없다. AWS Lambda를 사용하면:

- 서버를 직접 관리하지 않아도 된다
- 월 100만 요청까지 무료로 사용할 수 있다
- URL 하나로 전 세계 어디서든 내 코드를 호출할 수 있다

이 파트는 "내가 만든 서비스를 실제로 배포하는 경험"을 목표로 한다.

---

## 권장 학습 순서

### 0단계. 준비 확인

이 파트를 시작하기 전에 필요한 것:
- Python 함수를 작성하고 실행할 수 있는가
- GitHub에 코드를 push할 수 있는가
- 신용카드가 있는가 (AWS 가입 본인 확인용, 무료 티어 내 사용 시 청구 없음)

---

### 1단계. 배포 개념과 AWS 계정 준비

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 1 | [Chapter 01: 배포 개념과 검증](../05-aws-deploy/ai-train-aws-chapter-01-validation.md) | 배포란 무엇인가, 왜 하는가, 확인 방법 |
| 2 | [Chapter 02: AWS 계정과 Lambda](../05-aws-deploy/ai-train-aws-chapter-02-account-and-lambda-setup.md) | AWS 가입, MFA 설정, IAM 사용자, Lambda 첫 배포 |

**이 단계 마치면**: AWS 계정이 준비되고 Hello World Lambda 함수를 배포해서 테스트할 수 있다.

---

### 2단계. URL로 호출하고 로그 보기

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 3 | [Chapter 03: API Gateway로 URL 만들기](../05-aws-deploy/ai-train-aws-chapter-03-api-gateway.md) | Function URL, curl 테스트, Python 코드로 API 호출 |
| 4 | [Chapter 04: 로깅과 모니터링](../05-aws-deploy/ai-train-aws-chapter-04-logging-and-monitoring.md) | CloudWatch Logs, 로그 패턴, 오류 추적 |

**이 단계 마치면**: URL로 Lambda를 호출하고, 로그를 보며 문제를 진단할 수 있다.

---

### 3단계. 보안과 운영

| 순서 | 파일 | 핵심 내용 |
|------|------|----------|
| 5 | [Chapter 05: 환경 변수와 보안 설정](../05-aws-deploy/ai-train-aws-chapter-05-env-and-security.md) | `os.environ`, `.gitignore`, Secrets Manager, IAM 최소 권한 |
| 6 | [Chapter 06: 배포 런북과 운영 체크리스트](../05-aws-deploy/ai-train-aws-chapter-06-deployment-runbook.md) | 배포 전후 체크리스트, 롤백 방법, 주간 운영 점검 |

**이 단계 마치면**: 보안 설정이 된 서비스를 안전하게 배포하고 운영할 수 있다.

---

### 참고 자료

| 파일 | 내용 |
|------|------|
| [검증 실습](../05-aws-deploy/ai-train-aws-practice-01-validation.md) | Lambda 테스트 이벤트 만들기, 로그 확인 실습 |
| [검증 한 페이지 요약](../05-aws-deploy/ai-train-aws-quick-ref-01-validation.md) | 배포 후 확인 항목 빠른 참조 |
| [콘솔 참조 노트](../05-aws-deploy/ai-train-aws-cue-note-01-console.md) | AWS 콘솔 주요 메뉴 위치 |

---

## 학습 팁

- **MFA 설정은 필수**: AWS 계정 탈취 시 요금 폭탄이 발생한다. 계정 만들자마자 MFA를 설정한다
- **루트 계정은 쓰지 말자**: IAM 사용자를 만들어서 일상 작업에 사용한다
- **무료 티어 모니터링**: AWS 콘솔에서 Billing 알림을 설정하면 예상치 못한 요금을 예방할 수 있다
- **Deploy 버튼을 빠뜨리지 말자**: Lambda에서 코드를 수정하면 Deploy를 눌러야 반영된다. 가장 흔한 실수다

## 자주 막히는 지점

| 단계 | 막히는 부분 | 확인할 것 |
|------|-----------|----------|
| 계정 가입 | 신용카드 오류 | 해외 결제 가능 카드인지 확인 |
| Lambda 배포 | 함수 이름 없음 | `lambda_function.lambda_handler` 형식 확인 |
| URL 호출 | 오류 응답 | CloudWatch Logs에서 오류 메시지 확인 |
| 환경 변수 | 코드에서 읽기 실패 | `os.environ.get("KEY")` 형식 확인 |
| 비용 발생 | 예상치 못한 청구 | 리전 확인, 불필요한 리소스 삭제 |

## 무료 티어 범위 (2026년 기준)

| 서비스 | 무료 범위 |
|--------|----------|
| Lambda | 월 100만 요청, 400,000 GB-초 |
| API Gateway HTTP API | 월 100만 요청 (첫 12개월) |
| CloudWatch Logs | 월 5GB 수집 |

실습 수준의 사용량은 무료 범위를 초과하지 않는다.
