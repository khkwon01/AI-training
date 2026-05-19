## 이 장에서 배우는 것

- AWS EventBridge가 무엇인지, 왜 쓰는지 이해한다
- cron 표현식과 rate 표현식으로 스케줄을 설정하는 방법을 배운다
- Python Lambda 함수를 EventBridge 규칙과 연결하는 방법을 익힌다
- AWS 콘솔과 CLI 양쪽에서 스케줄을 만들고 확인하는 방법을 연습한다
- 스케줄이 정상 동작하는지 CloudWatch Logs로 확인하는 방법을 배운다

---

## 먼저 쉬운 설명

매일 오전 9시에 알람이 울리도록 스마트폰을 설정해 본 적 있나요? AWS EventBridge 스케줄링은 그것과 똑같습니다. 다만 스마트폰 대신 **클라우드 서버**에 알람을 설정하는 거예요.

예를 들어 이런 상황을 생각해 보세요.

- "매일 자정에 데이터베이스를 백업하고 싶어."
- "매주 월요일 오전 8시에 주간 리포트 이메일을 자동으로 보내고 싶어."
- "5분마다 서버 상태를 체크하고 싶어."

이런 반복 작업을 사람이 직접 하면 너무 힘들죠. EventBridge가 대신해 줍니다. EventBridge는 **"몇 시에, 뭘 실행해라"** 라고 AWS에 등록해두면, 그 시간이 되면 자동으로 Lambda 함수 같은 작업을 실행시켜 주는 서비스입니다.

> 핵심 한 줄 요약: EventBridge = 클라우드용 자동 알람 시계

---

## 1. EventBridge 기본 개념: 규칙(Rule)과 대상(Target)

EventBridge 스케줄링은 딱 두 가지만 기억하면 됩니다.

| 용어 | 쉬운 설명 | 예시 |
|------|-----------|------|
| **규칙 (Rule)** | 언제 실행할지 | "매일 오전 9시에" |
| **대상 (Target)** | 무엇을 실행할지 | "이 Lambda 함수를 실행해" |

규칙 하나에 대상을 최대 5개까지 연결할 수 있습니다. 즉, 오전 9시가 되면 Lambda 함수 실행 + SNS 알림 발송 + SQS 큐에 메시지 전송을 동시에 할 수 있어요.

```
[EventBridge 규칙]
     |
     ├──▶ Lambda 함수 (보고서 생성)
     ├──▶ SNS 토픽 (알림 발송)
     └──▶ SQS 큐 (작업 대기열에 추가)
```

---

## 2. 스케줄 표현식: rate와 cron

EventBridge에서 "언제 실행할지"를 표현하는 방법은 두 가지입니다.

### 2-1. rate 표현식 — 간단한 반복

가장 쉬운 방법입니다. **"몇 분/시간/일마다 실행해"** 라고 적으면 됩니다.

```
rate(5 minutes)   # 5분마다
rate(1 hour)      # 1시간마다
rate(1 day)       # 하루마다
rate(7 days)      # 7일마다
```

> 주의: `1`일 때는 단수형(`minute`, `hour`, `day`), `2` 이상일 때는 복수형(`minutes`, `hours`, `days`)을 써야 합니다.

```
rate(1 minute)    # ✅ 올바름
rate(1 minutes)   # ❌ 오류 발생
rate(2 minute)    # ❌ 오류 발생
rate(2 minutes)   # ✅ 올바름
```

### 2-2. cron 표현식 — 정밀한 시간 지정

특정 요일, 특정 시간에 실행하고 싶을 때 씁니다. AWS의 cron은 **6개 필드**로 구성됩니다.

```
cron(분 시 일 월 요일 연도)
```

| 필드 | 범위 | 특수문자 |
|------|------|----------|
| 분 | 0–59 | , - * / |
| 시 | 0–23 | , - * / |
| 일 | 1–31 | , - * ? / L W |
| 월 | 1–12 또는 JAN–DEC | , - * / |
| 요일 | 1–7 또는 SUN–SAT | , - * ? / L # |
| 연도 | 1970–2199 | , - * / |

