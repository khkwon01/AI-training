# Chapter 06: 배포 런북과 운영 체크리스트

## 이 장에서 배우는 것

- 런북(runbook)이 무엇인지, 왜 필요한지
- Lambda 함수 배포 전후 확인 항목
- 문제가 생겼을 때 빠르게 원인을 찾는 방법
- 롤백(rollback)이 무엇이고 어떻게 하는지
- 운영 중 주기적으로 확인해야 할 것들

---

## 먼저 쉬운 설명

코드를 배포하면 항상 잘 될까? 그렇지 않다.

배포 후 예상치 못한 오류가 생기거나, 이전 버전이 더 나을 때 빠르게 되돌려야 한다.

**런북(runbook)**은 "이 서비스를 어떻게 배포하고, 문제가 생기면 어떻게 대응한다"를 미리 정리해둔 문서다.

처음엔 간단한 체크리스트로 시작해도 충분하다.

---

## 1. 배포 전 체크리스트

코드를 Lambda에 올리기 전에 확인한다.

```
□ 로컬에서 테스트가 모두 통과했는가
□ 환경 변수가 올바르게 설정됐는가
□ 코드에 API 키, 비밀번호가 하드코딩되어 있지 않은가
□ GitHub에 최신 코드가 push됐는가
□ 어떤 기능이 추가/변경/삭제됐는지 정리했는가
□ 롤백이 필요하면 이전 버전으로 돌아갈 수 있는가
```

---

## 2. 배포 절차

### Step 1. 현재 버전 기록

Lambda → **Versions** 탭에서 현재 버전 번호 기록.

버전이 없다면 배포 전에 **Publish new version** 으로 스냅샷을 만든다.

```
배포 전 버전: 3  ← 문제 생기면 이 버전으로 돌아옴
```

### Step 2. 코드 업로드

Lambda 편집기에서 코드 붙여넣기 → **Deploy**

### Step 3. 배포 후 즉시 테스트

배포 직후 아래를 바로 확인한다.

```bash
# 헬스 체크 (기본 동작 확인)
curl "https://xxxxx.lambda-url.ap-northeast-2.on.aws/"

# 주요 기능 테스트
curl -X POST "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos?text=배포+테스트"
curl "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos"
```

---

## 3. 배포 후 체크리스트

```
□ 기본 URL 접속이 되는가
□ 주요 기능(추가/조회/삭제)이 정상 동작하는가
□ CloudWatch Logs에 오류가 없는가
□ 응답 시간이 이전과 비슷한가 (Monitor 탭 Duration 확인)
□ 오류율이 0%인가 (Monitor 탭 Errors 확인)
```

---

## 4. 문제 발생 시 대응 절차

### 빠른 진단

```
1. CloudWatch Logs 확인
   → [ERROR] 줄이 있는가?
   → 오류 메시지와 traceback 복사

2. Lambda Monitor 탭 확인
   → Errors 그래프가 올라가는가?
   → Duration이 급증했는가?

3. 최근 변경사항 확인
   → 방금 배포한 코드에서 무엇이 바뀌었는가?
```

### 오류 유형별 대응

| 오류 | 원인 가능성 | 대응 |
|------|-----------|------|
| `500 Internal Server Error` | 코드 오류 | CloudWatch Logs에서 traceback 확인 |
| `Task timed out` | 실행 시간 초과 | 타임아웃 설정 늘리거나 코드 최적화 |
| `Unable to import module` | 패키지 누락 | 배포 패키지에 의존성 포함 여부 확인 |
| `AccessDeniedException` | IAM 권한 부족 | 실행 역할에 필요한 권한 추가 |

---

## 5. 롤백 (Rollback)

배포 후 문제가 생기면 이전 버전으로 빠르게 돌아간다.

### Lambda 버전으로 롤백

```
Lambda → Versions 탭 → 이전 버전 클릭 → 코드 확인
→ 해당 버전 코드를 $LATEST에 다시 배포
```

### GitHub에서 이전 코드 가져오기

```bash
# 이전 commit으로 코드 복원
git log --oneline   # commit 목록 확인
git show abc1234:lambda_function.py > lambda_function.py  # 특정 commit의 파일 추출
```

그 다음 Lambda 편집기에 붙여넣고 Deploy.

---

## 6. 주간 운영 체크리스트

서비스를 운영하면서 주기적으로 확인한다.

```
□ CloudWatch Logs에 반복되는 오류가 없는가
□ Lambda 실행 시간이 타임아웃(15분)의 50% 이상 사용하는 케이스가 없는가
□ 에러율이 1% 이하인가
□ 불필요한 Lambda 호출이 없는가 (비용 최적화)
□ IAM 권한이 여전히 최소 수준인가
□ 사용하지 않는 Lambda 함수는 없는가
```

---

## 7. 메모 서비스 배포 런북 예시

실제 서비스에서 쓸 수 있는 런북 템플릿이다.

```markdown
# memo-service 배포 런북

## 서비스 정보
- 함수명: memo-service
- 리전: ap-northeast-2 (서울)
- Runtime: Python 3.12
- URL: https://xxxxx.lambda-url.ap-northeast-2.on.aws

## 배포 절차
1. 로컬 테스트 통과 확인
2. GitHub main에 push
3. Lambda 편집기에 코드 붙여넣기
4. Deploy 클릭
5. 헬스 체크: curl https://xxxxx.../memos
6. CloudWatch Logs에서 오류 없음 확인

## 롤백 절차
1. GitHub에서 이전 commit의 lambda_function.py 코드 복사
2. Lambda 편집기에 붙여넣기
3. Deploy 클릭
4. 헬스 체크 재실행

## 주요 연락처
- 담당자: (이름)
- GitHub 저장소: https://github.com/username/memo-service
```

---

## 따라 하기 실습

### 실습 1. 내 서비스 런북 만들기

`memo-service` 저장소에 `RUNBOOK.md` 파일을 만들고 위 템플릿을 채운다.

```bash
git add RUNBOOK.md
git commit -m "docs: 배포 런북 추가"
git push
```

### 실습 2. 의도적인 오류 배포 후 롤백 연습

1. Lambda 코드에 의도적인 오류 추가

```python
def lambda_handler(event, context):
    raise ValueError("의도적인 오류 - 롤백 테스트")
```

2. Deploy 후 URL 호출 → 500 오류 확인
3. CloudWatch Logs에서 traceback 확인
4. 이전 코드로 복원 후 Deploy
5. URL 재호출 → 정상 동작 확인

---

## 확인 체크리스트

- [ ] 배포 전후 체크리스트를 따를 수 있는가
- [ ] CloudWatch Logs에서 오류 원인을 찾을 수 있는가
- [ ] Lambda를 이전 버전으로 롤백하는 방법을 설명할 수 있는가
- [ ] 내 서비스의 런북을 작성할 수 있는가

---

## 참고 자료

- AWS Lambda 버전 관리 — https://docs.aws.amazon.com/lambda/latest/dg/configuration-versions.html
- AWS Lambda 모니터링 — https://docs.aws.amazon.com/lambda/latest/dg/monitoring-metrics.html
