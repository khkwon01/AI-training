# Chapter 20: Zero Downtime 배포

## 이 장에서 배우는 것

- Lambda 함수 버전(Version)과 별칭(Alias)의 개념과 역할
- 블루/그린(Blue/Green) 배포로 서비스 중단 없이 업데이트하는 방법
- 카나리(Canary) 배포로 트래픽을 점진적으로 전환하는 방법
- AWS CodeDeploy를 활용한 자동 트래픽 전환 설정
- 배포 실패 시 자동 롤백(Rollback)을 구성하는 방법
- CloudWatch 경보를 배포 안전망으로 연결하는 방법

---

## 먼저 쉬운 설명

여러분이 편의점을 운영한다고 상상해 보세요. 인테리어 공사를 하려면 가게 문을 닫아야 할까요?

영리한 사장님은 이렇게 합니다. 옆 공간에 새 가게를 먼저 다 꾸며 놓고, 손님 10명 중 1명만 새 가게로 안내해 봅니다. 문제가 없으면 조금씩 더 많이 보내다가, 결국 모든 손님을 새 가게로 옮깁니다. 구 가게는 그때 철거하면 됩니다.

Lambda 무중단 배포도 정확히 이 원리입니다. 새 코드를 올렸다고 바로 모든 트래픽을 보내지 않습니다. 버전과 별칭이라는 도구로 "어느 코드에 얼마만큼 트래픽을 보낼지"를 정밀하게 제어합니다. 결과적으로 사용자는 배포가 일어나는지조차 모릅니다.

---

## 1. Lambda 버전과 별칭 이해하기

Lambda 함수를 배포할 때마다 **버전**이 숫자로 쌓입니다. `$LATEST`는 항상 가장 최신 코드를 가리키는 특수 포인터입니다. **별칭(Alias)**은 특정 버전에 붙이는 이름표이며, 트래픽 비율 설정이 가능합니다.

```bash
# 현재 $LATEST 코드를 새 버전으로 게시
aws lambda publish-version \
  --function-name my-order-api \
  --description "v2: 주문 처리 로직 개선"

# 출력 예시
# {
#   "FunctionName": "my-order-api",
#   "Version": "5",
#   "Description": "v2: 주문 처리 로직 개선"
# }

# 'production' 별칭이 버전 5를 가리키도록 설정
aws lambda update-alias \
  --function-name my-order-api \
  --name production \
  --function-version 5
```

```python
# handler.py — 버전 확인용 핸들러
import json
import os

def lambda_handler(event, context):
    # context.function_version 으로 실행 중인 버전 확인 가능
    version = context.function_version
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "주문 처리 완료",
            "deployed_version": version
        })
    }
```

> **핵심 규칙**: API Gateway나 다른 서비스는 `$LATEST`가 아닌 **별칭**을 바라보도록 설정하세요. 별칭만 바꾸면 트리거 설정을 건드리지 않아도 됩니다.

---

## 2. 블루/그린 배포 — 즉시 전환

블루(현재 운영) 버전을 그대로 유지한 채, 그린(새 버전)을 완전히 준비한 뒤 별칭 하나만 바꿔 전환하는 방식입니다. 전환이 순간적으로 일어나므로 "빅뱅 전환"이라고도 합니다.

```bash
# 1단계: 새 코드 배포 후 버전 게시
aws lambda update-function-code \
  --function-name my-order-api \
  --zip-file fileb://function.zip

aws lambda publish-version \
  --function-name my-order-api \
  --description "그린: 결제 모듈 v3"
# → 버전 6 생성됨

# 2단계: 'staging' 별칭으로 그린 버전 테스트
aws lambda create-alias \
  --function-name my-order-api \
  --name staging \
  --function-version 6

# 3단계: 테스트 통과 후 production 별칭을 버전 6으로 전환
aws lambda update-alias \
  --function-name my-order-api \
  --name production \
  --function-version 6

# 4단계: 롤백이 필요하면 이전 버전(5)으로 되돌리기
aws lambda update-alias \
  --function-name my-order-api \
  --name production \
  --function-version 5
```