> **중요**: AWS cron에서 시간은 항상 **UTC** 기준입니다. 한국 시간(KST)은 UTC+9이므로, 한국 오전 9시 = UTC 오전 0시(자정)입니다.

```
# 자주 쓰는 cron 예시 (모두 UTC 기준)

cron(0 0 * * ? *)      # 매일 UTC 00:00 = 한국 오전 9시
cron(0 15 * * ? *)     # 매일 UTC 15:00 = 한국 자정(00:00)
cron(0 9 ? * MON *)    # 매주 월요일 UTC 09:00 = 한국 월요일 오후 6시
cron(30 8 1 * ? *)     # 매월 1일 UTC 08:30
cron(0 0 ? * MON-FRI *) # 평일(월~금)마다 UTC 자정
```

`?` 는 "상관없음"이라는 뜻입니다. 일(day)과 요일(weekday) 중 하나를 지정하면 나머지는 반드시 `?`로 써야 합니다.

---

## 3. Python Lambda 함수 준비하기

EventBridge가 실행할 Lambda 함수를 먼저 만들어야 합니다. 스케줄에 의해 실행되는 Lambda는 `event` 안에 EventBridge가 보내주는 정보가 담겨 옵니다.

```python
# lambda_function.py
import json
import logging
from datetime import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    """
    EventBridge 스케줄에 의해 자동으로 호출되는 Lambda 함수.
    event 딕셔너리에는 EventBridge가 보내는 메타데이터가 들어 있다.
    """
    # EventBridge가 보내주는 실행 시각
    execution_time = event.get("time", "시간 정보 없음")
    
    logger.info(f"스케줄 작업 시작: {execution_time}")
    
    # 실제로 하고 싶은 작업을 여기에 작성
    result = do_scheduled_task()
    
    logger.info(f"스케줄 작업 완료: {result}")
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "스케줄 작업 완료",
            "executed_at": execution_time,
            "result": result
        }, ensure_ascii=False)
    }

def do_scheduled_task():
    """실제 업무 로직을 여기에 작성한다."""
    now = datetime.utcnow().strftime("%Y-%m-%d %H:%M:%S UTC")
    return f"작업 성공적으로 완료됨 ({now})"
```

EventBridge에서 Lambda로 전달되는 `event` 객체는 다음과 같은 형태입니다.

```json
{
    "version": "0",
    "id": "12345678-1234-1234-1234-123456789012",
    "detail-type": "Scheduled Event",
    "source": "aws.events",
    "account": "123456789012",
    "time": "2026-05-18T00:00:00Z",
    "region": "ap-northeast-2",
    "resources": [
        "arn:aws:events:ap-northeast-2:123456789012:rule/my-daily-rule"
    ],
    "detail": {}
}
```

---

## 4. AWS CLI로 EventBridge 규칙 만들기

콘솔(웹 화면) 대신 명령줄(터미널)에서 EventBridge 규칙을 만드는 방법입니다. 코드로 관리하면 나중에 수정하거나 다른 환경에 똑같이 복사하기 쉽습니다.

### 4-1. 규칙(Rule) 생성

```bash
# schedule_setup.sh

# 매일 한국 오전 9시(UTC 00:00)에 실행되는 규칙 생성
aws events put-rule \
  --name "daily-report-rule" \
  --schedule-expression "cron(0 0 * * ? *)" \
  --state ENABLED \
  --description "매일 오전 9시 일간 리포트 생성" \
  --region ap-northeast-2
```

성공하면 아래처럼 규칙의 ARN이 출력됩니다.

```json
{
    "RuleArn": "arn:aws:events:ap-northeast-2:123456789012:rule/daily-report-rule"
}
```

### 4-2. Lambda에 EventBridge 호출 권한 부여

EventBridge가 Lambda를 실행하려면 권한이 필요합니다. 권한 없이 연결하면 Lambda가 실행되지 않습니다.

```bash
# Lambda 함수에 EventBridge 호출 권한 추가
aws lambda add-permission \
  --function-name "my-report-lambda" \
  --statement-id "EventBridgeInvokePermission" \
  --action "lambda:InvokeFunction" \
  --principal "events.amazonaws.com" \
  --source-arn "arn:aws:events:ap-northeast-2:123456789012:rule/daily-report-rule" \
  --region ap-northeast-2
```

