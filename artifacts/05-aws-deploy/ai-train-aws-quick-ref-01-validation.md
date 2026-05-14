## 이 장에서 배우는 것

- 배포 후 반드시 확인해야 할 AWS 서비스 상태 체크 방법
- 성공과 실패를 구분하는 기준과 지표
- 문제가 생겼을 때 가장 먼저 봐야 할 로그 위치
- 한 페이지짜리 validation sheet를 직접 만들고 활용하는 법
- 초보자도 바로 쓸 수 있는 AWS CLI 명령어 모음

---

## 먼저 쉬운 설명

배포를 눌렀다. 에러는 없었다. 그런데 정말 잘 된 걸까?

배포 성공과 서비스 정상 동작은 다릅니다. 코드가 올라갔어도 EC2가 꺼져 있을 수 있고, S3에 파일이 올라갔어도 권한이 잘못 설정되어 있을 수 있습니다.

이 장에서는 배포 직후 5분 안에 "정말 잘 됐나?" 를 빠르게 확인하는 체크리스트를 만듭니다. 마치 비행기 조종사가 이륙 전에 체크리스트를 읽듯이, 배포 후에도 같은 순서로 점검하는 습관을 만드는 것이 목표입니다.

---

## 1. AWS CLI로 서비스 상태 빠르게 확인하기

배포 후 가장 먼저 해야 할 것은 AWS CLI로 리소스 상태를 직접 눈으로 확인하는 것입니다.

```bash
# EC2 인스턴스 상태 확인
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=my-app-server" \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,PublicIpAddress]" \
  --output table

# 결과 예시 (정상)
# +-----------------------+----------+-----------------+
# |  i-0a1b2c3d4e5f6789  | running  | 54.123.45.67    |
# +-----------------------+----------+-----------------+

# 결과 예시 (비정상)
# +-----------------------+----------+-----------------+
# |  i-0a1b2c3d4e5f6789  | stopped  | None            |
# +-----------------------+----------+-----------------+
```

```bash
# ECS 서비스 상태 확인
aws ecs describe-services \
  --cluster my-cluster \
  --services my-app-service \
  --query "services[*].[serviceName,status,runningCount,desiredCount]" \
  --output table

# 정상: runningCount == desiredCount
# 비정상: runningCount < desiredCount (태스크가 계속 재시작되는 중)
```

**성공 기준:**
- EC2: `running` 상태
- ECS: `runningCount == desiredCount`
- Lambda: 함수 존재 + 최근 업데이트 시간 확인

---

## 2. HTTP 응답으로 애플리케이션 동작 확인하기

서버가 켜져 있어도 앱이 제대로 응답하지 않을 수 있습니다. `curl`로 직접 확인합니다.

```bash
# 헬스체크 엔드포인트 확인
curl -s -o /dev/null -w "%{http_code}" http://54.123.45.67:8080/health
# 기댓값: 200

# JSON 응답까지 확인하고 싶을 때
curl -s http://54.123.45.67:8080/health | python3 -m json.tool
# 정상 응답 예시:
# {
#   "status": "healthy",
#   "database": "connected",
#   "version": "1.2.3"
# }
```

```bash
# ALB(로드밸런서) DNS로 확인 (EC2 IP 직접 접근보다 이쪽이 실제 서비스 경로)
ALB_DNS="my-alb-123456789.ap-northeast-2.elb.amazonaws.com"
curl -s -o /dev/null -w "HTTP 상태코드: %{http_code}\n" http://$ALB_DNS/health

# 자주 보는 에러 코드
# 200 → 정상
# 502 Bad Gateway → 앱 서버가 응답 안 함 (ECS 태스크 죽었는지 확인)
# 503 Service Unavailable → 헬스체크 실패로 타깃 그룹에서 제외됨
# 504 Gateway Timeout → 앱이 너무 느리게 응답 중
```

---

## 3. CloudWatch 로그에서 에러 찾기

앱이 응답은 하는데 내부에서 에러가 나고 있을 수 있습니다. 로그를 봅니다.

