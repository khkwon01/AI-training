## 이 장에서 배우는 것

- AWS 서버리스(Lambda, API Gateway, DynamoDB)에서 비용이 발생하는 원리
- 무료 티어(Free Tier) 한도와 실제로 돈이 나가는 시점
- Lambda 메모리·타임아웃 설정이 비용에 미치는 영향
- DynamoDB 온디맨드 vs 프로비저닝 모드 선택 기준
- AWS Cost Explorer와 Budgets 알림으로 청구 폭탄 예방하기
- 코드 한 줄로 Lambda 콜드 스타트 줄이는 방법

---

## 먼저 쉬운 설명

서버리스는 "서버를 직접 관리하지 않는다"는 뜻이지, "공짜"라는 뜻이 아닙니다.

처음 AWS를 쓰면 이런 일이 생깁니다.

> "분명히 작은 프로젝트인데 한 달 뒤 청구서가 50달러…?"

왜 그럴까요? 서버리스는 **쓴 만큼만** 낸다는 게 장점인데, 바로 그 "쓴 만큼"이 생각보다 빠르게 쌓이기 때문입니다. Lambda를 잘못 설정하면 함수가 1초에 100번 호출되고, DynamoDB를 온디맨드로 놔두면 트래픽 스파이크 한 번에 수십 달러가 나갑니다.

이 장에서는 **비용이 폭발하기 전에** 미리 막는 방법을 배웁니다. 실제로 비용을 아끼는 코드와 설정을 직접 적용해 봅니다.

---

## 1. 비용 구조 이해하기 — 어디서 돈이 나가나?

서버리스 앱의 주요 과금 요소는 세 가지입니다.

| 서비스 | 과금 단위 | 무료 티어 (월) |
|---|---|---|
| Lambda | 요청 수 + GB-초 | 100만 요청, 400,000 GB-초 |
| API Gateway (REST) | 요청 수 | 100만 요청 |
| DynamoDB | 읽기/쓰기 유닛 | 25 WCU, 25 RCU, 25 GB |

**GB-초**가 낯설죠? 간단히 말하면 `메모리(GB) × 실행 시간(초)`입니다.

```python
# 예시: Lambda 비용 계산
메모리 = 512  # MB → 0.5 GB
실행시간 = 2  # 초
호출수 = 1_000_000  # 월 100만 번

gb_초 = (메모리 / 1024) * 실행시간 * 호출수
# → 0.5 × 2 × 1,000,000 = 1,000,000 GB-초
# 무료 티어 400,000 GB-초 초과 → 600,000 GB-초 과금
# 요금: 600,000 × $0.0000166667 ≈ $10
```

**핵심 교훈**: 메모리를 1024MB로 올리면 같은 호출 수에서 비용이 두 배가 됩니다.

---

## 2. Lambda 메모리와 타임아웃 최적화

Lambda는 메모리를 늘리면 CPU도 함께 올라갑니다. 그래서 메모리를 늘렸을 때 **실행 시간이 크게 줄면** 오히려 비용이 내려갑니다.

```python
# lambda_function.py — 타임아웃과 메모리를 의식한 구조

import json
import boto3
import os

# ✅ 핸들러 바깥에서 클라이언트 생성 → 콜드 스타트 이후 재사용됨
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table(os.environ['TABLE_NAME'])

def lambda_handler(event, context):
    # 남은 실행 시간 확인 (ms)
    remaining_ms = context.get_remaining_time_in_millis()
    
    if remaining_ms < 500:
        # 타임아웃 직전이면 빠르게 에러 반환
        return {
            'statusCode': 503,
            'body': json.dumps({'error': '처리 시간 초과 임박, 다시 시도하세요.'})
        }
    
    try:
        response = table.get_item(Key={'userId': event['userId']})
        item = response.get('Item', {})
        return {
            'statusCode': 200,
            'body': json.dumps(item, ensure_ascii=False)
        }
    except Exception as e:
        print(f"오류 발생: {e}")  # CloudWatch Logs로 전송됨
        return {
            'statusCode': 500,
            'body': json.dumps({'error': '서버 오류'})
        }
```

**타임아웃 설정 원칙**:

```yaml
# serverless.yml 또는 SAM template.yaml 예시
functions:
  getUser:
    handler: lambda_function.lambda_handler
    memorySize: 256   # 시작은 256MB, 프로파일링 후 조정
    timeout: 10       # ❌ 기본값 3초를 무작정 300초로 올리지 말 것
    environment:
      TABLE_NAME: !Ref UsersTable
```