### 4-3. 대상(Target) 연결

규칙에 Lambda 함수를 대상으로 등록합니다.

```bash
# targets.json 파일을 만들어서 사용
cat > targets.json << 'EOF'
[
  {
    "Id": "DailyReportLambdaTarget",
    "Arn": "arn:aws:lambda:ap-northeast-2:123456789012:function:my-report-lambda"
  }
]
EOF

# 규칙에 대상 연결
aws events put-targets \
  --rule "daily-report-rule" \
  --targets file://targets.json \
  --region ap-northeast-2
```

성공하면 아래처럼 출력됩니다.

```json
{
    "FailedEntryCount": 0,
    "FailedEntries": []
}
```

`FailedEntryCount`가 0이면 성공입니다. 1 이상이면 `FailedEntries`에서 오류 이유를 확인하세요.

---

## 5. CloudWatch Logs로 실행 확인하기

규칙을 만들었다고 끝이 아닙니다. 실제로 Lambda가 잘 실행되는지 확인해야 합니다.

```python
# check_logs.py — 최근 Lambda 실행 로그를 가져오는 스크립트
import boto3
from datetime import datetime, timedelta

def get_recent_lambda_logs(function_name: str, hours: int = 1):
    """최근 N시간 동안의 Lambda 실행 로그를 출력한다."""
    
    logs_client = boto3.client("logs", region_name="ap-northeast-2")
    
    log_group = f"/aws/lambda/{function_name}"
    start_time = int((datetime.utcnow() - timedelta(hours=hours)).timestamp() * 1000)
    
    try:
        # 로그 스트림 목록 조회
        streams = logs_client.describe_log_streams(
            logGroupName=log_group,
            orderBy="LastEventTime",
            descending=True,
            limit=3
        )
        
        for stream in streams["logStreams"]:
            stream_name = stream["logStreamName"]
            print(f"\n--- 로그 스트림: {stream_name} ---")
            
            # 각 스트림에서 로그 이벤트 조회
            events = logs_client.get_log_events(
                logGroupName=log_group,
                logStreamName=stream_name,
                startTime=start_time,
                limit=20
            )
            
            for event in events["events"]:
                timestamp = datetime.fromtimestamp(event["timestamp"] / 1000)
                print(f"[{timestamp}] {event['message'].strip()}")
                
    except logs_client.exceptions.ResourceNotFoundException:
        print(f"로그 그룹 '{log_group}'을 찾을 수 없습니다.")
        print("Lambda 함수가 아직 한 번도 실행되지 않았을 수 있습니다.")

if __name__ == "__main__":
    get_recent_lambda_logs("my-report-lambda", hours=24)
```

```bash
python check_logs.py
```

정상 실행 시 출력 예시:

```
--- 로그 스트림: 2026/05/18/[$LATEST]abc123 ---
[2026-05-18 00:00:02] START RequestId: abc-123
[2026-05-18 00:00:02] INFO 스케줄 작업 시작: 2026-05-18T00:00:00Z
[2026-05-18 00:00:03] INFO 스케줄 작업 완료: 작업 성공적으로 완료됨 (2026-05-18 00:00:03 UTC)
[2026-05-18 00:00:03] END RequestId: abc-123
[2026-05-18 00:00:03] REPORT Duration: 1234.56 ms
```

---

## 따라 하기 실습

### 실습 1: 5분마다 실행되는 헬스체크 Lambda 만들기

**목표**: 5분마다 "서버 살아있음"을 로그에 기록하는 시스템을 만든다.

1. 아래 내용으로 `healthcheck_lambda.py` 파일을 만든다.

```python
# healthcheck_lambda.py
import json
import logging
from datetime import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    now = datetime.utcnow().isoformat()
    logger.info(f"[헬스체크] 서버 정상 동작 중 - {now}")
    return {
        "statusCode": 200,
        "body": json.dumps({"status": "healthy", "timestamp": now})
    }
```