```bash
# 최근 5분간 로그 확인 (ECS 기준)
aws logs filter-log-events \
  --log-group-name "/ecs/my-app-service" \
  --start-time $(date -d '5 minutes ago' +%s000) \
  --filter-pattern "ERROR" \
  --query "events[*].message" \
  --output text

# Lambda 로그 확인
aws logs filter-log-events \
  --log-group-name "/aws/lambda/my-function" \
  --start-time $(date -d '10 minutes ago' +%s000) \
  --filter-pattern "?ERROR ?Exception ?Traceback" \
  --output text
```

```bash
# 로그 스트림 이름을 모를 때: 가장 최근 스트림 자동으로 찾기
LOG_STREAM=$(aws logs describe-log-streams \
  --log-group-name "/ecs/my-app-service" \
  --order-by LastEventTime \
  --descending \
  --query "logStreams[0].logStreamName" \
  --output text)

aws logs get-log-events \
  --log-group-name "/ecs/my-app-service" \
  --log-stream-name "$LOG_STREAM" \
  --limit 50 \
  --query "events[*].message" \
  --output text
```

---

## 4. 배포 후 Validation Sheet 스크립트로 자동화하기

위 명령어들을 하나의 스크립트로 묶으면 매번 타이핑할 필요 없이 한 번에 실행할 수 있습니다.

```bash
#!/bin/bash
# 파일명: validate-deployment.sh

set -euo pipefail

APP_NAME="my-app"
CLUSTER="my-cluster"
SERVICE="my-app-service"
ALB_DNS="my-alb-123456789.ap-northeast-2.elb.amazonaws.com"
LOG_GROUP="/ecs/my-app-service"
REGION="ap-northeast-2"

echo "======================================"
echo "  배포 후 Validation 체크 시작"
echo "  시각: $(date '+%Y-%m-%d %H:%M:%S')"
echo "======================================"

# 1. ECS 서비스 상태
echo ""
echo "[1/4] ECS 서비스 상태 확인 중..."
RUNNING=$(aws ecs describe-services \
  --cluster $CLUSTER --services $SERVICE --region $REGION \
  --query "services[0].runningCount" --output text)
DESIRED=$(aws ecs describe-services \
  --cluster $CLUSTER --services $SERVICE --region $REGION \
  --query "services[0].desiredCount" --output text)

if [ "$RUNNING" -eq "$DESIRED" ] && [ "$DESIRED" -gt 0 ]; then
  echo "  ✅ ECS 정상: 실행중 $RUNNING / 목표 $DESIRED"
else
  echo "  ❌ ECS 이상: 실행중 $RUNNING / 목표 $DESIRED"
fi

# 2. HTTP 헬스체크
echo ""
echo "[2/4] HTTP 헬스체크 확인 중..."
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 \
  "http://$ALB_DNS/health" || echo "000")

if [ "$HTTP_CODE" = "200" ]; then
  echo "  ✅ HTTP 정상: 응답코드 $HTTP_CODE"
else
  echo "  ❌ HTTP 이상: 응답코드 $HTTP_CODE"
fi

# 3. 최근 에러 로그
echo ""
echo "[3/4] 최근 5분 에러 로그 확인 중..."
START_TIME=$(date -d '5 minutes ago' +%s000 2>/dev/null || \
             date -v-5M +%s000)  # macOS 호환

ERROR_COUNT=$(aws logs filter-log-events \
  --log-group-name "$LOG_GROUP" \
  --start-time $START_TIME \
  --filter-pattern "ERROR" \
  --region $REGION \
  --query "length(events)" --output text 2>/dev/null || echo "0")

if [ "$ERROR_COUNT" = "0" ]; then
  echo "  ✅ 에러 로그 없음"
else
  echo "  ⚠️  최근 에러 로그 ${ERROR_COUNT}건 발견 — 확인 필요"
fi

# 4. 최종 판정
echo ""
echo "======================================"
if [ "$HTTP_CODE" = "200" ] && [ "$RUNNING" -eq "$DESIRED" ]; then
  echo "  🎉 최종 결과: 배포 성공"
else
  echo "  🚨 최종 결과: 확인 필요 — 위 항목을 점검하세요"
fi
echo "======================================"
```