```python
# test_green.py — 그린 버전 스모크 테스트
import boto3

client = boto3.client("lambda", region_name="ap-northeast-2")

def smoke_test():
    response = client.invoke(
        FunctionName="my-order-api:staging",  # staging 별칭 호출
        InvocationType="RequestResponse",
        Payload=b'{"test": true}'
    )
    status = response["StatusCode"]
    assert status == 200, f"예상치 못한 상태 코드: {status}"
    print("스모크 테스트 통과 ✓")

smoke_test()
```

---

## 3. 카나리 배포 — 점진적 트래픽 전환

별칭 하나에 두 버전의 트래픽 **가중치(weight)**를 지정할 수 있습니다. 5% → 20% → 50% → 100% 식으로 단계를 밟아 전환합니다.

```bash
# production 별칭을 카나리 모드로 설정
# 버전 5(구) 95%, 버전 6(신) 5%
aws lambda update-alias \
  --function-name my-order-api \
  --name production \
  --function-version 5 \
  --routing-config AdditionalVersionWeights={"6"=0.05}

# 이상 없으면 50%로 확대
aws lambda update-alias \
  --function-name my-order-api \
  --name production \
  --function-version 5 \
  --routing-config AdditionalVersionWeights={"6"=0.50}

# 완전 전환 (routing-config 제거 = 100%)
aws lambda update-alias \
  --function-name my-order-api \
  --name production \
  --function-version 6 \
  --routing-config AdditionalVersionWeights={}
```

```python
# canary_monitor.py — 오류율 모니터링 후 자동 롤백
import boto3
import time

cloudwatch = boto3.client("cloudwatch", region_name="ap-northeast-2")
lambda_client = boto3.client("lambda", region_name="ap-northeast-2")

FUNCTION_NAME = "my-order-api"
OLD_VERSION = "5"
NEW_VERSION = "6"
ERROR_THRESHOLD = 1.0  # 오류율 1% 초과 시 롤백

def get_error_rate(alias: str) -> float:
    """최근 5분간 오류율(%) 반환"""
    response = cloudwatch.get_metric_statistics(
        Namespace="AWS/Lambda",
        MetricName="Errors",
        Dimensions=[
            {"Name": "FunctionName", "Value": FUNCTION_NAME},
            {"Name": "Resource", "Value": f"{FUNCTION_NAME}:{alias}"},
        ],
        StartTime=time.time() - 300,
        EndTime=time.time(),
        Period=300,
        Statistics=["Sum"],
    )
    errors = sum(d["Sum"] for d in response["Datapoints"])

    response_inv = cloudwatch.get_metric_statistics(
        Namespace="AWS/Lambda",
        MetricName="Invocations",
        Dimensions=[
            {"Name": "FunctionName", "Value": FUNCTION_NAME},
            {"Name": "Resource", "Value": f"{FUNCTION_NAME}:{alias}"},
        ],
        StartTime=time.time() - 300,
        EndTime=time.time(),
        Period=300,
        Statistics=["Sum"],
    )
    invocations = sum(d["Sum"] for d in response_inv["Datapoints"])
    return (errors / invocations * 100) if invocations > 0 else 0.0

def rollback():
    print("오류율 초과 — 롤백 실행 중...")
    lambda_client.update_alias(
        FunctionName=FUNCTION_NAME,
        Name="production",
        FunctionVersion=OLD_VERSION,
        RoutingConfig={"AdditionalVersionWeights": {}},
    )
    print(f"롤백 완료: 버전 {OLD_VERSION} 으로 복구됨")

# 카나리 단계별 진행
for weight in [0.05, 0.20, 0.50, 1.0]:
    print(f"\n트래픽 {int(weight * 100)}% 를 버전 {NEW_VERSION} 으로 전환")
    if weight < 1.0:
        lambda_client.update_alias(
            FunctionName=FUNCTION_NAME,
            Name="production",
            FunctionVersion=OLD_VERSION,
            RoutingConfig={"AdditionalVersionWeights": {NEW_VERSION: weight}},
        )
    else:
        lambda_client.update_alias(
            FunctionName=FUNCTION_NAME,
            Name="production",
            FunctionVersion=NEW_VERSION,
            RoutingConfig={"AdditionalVersionWeights": {}},
        )

    print("5분 대기 후 오류율 확인...")
    time.sleep(300)

    error_rate = get_error_rate("production")
    print(f"현재 오류율: {error_rate:.2f}%")

    if error_rate > ERROR_THRESHOLD:
        rollback()
        break
    print(f"정상 범위 — 다음 단계로 진행")
```