2. AWS 콘솔에서 Lambda 함수 `healthcheck-lambda`를 생성하고 위 코드를 붙여넣는다.

3. 터미널에서 아래 명령을 실행하여 5분마다 실행되는 EventBridge 규칙을 만든다.

```bash
aws events put-rule \
  --name "healthcheck-every-5min" \
  --schedule-expression "rate(5 minutes)" \
  --state ENABLED \
  --region ap-northeast-2

aws lambda add-permission \
  --function-name "healthcheck-lambda" \
  --statement-id "HealthCheckSchedule" \
  --action "lambda:InvokeFunction" \
  --principal "events.amazonaws.com" \
  --source-arn "arn:aws:events:ap-northeast-2:$(aws sts get-caller-identity --query Account --output text):rule/healthcheck-every-5min" \
  --region ap-northeast-2

aws events put-targets \
  --rule "healthcheck-every-5min" \
  --targets '[{"Id":"HealthCheckTarget","Arn":"arn:aws:lambda:ap-northeast-2:YOUR_ACCOUNT_ID:function:healthcheck-lambda"}]' \
  --region ap-northeast-2
```

4. 5분을 기다린 후 CloudWatch Logs에서 `/aws/lambda/healthcheck-lambda` 로그 그룹을 열어 로그가 쌓이는지 확인한다.

---

### 실습 2: 평일 오전 9시에 실행되는 일간 리포트 만들기

**목표**: 실습 1을 발전시켜, 주말은 건너뛰고 평일에만 실행되는 스케줄을 만든다.

1. `daily_report_lambda.py` 파일을 만든다.

```python
# daily_report_lambda.py
import json
import logging
from datetime import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    execution_time = event.get("time", "")
    
    logger.info(f"일간 리포트 생성 시작: {execution_time}")
    
    report = generate_daily_report()
    
    logger.info(f"일간 리포트 완료: {report['summary']}")
    
    return {
        "statusCode": 200,
        "body": json.dumps(report, ensure_ascii=False)
    }

def generate_daily_report():
    """실제 서비스에서는 DB 조회, 계산 등을 여기서 수행한다."""
    today = datetime.utcnow().strftime("%Y-%m-%d")
    return {
        "date": today,
        "summary": f"{today} 일간 리포트 생성 완료",
        "metrics": {
            "total_users": 1234,
            "new_signups": 56,
            "active_sessions": 789
        }
    }
```

2. Lambda 함수 `daily-report-lambda`를 생성하고 위 코드를 적용한다.

3. 평일 오전 9시(KST) = UTC 00:00에 실행되는 cron 규칙을 만든다.

```bash
# 평일(월~금) UTC 00:00 = 한국 오전 9시
aws events put-rule \
  --name "weekday-daily-report" \
  --schedule-expression "cron(0 0 ? * MON-FRI *)" \
  --state ENABLED \
  --description "평일 오전 9시 일간 리포트" \
  --region ap-northeast-2
```

4. 권한 추가와 대상 연결은 실습 1의 명령을 참고하여 `daily-report-lambda`에 적용한다.

---

### 실습 3: 규칙 목록 조회 및 비활성화하기

**목표**: 만들어 둔 규칙을 관리하는 방법을 익힌다.

1. 현재 계정의 모든 EventBridge 규칙을 조회한다.

```bash
aws events list-rules --region ap-northeast-2
```

2. 특정 규칙의 상세 정보와 연결된 대상을 확인한다.

```bash
# 규칙 상세 조회
aws events describe-rule \
  --name "healthcheck-every-5min" \
  --region ap-northeast-2

# 규칙에 연결된 대상 조회
aws events list-targets-by-rule \
  --rule "healthcheck-every-5min" \
  --region ap-northeast-2
```

3. 잠시 규칙을 중단하고 싶을 때는 삭제 대신 비활성화한다. (삭제하면 다시 만들어야 하지만, 비활성화는 언제든 다시 켤 수 있습니다.)

```bash
# 규칙 비활성화 (DISABLED)
aws events disable-rule \
  --name "healthcheck-every-5min" \
  --region ap-northeast-2

# 다시 활성화
aws events enable-rule \
  --name "healthcheck-every-5min" \
  --region ap-northeast-2
```