> **팁**: 타임아웃을 높게 잡으면 버그가 있는 함수가 오래 실행되어 비용이 쌓입니다. 실제 P99 응답 시간의 2~3배로 설정하세요.

---

## 3. DynamoDB 온디맨드 vs 프로비저닝 선택하기

```python
# dynamo_cost_check.py — 어떤 모드가 유리한지 판단하는 기준

def 모드_추천(월간_읽기_횟수: int, 월간_쓰기_횟수: int) -> str:
    """
    온디맨드: 쓴 만큼 내는 방식 (예측 불가 트래픽에 유리)
    프로비저닝: 미리 용량 예약 (예측 가능 트래픽에 유리, 더 저렴)
    """
    # 온디맨드 단가 (서울 리전 기준, 2025년)
    온디맨드_읽기_단가 = 0.000000025  # per RCU
    온디맨드_쓰기_단가 = 0.000000125  # per WCU
    
    온디맨드_비용 = (월간_읽기_횟수 * 온디맨드_읽기_단가 +
                    월간_쓰기_횟수 * 온디맨드_쓰기_단가)
    
    # 프로비저닝: 무료 티어 25 RCU / 25 WCU 이후 과금
    프로비저닝_비용_예시 = 0.00065 * 10  # 10 WCU 예약 시 월 $0.0065
    
    if 온디맨드_비용 < 프로비저닝_비용_예시:
        return f"온디맨드 추천 (예상 월 비용: ${온디맨드_비용:.4f})"
    else:
        return f"프로비저닝 추천 (온디맨드 예상: ${온디맨드_비용:.2f}/월)"


# 실행 예시
print(모드_추천(월간_읽기_횟수=50_000, 월간_쓰기_횟수=10_000))
# → 온디맨드 추천 (예상 월 비용: $0.0026)

print(모드_추천(월간_읽기_횟수=5_000_000, 월간_쓰기_횟수=1_000_000))
# → 프로비저닝 추천 (온디맨드 예상: $0.25/월)
```

**초보자 기본 전략**:
- 트래픽이 하루 1,000회 미만 → **온디맨드** (무료 티어로 거의 충당)
- 트래픽이 안정적으로 하루 10,000회 이상 → **프로비저닝 + Auto Scaling**

---

## 4. CloudWatch Logs 비용 줄이기

많은 초보자가 놓치는 비용 항목이 바로 로그입니다.

```python
# lambda_function.py — 로그 레벨 환경변수로 제어하기

import logging
import os
import json

# ❌ 나쁜 예: 항상 DEBUG 레벨로 모든 것을 출력
# logging.basicConfig(level=logging.DEBUG)

# ✅ 좋은 예: 환경변수로 레벨 조절
LOG_LEVEL = os.environ.get('LOG_LEVEL', 'WARNING')
logger = logging.getLogger()
logger.setLevel(getattr(logging, LOG_LEVEL))

def lambda_handler(event, context):
    # DEBUG 로그는 개발 환경에서만 출력됨
    logger.debug(f"전체 이벤트: {json.dumps(event)}")  # 프로덕션에서는 출력 안 됨
    logger.info("함수 시작")
    
    # 비즈니스 로직
    결과 = 처리(event)
    
    logger.warning("처리 완료")  # WARNING 이상만 실제 로그로 저장됨
    return 결과

def 처리(event):
    return {'status': 'ok'}
```

```yaml
# SAM template.yaml — 로그 보존 기간 설정 (기본값은 무제한!)
Resources:
  GetUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      LoggingConfig:
        LogFormat: JSON
        ApplicationLogLevel: WARN   # INFO/DEBUG 로그 제외
        SystemLogLevel: WARN

  # ✅ 로그 그룹에 보존 기간 명시 (기본: 무제한 → 비용 계속 쌓임)
  GetUserFunctionLogGroup:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: !Sub /aws/lambda/${GetUserFunction}
      RetentionInDays: 14   # 2주 후 자동 삭제
```

---

## 5. AWS Budgets로 청구 폭탄 예방하기

코드로 예산 알림을 설정하면 비용이 임계값을 넘을 때 이메일을 받습니다.