---

## 4. AWS CodeDeploy로 배포 자동화하기

매번 CLI를 손으로 치는 건 번거롭고 실수하기 쉽습니다. **CodeDeploy**를 쓰면 SAM/CloudFormation 설정 한 줄로 카나리·리니어 배포가 자동화됩니다.

```yaml
# template.yaml — SAM 템플릿
AWSTemplateFormatVersion: "2010-09-09"
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Runtime: python3.12
    Timeout: 30

Resources:
  OrderFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: my-order-api
      Handler: handler.lambda_handler
      CodeUri: src/
      AutoPublishAlias: production        # 배포 시 자동으로 버전 게시 + 별칭 갱신
      DeploymentPreference:
        Type: Canary10Percent5Minutes     # 10% → 5분 대기 → 90% 전환
        Alarms:
          - !Ref OrderFunctionErrorAlarm  # 경보 발생 시 자동 롤백
        Hooks:
          PreTraffic: !Ref PreTrafficHook # 전환 전 실행할 테스트 함수

  # 배포 실패 감지 경보
  OrderFunctionErrorAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: OrderFunction-HighErrorRate
      MetricName: Errors
      Namespace: AWS/Lambda
      Dimensions:
        - Name: FunctionName
          Value: !Ref OrderFunction
        - Name: Resource
          Value: !Sub "${OrderFunction}:production"
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 2
      Threshold: 5          # 2분 내 오류 5회 초과 시 롤백
      ComparisonOperator: GreaterThanThreshold

  # 트래픽 전환 전 실행되는 훅 함수
  PreTrafficHook:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: CodeDeployHook_PreTraffic_my-order-api
      Handler: hooks.pre_traffic
      CodeUri: src/
      Policies:
        - Version: "2012-10-17"
          Statement:
            - Effect: Allow
              Action:
                - codedeploy:PutLifecycleEventHookExecutionStatus
              Resource: "*"
            - Effect: Allow
              Action:
                - lambda:InvokeFunction
              Resource: !Ref OrderFunction.Version
```

```python
# src/hooks.py — PreTraffic 훅
import boto3
import json
import os

codedeploy = boto3.client("codedeploy")
lambda_client = boto3.client("lambda")

def pre_traffic(event, context):
    """새 버전 트래픽 전환 전 자동 실행되는 검증 함수"""
    deployment_id = event["DeploymentId"]
    lifecycle_event_hook_execution_id = event["LifecycleEventHookExecutionId"]

    # 새 버전 ARN 가져오기 (환경변수로 SAM이 자동 주입)
    new_version_arn = os.environ["NewVersion"]

    try:
        # 새 버전 직접 호출하여 검증
        response = lambda_client.invoke(
            FunctionName=new_version_arn,
            InvocationType="RequestResponse",
            Payload=json.dumps({"healthcheck": True}).encode(),
        )
        payload = json.loads(response["Payload"].read())
        assert response["StatusCode"] == 200
        assert payload.get("statusCode") == 200

        # 훅 성공 보고
        codedeploy.put_lifecycle_event_hook_execution_status(
            deploymentId=deployment_id,
            lifecycleEventHookExecutionId=lifecycle_event_hook_execution_id,
            status="Succeeded",
        )
        print("PreTraffic 훅 통과 — 트래픽 전환 시작")

    except Exception as e:
        print(f"PreTraffic 훅 실패: {e}")
        # 훅 실패 보고 → CodeDeploy가 자동 롤백
        codedeploy.put_lifecycle_event_hook_execution_status(
            deploymentId=deployment_id,
            lifecycleEventHookExecutionId=lifecycle_event_hook_execution_id,
            status="Failed",
        )
```