---

## 자주 하는 실수

| 실수 | 실제 오류 메시지 | 해결 방법 |
|------|-----------------|-----------|
| rate 표현식에서 단수/복수 혼동 | `Parameter ScheduleExpression is not valid.` | `rate(1 minute)`, `rate(2 minutes)` 처럼 숫자에 맞춰 단복수 구분 |
| cron에서 일(day)과 요일(weekday)을 동시에 지정 | `Parameter ScheduleExpression is not valid.` | 둘 중 하나만 지정하고 나머지는 반드시 `?` 로 표시 |
| Lambda 호출 권한 누락 | Lambda 함수가 실행되지 않음, CloudWatch에 아무 로그도 없음 | `aws lambda add-permission` 으로 EventBridge 호출 권한 추가 |
| 시간대(Timezone) 혼동 | 엉뚱한 시간에 실행됨 | AWS cron은 항상 UTC 기준; KST = UTC+9, 오전 9시 KST = `cron(0 0 ...)` |
| 계정 ID를 ARN에 하드코딩 후 다른 계정에서 오류 | `ResourceNotFoundException` 또는 권한 오류 | `$(aws sts get-caller-identity --query Account --output text)` 로 동적 조회 |
| 대상 연결 후 `FailedEntryCount: 1` | `FailedEntries` 에 `AccessDenied` | Lambda 함수 ARN 오타 확인, Lambda add-permission 단계 누락 여부 확인 |
| cron에서 연도 필드 누락 | `Parameter ScheduleExpression is not valid.` | AWS cron은 6개 필드 필수: `cron(분 시 일 월 요일 연도)` |

---

## 확인 체크리스트

- [ ] `rate(5 minutes)` 와 `cron(0 9 * * ? *)` 의 차이를 설명할 수 있다
- [ ] AWS cron이 UTC 기준임을 알고, 한국 오전 9시를 cron으로 올바르게 표현할 수 있다
- [ ] EventBridge 규칙(Rule)과 대상(Target)이 각각 무엇인지 설명할 수 있다
- [ ] Lambda 함수에 EventBridge 호출 권한(`lambda:InvokeFunction`)을 추가하는 CLI 명령을 쓸 수 있다
- [ ] `aws events put-rule`, `aws events put-targets` 명령으로 스케줄을 직접 만들 수 있다
- [ ] 규칙을 삭제하지 않고 비활성화(`disable-rule`)하는 방법을 안다
- [ ] CloudWatch Logs에서 Lambda 실행 로그를 찾아 성공 여부를 확인할 수 있다
- [ ] `FailedEntryCount: 1` 오류가 났을 때 어디를 먼저 확인해야 하는지 안다

---

## 한 번 더 생각해 보기

1. **time zone 문제**: 서비스를 한국 사용자뿐 아니라 미국 사용자에게도 제공한다면, "매일 오전 9시에 이메일을 보내는" 스케줄을 어떻게 설계해야 할까요? EventBridge 규칙을 여러 개 만드는 방법과 Lambda 내부에서 시간대를 계산하는 방법 중 어느 쪽이 더 나을까요?

2. **장애 대응**: EventBridge가 Lambda를 실행하려 했는데 Lambda에 오류가 발생했습니다. EventBridge는 자동으로 재시도할까요? 재시도 횟수와 간격을 어디서 설정할 수 있는지 AWS 문서에서 찾아보세요. (힌트: EventBridge의 "재시도 정책" 항목을 검색해 보세요.)

3. **비용 생각하기**: `rate(1 minute)` 으로 설정한 규칙이 하루 동안 실행된다면 Lambda가 몇 번 호출될까요? AWS Lambda 프리 티어(월 100만 건 무료)와 비교했을 때 이 스케줄이 비용 측면에서 안전한지 계산해 보세요.

---

## 다음 장

다음 장에서는 EventBridge 스케줄이 실패했을 때 자동으로 Slack 또는 이메일로 알림을 받는 **CloudWatch 경보(Alarm)와 SNS 연동** 방법을 배웁니다.