```python
# setup_budget_alert.py — 월 $5 초과 시 알림 설정

import boto3

def 예산_알림_생성(이메일: str, 한도_달러: float = 5.0):
    budgets = boto3.client('budgets')
    sts = boto3.client('sts')
    
    계정_id = sts.get_caller_identity()['Account']
    
    response = budgets.create_budget(
        AccountId=계정_id,
        Budget={
            'BudgetName': '서버리스-앱-월예산',
            'BudgetLimit': {
                'Amount': str(한도_달러),
                'Unit': 'USD'
            },
            'TimeUnit': 'MONTHLY',
            'BudgetType': 'COST'
        },
        NotificationsWithSubscribers=[
            {
                'Notification': {
                    'NotificationType': 'ACTUAL',
                    'ComparisonOperator': 'GREATER_THAN',
                    'Threshold': 80,           # 한도의 80% 도달 시
                    'ThresholdType': 'PERCENTAGE'
                },
                'Subscribers': [
                    {
                        'SubscriptionType': 'EMAIL',
                        'Address': 이메일
                    }
                ]
            }
        ]
    )
    print(f"예산 알림 생성 완료: 월 ${한도_달러} 한도, 80% 도달 시 {이메일}로 알림")
    return response


if __name__ == '__main__':
    예산_알림_생성('내이메일@example.com', 한도_달러=5.0)
```

---

## 따라 하기 실습

### 실습 1 — 내 Lambda 함수 비용 시뮬레이션하기

`cost_simulator.py` 파일을 만들어 현재 설정의 월 예상 비용을 계산합니다.

```python
# cost_simulator.py

def lambda_월_비용(
    메모리_mb: int,
    평균_실행시간_ms: int,
    월_호출수: int
) -> dict:
    메모리_gb = 메모리_mb / 1024
    실행시간_초 = 평균_실행시간_ms / 1000
    
    gb_초 = 메모리_gb * 실행시간_초 * 월_호출수
    무료_gb_초 = 400_000
    과금_gb_초 = max(0, gb_초 - 무료_gb_초)
    
    gb_초_요금 = 과금_gb_초 * 0.0000166667
    
    무료_요청수 = 1_000_000
    과금_요청수 = max(0, 월_호출수 - 무료_요청수)
    요청_요금 = 과금_요청수 * 0.0000002
    
    총_요금 = gb_초_요금 + 요청_요금
    
    return {
        'GB초': round(gb_초, 2),
        '과금_GB초': round(과금_gb_초, 2),
        '총_요금_USD': round(총_요금, 4)
    }


# 현재 설정 시뮬레이션
현재 = lambda_월_비용(메모리_mb=512, 평균_실행시간_ms=800, 월_호출수=200_000)
최적화 = lambda_월_비용(메모리_mb=256, 평균_실행시간_ms=1200, 월_호출수=200_000)

print(f"현재 설정: {현재}")
print(f"메모리 절반으로 줄인 경우: {최적화}")
print(f"차이: ${현재['총_요금_USD'] - 최적화['총_요금_USD']:.4f}/월")
```

터미널에서 실행합니다:

```bash
python cost_simulator.py
```

예상 출력:
```
현재 설정: {'GB초': 81920.0, '과금_GB초': 0, '총_요금_USD': 0.0}
메모리 절반으로 줄인 경우: {'GB초': 61440.0, '과금_GB초': 0, '총_요금_USD': 0.0}
차이: $0.0000/월
```

> 호출 수가 적으면 무료 티어에 포함됩니다. `월_호출수=2_000_000`으로 바꿔서 차이를 확인해 보세요.

---

### 실습 2 — CloudWatch에서 실제 실행 시간 조회하기

`check_lambda_stats.py` 파일을 만들어 지난 7일간 함수 실행 통계를 가져옵니다.

```python
# check_lambda_stats.py

import boto3
from datetime import datetime, timedelta, timezone

def lambda_통계_조회(함수명: str, 일수: int = 7):
    cloudwatch = boto3.client('cloudwatch', region_name='ap-northeast-2')
    
    종료시각 = datetime.now(timezone.utc)
    시작시각 = 종료시각 - timedelta(days=일수)
    
    지표_목록 = ['Duration', 'Invocations', 'Errors']
    결과 = {}
    
    for 지표 in 지표_목록:
        통계 = cloudwatch.get_metric_statistics(
            Namespace='AWS/Lambda',
            MetricName=지표,
            Dimensions=[{'Name': 'FunctionName', 'Value': 함수명}],
            StartTime=시작시각,
            EndTime=종료시각,
            Period=86400,  # 1일 단위
            Statistics=['Average', 'Sum', 'Maximum']
        )
        결과[지표] = 통계['Datapoints']
    
    return 결과


함수명 = 'my-serverless-app-dev-getUser'  # ← 본인의 함수명으로 변경
통계 = lambda_통계_조회(함수명)

for 지표, 데이터 in 통계.items():
    if 데이터:
        평균 = sum(d.get('Average', 0) for d in 데이터) / len(데이터)
        print(f"{지표}: 평균 {평균:.2f}")
    else:
        print(f"{지표}: 데이터 없음 (함수가 호출되지 않았거나 함수명 확인 필요)")
```