---

## 5. CodeDeploy 배포 전략 종류 한눈에 보기

| 전략 이름 | 동작 방식 | 언제 쓰나 |
|---|---|---|
| `AllAtOnce` | 즉시 100% 전환 | 개발/테스트 환경 |
| `Canary10Percent5Minutes` | 10% → 5분 대기 → 90% | 일반 운영 서비스 |
| `Canary10Percent30Minutes` | 10% → 30분 대기 → 90% | 트래픽이 적은 서비스 |
| `Linear10PercentEvery1Minute` | 매 1분마다 10%씩 선형 증가 | 빠른 점진 배포 |
| `Linear10PercentEvery10Minutes` | 매 10분마다 10%씩 선형 증가 | 안전 최우선 배포 |

```bash
# sam deploy 한 줄로 위 전략이 모두 자동 실행됨
sam deploy \
  --stack-name my-order-api-stack \
  --s3-bucket my-deploy-bucket \
  --capabilities CAPABILITY_IAM \
  --region ap-northeast-2
```

---

## 따라 하기 실습

### 실습 1 — 버전과 별칭으로 블루/그린 배포 체험하기

**목표**: 버전 1(블루)에서 버전 2(그린)로 무중단 전환 후 롤백까지 경험합니다.

```bash
# 1. 프로젝트 폴더 생성
mkdir zero-downtime-lab && cd zero-downtime-lab

# 2. 블루 버전 코드 작성
cat > handler_v1.py << 'EOF'
import json

def lambda_handler(event, context):
    return {"statusCode": 200, "body": json.dumps({"version": "blue", "msg": "구 버전입니다"})}
EOF

# 3. zip 패키징 후 Lambda 함수 생성 (함수가 이미 있으면 update)
zip function_v1.zip handler_v1.py

aws lambda create-function \
  --function-name zero-downtime-demo \
  --runtime python3.12 \
  --role arn:aws:iam::YOUR_ACCOUNT_ID:role/lambda-basic-role \
  --handler handler_v1.lambda_handler \
  --zip-file fileb://function_v1.zip

# 4. 버전 1 게시
aws lambda publish-version \
  --function-name zero-downtime-demo \
  --description "블루: 초기 버전"

# 5. production 별칭 생성 (버전 1 가리킴)
aws lambda create-alias \
  --function-name zero-downtime-demo \
  --name production \
  --function-version 1

# 6. 호출 확인
aws lambda invoke \
  --function-name zero-downtime-demo:production \
  --payload '{}' \
  output.json && cat output.json
# 예상 출력: {"version": "blue", "msg": "구 버전입니다"}
```