---

## 따라 하기 실습

### 실습 1 — validate-deployment.sh 만들고 실행하기

1. 위 스크립트를 로컬에 저장합니다.

```bash
# 프로젝트 루트에서 실행
mkdir -p scripts
vi scripts/validate-deployment.sh
# (위 스크립트 내용 붙여넣기)
chmod +x scripts/validate-deployment.sh
```

2. 자신의 환경에 맞게 변수를 수정합니다.

```bash
# scripts/validate-deployment.sh 상단 변수 수정
APP_NAME="my-app"          # 본인 앱 이름으로 변경
CLUSTER="my-cluster"       # 본인 ECS 클러스터 이름으로 변경
ALB_DNS="..."              # AWS 콘솔 > EC2 > 로드밸런서에서 DNS 복사
LOG_GROUP="/ecs/..."       # CloudWatch > 로그 그룹에서 확인
```

3. 스크립트를 실행하고 결과를 확인합니다.

```bash
./scripts/validate-deployment.sh

# 정상 출력 예시:
# ✅ ECS 정상: 실행중 2 / 목표 2
# ✅ HTTP 정상: 응답코드 200
# ✅ 에러 로그 없음
# 🎉 최종 결과: 배포 성공
```

---

### 실습 2 — 일부러 실패 상황 만들어서 에러 읽기

실제 장애를 경험하기 전에, 의도적으로 실패를 만들어 봅니다.

1. 헬스체크 경로를 잘못된 URL로 바꿔서 502 에러를 확인합니다.

```bash
# ALB_DNS 변수만 틀린 값으로 바꿔서 실행
ALB_DNS="wrong-dns-name" ./scripts/validate-deployment.sh

# 예상 출력:
# ❌ HTTP 이상: 응답코드 000
# 🚨 최종 결과: 확인 필요 — 위 항목을 점검하세요
```

2. 에러 코드별 의미를 매핑표로 만들어 스크립트에 추가합니다.

```bash
# validate-deployment.sh 의 HTTP 체크 부분 아래에 추가
case $HTTP_CODE in
  200) echo "  정상 응답입니다." ;;
  502) echo "  힌트: ECS 태스크가 죽었거나 포트가 틀렸을 수 있습니다." ;;
  503) echo "  힌트: ALB 타깃 그룹 헬스체크를 확인하세요." ;;
  504) echo "  힌트: 앱 응답이 너무 느립니다. 타임아웃을 확인하세요." ;;
  000) echo "  힌트: DNS 이름이 틀렸거나 네트워크 연결 문제입니다." ;;
  *)   echo "  힌트: 예상치 못한 응답코드입니다. 로그를 확인하세요." ;;
esac
```

---

### 실습 3 — CI/CD 파이프라인 배포 단계 뒤에 validation 연결하기

배포 스크립트 뒤에 validation을 자동으로 실행되도록 연결합니다.

```bash
# deploy.sh (기존 배포 스크립트 끝에 추가)

echo "배포 완료. 60초 후 validation 시작..."
sleep 60  # ECS 태스크가 완전히 뜰 시간을 줌

bash scripts/validate-deployment.sh
EXIT_CODE=$?

if [ $EXIT_CODE -ne 0 ]; then
  echo "Validation 실패! 배포를 롤백하거나 수동 확인이 필요합니다."
  exit 1
fi

echo "배포 + Validation 모두 완료."
```