```bash
python check_lambda_stats.py
```

---

### 실습 3 — 예산 알림 자동 설정 스크립트 실행하기

실습 1, 2에서 확인한 비용 구조를 바탕으로, `setup_budget_alert.py`에서 본인 이메일과 한도 금액을 수정한 뒤 실행합니다.

```bash
# AWS 자격증명이 설정되어 있어야 합니다
aws sts get-caller-identity  # 연결 확인

python setup_budget_alert.py
```

예상 출력:
```
예산 알림 생성 완료: 월 $5.0 한도, 80% 도달 시 내이메일@example.com로 알림
```

AWS 콘솔 → **Billing** → **Budgets**에서 생성된 예산을 확인하세요.

---

## 자주 하는 실수

| 실수 | 에러 메시지 또는 증상 | 해결 방법 |
|---|---|---|
| Lambda 타임아웃을 300초로 설정 | 비용 폭증, 버그 함수가 5분간 실행 | 실제 P99 응답 시간의 2~3배로 설정 |
| DynamoDB `Scan` 남용 | `ConsumedCapacity`가 예상보다 10배 이상 | `Query` + GSI(글로벌 보조 인덱스) 사용 |
| CloudWatch 로그 보존 기간 미설정 | 로그 비용이 매달 $2~5씩 쌓임 | `RetentionInDays: 14` 명시 |
| Lambda 핸들러 안에서 DB 클라이언트 생성 | 호출마다 연결 초기화 → 실행 시간 증가 | 핸들러 바깥(전역)에서 클라이언트 생성 |
| `print()` 로 대용량 객체 출력 | `Task timed out` 또는 로그 비용 급증 | `logger.debug()` + `LOG_LEVEL=WARNING` |
| 예산 알림 미설정 | 월말에 예상치 못한 청구서 수령 | AWS Budgets에서 월 $5 알림 설정 |
| API Gateway 캐싱 미사용 | 동일 요청마다 Lambda 호출 → 비용 발생 | GET 엔드포인트에 캐시 TTL 60초 설정 |

---

## 확인 체크리스트

- [ ] Lambda 함수의 메모리 설정이 128MB~512MB 사이에서 시작했다
- [ ] Lambda 타임아웃이 실제 응답 시간의 2~3배로 설정되어 있다
- [ ] DynamoDB 테이블 접근 패턴에 맞게 온디맨드/프로비저닝을 선택했다
- [ ] `Scan` 대신 `Query`를 사용하고 있다
- [ ] CloudWatch 로그 그룹에 `RetentionInDays`가 명시되어 있다
- [ ] Lambda 핸들러 바깥에서 boto3 클라이언트를 생성한다
- [ ] 프로덕션 환경의 `LOG_LEVEL`이 `WARNING` 이상으로 설정되어 있다
- [ ] AWS Budgets에서 월 한도 알림이 설정되어 있다
- [ ] `cost_simulator.py`로 현재 설정의 예상 비용을 계산해 봤다
- [ ] AWS Cost Explorer에서 지난 달 실제 비용 내역을 확인했다

---

## 한 번 더 생각해 보기

1. Lambda 메모리를 128MB에서 256MB로 두 배 올렸더니 실행 시간이 절반으로 줄었습니다. 이 경우 비용은 올라갈까요, 내려갈까요? 어떻게 계산하면 알 수 있을까요?

2. DynamoDB 온디맨드 모드는 트래픽이 갑자기 100배 폭증해도 자동으로 확장됩니다. 이것이 항상 좋은 일일까요? 비용 관점에서 어떤 위험이 있을까요?

3. 무료 티어가 끝나는 12개월 이후에도 같은 아키텍처를 유지한다면 월 비용이 어떻게 달라질까요? 지금부터 준비할 수 있는 것이 있을까요?

---

## 다음 장

다음 장에서는 실제 서버리스 앱을 GitHub Actions로 자동 배포하는 CI/CD 파이프라인을 구축합니다.