```bash
# 7. 그린 버전 코드 작성
cat > handler_v2.py << 'EOF'
import json

def lambda_handler(event, context):
    return {"statusCode": 200, "body": json.dumps({"version": "green", "msg": "신 버전입니다"})}
EOF

zip function_v2.zip handler_v2.py

# 8. 코드 업데이트 후 버전 2 게시
aws lambda update-function-code \
  --function-name zero-downtime-demo \
  --zip-file fileb://function_v2.zip

aws lambda publish-version \
  --function-name zero-downtime-demo \
  --description "그린: 신 버전"

# 9. production 별칭을 버전 2로 전환
aws lambda update-alias \
  --function-name zero-downtime-demo \
  --name production \
  --function-version 2

# 10. 전환 확인
aws lambda invoke \
  --function-name zero-downtime-demo:production \
  --payload '{}' \
  output.json && cat output.json
# 예상 출력: {"version": "green", "msg": "신 버전입니다"}

# 11. 문제 발생 가정 — 버전 1로 롤백
aws lambda update-alias \
  --function-name zero-downtime-demo \
  --name production \
  --function-version 1

aws lambda invoke \
  --function-name zero-downtime-demo:production \
  --payload '{}' \
  output.json && cat output.json
# 다시 blue 버전으로 복구 확인
```

---

### 실습 2 — 카나리 배포로 트래픽 5%만 신 버전으로 보내기

실습 1에 이어서 진행합니다. 버전 1을 구 버전, 버전 2를 신 버전으로 사용합니다.

```python
# canary_test.py — 100회 호출 후 버전 분포 확인
import boto3
import json
from collections import Counter

client = boto3.client("lambda", region_name="ap-northeast-2")

def run_canary_test(alias: str, call_count: int = 100):
    versions = []
    for i in range(call_count):
        response = client.invoke(
            FunctionName=f"zero-downtime-demo:{alias}",
            InvocationType="RequestResponse",
            Payload=b"{}",
        )
        body = json.loads(response["Payload"].read())
        inner = json.loads(body["body"])
        versions.append(inner["version"])

    dist = Counter(versions)
    print(f"\n{call_count}회 호출 결과:")
    for ver, cnt in dist.items():
        print(f"  {ver}: {cnt}회 ({cnt/call_count*100:.1f}%)")

# 5% 카나리 설정
client.update_alias(
    FunctionName="zero-downtime-demo",
    Name="production",
    FunctionVersion="1",               # 메인 버전
    RoutingConfig={"AdditionalVersionWeights": {"2": 0.05}},  # 5%만 신 버전
)
print("카나리 5% 설정 완료")
run_canary_test("production", 200)
# 기대값: blue ≈ 190회(95%), green ≈ 10회(5%)

# 완전 전환
client.update_alias(
    FunctionName="zero-downtime-demo",
    Name="production",
    FunctionVersion="2",
    RoutingConfig={"AdditionalVersionWeights": {}},
)
print("\n100% 전환 완료")
run_canary_test("production", 20)
# 기대값: green 20회(100%)
```

---

### 실습 3 — SAM + CodeDeploy 자동화 배포 파이프라인 구성하기

실습 1~2에서 배운 개념을 SAM으로 코드화합니다.

```
zero-downtime-lab/
├── template.yaml       # SAM 템플릿 (4번 절 예제)
├── src/
│   ├── handler.py      # 실제 Lambda 핸들러
│   └── hooks.py        # PreTraffic 훅 (4번 절 예제)
└── samconfig.toml      # 배포 파라미터 저장
```

```toml
# samconfig.toml
version = 0.1

[default.deploy.parameters]
stack_name        = "zero-downtime-demo-stack"
s3_bucket         = "my-sam-deploy-bucket"
region            = "ap-northeast-2"
capabilities      = "CAPABILITY_IAM"
confirm_changeset = false
```

```python
# src/handler.py
import json

def lambda_handler(event, context):
    # 헬스체크 요청 처리
    if event.get("healthcheck"):
        return {"statusCode": 200, "body": json.dumps({"healthy": True})}

    return {
        "statusCode": 200,
        "body": json.dumps({"message": "주문 처리 완료", "version": "3"}),
    }
```