```yaml
# GitHub Actions에서 사용하는 경우 (.github/workflows/deploy.yml)
- name: Deploy to ECS
  run: bash scripts/deploy.sh

- name: Run deployment validation
  run: |
    sleep 60
    bash scripts/validate-deployment.sh
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    AWS_DEFAULT_REGION: ap-northeast-2
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 원인 | 해결 방법 |
|---|---|---|---|
| AWS CLI 인증 안 됨 | `Unable to locate credentials` | AWS 자격증명 미설정 | `aws configure` 실행 후 Access Key 입력 |
| 리전 누락 | `Could not connect to the endpoint URL` | 리전이 기본값과 다름 | 명령어에 `--region ap-northeast-2` 추가 |
| ECS 태스크 계속 재시작 | `runningCount=0, desiredCount=2` | 컨테이너 크래시 또는 메모리 부족 | CloudWatch 로그에서 `Task stopped` 이유 확인 |
| ALB 502 에러 | HTTP 응답코드 502 | 컨테이너 포트 불일치 또는 앱 크래시 | ECS 태스크 정의의 포트 매핑 확인 |
| ALB 503 에러 | HTTP 응답코드 503 | 타깃 그룹 헬스체크 실패 | EC2 > 타깃 그룹 > 헬스체크 경로 확인 |
| 로그 그룹 없음 | `ResourceNotFoundException` | 로그 그룹 이름 오타 또는 미생성 | `aws logs describe-log-groups`로 정확한 이름 확인 |
| macOS에서 `date -d` 에러 | `date: illegal option -- d` | macOS는 BSD date 사용 | `date -d '5 minutes ago'` → `date -v-5M` 로 변경 |
| curl 타임아웃 | `curl: (28) Operation timed out` | 보안 그룹에서 포트 막힘 | EC2 보안 그룹 인바운드 규칙 80/443 포트 허용 확인 |

---

## 확인 체크리스트

배포를 마친 후 아래 항목을 위에서 아래로 순서대로 체크합니다.

**인프라 레벨**
- [ ] ECS 서비스의 `runningCount == desiredCount` 확인
- [ ] EC2 인스턴스 상태가 `running`이고 상태 검사(Status Check)가 `2/2 passed`
- [ ] Auto Scaling 그룹의 인스턴스 수가 예상값과 일치

**네트워크 레벨**
- [ ] ALB 타깃 그룹에 `healthy` 상태의 타깃이 1개 이상 존재
- [ ] 보안 그룹 인바운드 규칙에 서비스 포트(80/443) 허용 여부
- [ ] Route 53 또는 도메인 DNS가 ALB로 올바르게 연결

**애플리케이션 레벨**
- [ ] `/health` 엔드포인트가 HTTP 200 응답
- [ ] 응답 바디에 `"status": "healthy"` (또는 앱 기준 정상 응답) 포함
- [ ] 최근 5분 CloudWatch 로그에 `ERROR` 없음

**데이터 레벨**
- [ ] DB 연결 성공 (헬스체크 응답 또는 로그로 확인)
- [ ] 환경 변수 (`DATABASE_URL`, `SECRET_KEY` 등) 정상 주입 확인

**최종 확인**
- [ ] `validate-deployment.sh` 스크립트 실행 결과 전 항목 `✅`
- [ ] 이전 버전 대비 응답 속도 큰 변화 없음 (CloudWatch 메트릭)
- [ ] 팀 채널에 배포 완료 공지

---

## 한 번 더 생각해 보기

1. `validate-deployment.sh` 스크립트가 모든 항목을 `✅`로 통과했는데 실제 사용자는 에러를 경험하고 있다면, 어떤 항목이 빠진 것일까요? 어떤 체크를 추가하면 좋을까요?

2. 배포 후 validation은 몇 번, 얼마나 자주 실행해야 할까요? 배포 직후 한 번만 하면 충분할까요, 아니면 일정 시간 동안 반복해서 확인해야 할까요?

3. 지금 만든 validation sheet는 배포가 성공한 뒤를 확인합니다. 배포 **전**에 미리 확인해야 할 항목과 **배포 중**에 봐야 할 항목은 무엇이 다를까요?

---

## 다음 장

다음 장에서는 validation 결과를 Slack이나 이메일로 자동 알림을 보내는 방법과, 실패 시 자동 롤백을 트리거하는 파이프라인 구성을 배웁니다.