```bash
# 배포 실행 — CodeDeploy가 Canary10Percent5Minutes 전략으로 자동 진행
sam build && sam deploy

# 배포 진행 상황 확인
aws deploy list-deployments \
  --application-name my-order-api \
  --deployment-group-name my-order-api-DeploymentGroup \
  --query 'deployments[0]' \
  --output text | xargs -I{} aws deploy get-deployment --deployment-id {}
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 / 증상 | 해결 방법 |
|---|---|---|
| `$LATEST`에 직접 API Gateway 연결 | 배포할 때마다 `$LATEST`가 바뀌어 의도치 않은 코드 노출 | API Gateway는 반드시 **별칭(Alias)**을 바라보도록 설정 |
| 버전 게시 없이 별칭 업데이트 시도 | `ResourceNotFoundException: Function not found: arn:...:42` | `publish-version` 먼저 실행 후 반환된 버전 번호로 별칭 업데이트 |
| 카나리 롤백 시 `RoutingConfig` 미제거 | 롤백 후에도 신 버전으로 일부 트래픽 계속 유입 | 롤백 시 `--routing-config AdditionalVersionWeights={}` 명시 |
| PreTraffic 훅에 `codedeploy:PutLifecycleEventHookExecutionStatus` 권한 누락 | `AccessDeniedException: ... is not authorized to perform: codedeploy:PutLifecycleEventHookExecutionStatus` | IAM 역할에 해당 Action 추가 |
| CloudWatch 경보 기준 너무 낮게 설정 | 정상 트래픽 스파이크에도 롤백 발생 | `EvaluationPeriods` 를 2 이상으로 설정하고 `Threshold` 는 평소 오류율 3~5배 기준으로 설정 |
| `AdditionalVersionWeights` 가중치 합이 1 초과 | `InvalidParameterValueException: The sum of ... must not exceed 1` | 두 버전 가중치의 합이 정확히 1.0 이하여야 함 (메인 버전 가중치는 나머지를 자동 계산) |
| SAM `DeploymentPreference` 없이 `AutoPublishAlias` 만 설정 | 배포는 되지만 카나리/롤백 없이 즉시 전환됨 | `DeploymentPreference.Type` 을 반드시 명시 |

---

## 확인 체크리스트

- [ ] Lambda 버전과 별칭의 차이를 설명할 수 있다
- [ ] `publish-version` 명령으로 새 버전을 게시할 수 있다
- [ ] `update-alias` 로 별칭이 가리키는 버전을 바꿀 수 있다
- [ ] `routing-config AdditionalVersionWeights` 로 트래픽 비율을 설정할 수 있다
- [ ] 카나리 배포 중 오류 발생 시 이전 버전으로 롤백하는 CLI 명령을 작성할 수 있다
- [ ] SAM 템플릿에서 `AutoPublishAlias` 와 `DeploymentPreference` 를 함께 설정할 수 있다
- [ ] CloudWatch 경보를 CodeDeploy 롤백 트리거로 연결할 수 있다
- [ ] PreTraffic 훅 함수가 실패했을 때 어떤 일이 발생하는지 설명할 수 있다

---

## 한 번 더 생각해 보기

1. **블루/그린 vs 카나리**: 결제처럼 데이터 정합성이 중요한 기능을 배포할 때, 두 전략 중 어느 것이 더 안전할까요? 그 이유는 무엇인가요?

2. **PreTraffic 훅의 한계**: PreTraffic 훅은 트래픽 전환 전에 실행되지만, 실제 사용자 트래픽을 받아보기 전에는 발견하기 어려운 버그 유형이 있습니다. 어떤 종류의 버그가 그럴 수 있을까요?

3. **비용과 안전의 균형**: `Linear10PercentEvery1Minute` 전략은 10분에 걸쳐 배포가 완료됩니다. 배포 시간이 길어지면 어떤 비용이나 위험이 생길 수 있을까요? 반대로 `AllAtOnce` 를 쓰면 안 되는 이유는 무엇인가요?

---

## 다음 장

다음 장에서는 Lambda 함수의 콜드 스타트를 줄이고 응답 속도를 높이는 **프로비저닝된 동시성(Provisioned Concurrency)** 설정을 배웁니다.