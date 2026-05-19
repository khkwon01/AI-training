## 이 장에서 배우는 것

- Lambda 블루-그린 배포가 무엇인지 이해한다
- Lambda 별칭(alias)과 버전(version)의 차이를 안다
- AWS CLI와 SAM으로 블루-그린 배포를 직접 실행한다
- 트래픽을 점진적으로 전환하는 방법을 실습한다
- 배포 실패 시 즉시 롤백하는 방법을 익힌다

---

## 먼저 쉬운 설명

식당을 운영한다고 상상해 보세요. 메뉴를 완전히 바꾸고 싶을 때 모든 손님을 내보내고 문을 잠근 뒤 바꾸면 큰일이죠. 대신 이렇게 합니다.

1. **파란 주방(Blue)** 에서 지금 요리를 계속 만들고 있습니다.
2. **초록 주방(Green)** 을 새로 준비해 새 메뉴를 완성합니다.
3. 손님 중 **10명만** 초록 주방 음식을 먼저 먹어봅니다.
4. 문제없으면 **모든 손님**을 초록 주방으로 옮깁니다.
5. 만약 문제가 생기면? **즉시 파란 주방**으로 돌아갑니다.

Lambda 블루-그린 배포가 바로 이 방식입니다. 서비스를 멈추지 않고 새 코드를 안전하게 바꿀 수 있습니다.

---

## 1. Lambda 버전(Version)이란?

Lambda 함수를 배포할 때마다 AWS는 **변경 불가능한 스냅샷**을 숫자로 기록합니다. 이것이 버전입니다.

```
내 함수: my-api
  ├── $LATEST  (지금 편집 중인 코드, 항상 최신)
  ├── 1        (지난달 배포, 변경 불가)
  ├── 2        (2주 전 배포, 변경 불가)
  └── 3        (어제 배포, 변경 불가)
```

버전을 게시(publish)하는 명령어입니다.

```bash
# 현재 $LATEST 코드를 새 버전으로 고정
aws lambda publish-version \
  --function-name my-api \
  --description "v2.1 - 결제 버그 수정"
```

응답 예시:

```json
{
  "FunctionName": "my-api",
  "FunctionArn": "arn:aws:lambda:ap-northeast-2:123456789:function:my-api:3",
  "Version": "3",
  "Description": "v2.1 - 결제 버그 수정"
}
```

> **주의:** `$LATEST`는 버전 번호가 아닙니다. 항상 편집 가능한 작업 공간입니다. 블루-그린 배포에는 반드시 번호가 있는 버전을 사용하세요.

---

## 2. Lambda 별칭(Alias)이란?

별칭은 특정 버전을 가리키는 **이름표**입니다. API Gateway나 클라이언트는 버전 번호가 아닌 별칭을 호출하므로, 별칭이 가리키는 버전만 바꾸면 됩니다.

```
클라이언트 → 별칭(production) → 버전 2  ← 현재 파란색(Blue)
                               → 버전 3  ← 곧 초록색(Green)
```

별칭을 만드는 예시입니다.

```bash
# production 별칭을 버전 2로 생성
aws lambda create-alias \
  --function-name my-api \
  --name production \
  --function-version 2 \
  --description "운영 환경"
```

별칭을 업데이트할 때는 `update-alias`를 씁니다.

```bash
# production 별칭을 버전 3으로 전환 (그린으로 완전 이동)
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 3
```

---

## 3. 가중 트래픽 라우팅 (핵심!)

블루-그린의 핵심은 **일부 트래픽만** 새 버전에 먼저 보내는 것입니다. Lambda 별칭은 두 버전에 트래픽 비율을 설정할 수 있습니다.

```
production 별칭
  ├── 버전 2 (Blue)  ← 90% 트래픽
  └── 버전 3 (Green) ← 10% 트래픽  ← 소수 사용자 먼저 테스트
```

```bash
# 10%만 버전 3(Green)으로 보내기
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 2 \
  --routing-config '{"AdditionalVersionWeights": {"3": 0.10}}'
```

문제없으면 비율을 올립니다.

```bash
# 50%로 늘리기
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 2 \
  --routing-config '{"AdditionalVersionWeights": {"3": 0.50}}'

# 100% 전환 (그린 완전 이동, Blue 제거)
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 3 \
  --routing-config '{}'
```

문제가 생기면 즉시 롤백합니다.

```bash
# 롤백: 버전 2(Blue)로 완전 복귀
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 2 \
  --routing-config '{}'
```

---

## 4. SAM으로 블루-그린 배포 자동화

매번 CLI를 직접 치는 것은 번거롭습니다. AWS SAM을 쓰면 `template.yaml` 한 파일로 전략을 선언할 수 있습니다.

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  MyApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: my-api
      Handler: app.lambda_handler
      Runtime: python3.12
      CodeUri: src/
      AutoPublishAlias: production        # 배포 시 자동으로 버전 게시 + 별칭 업데이트
      DeploymentPreference:
        Type: Linear10PercentEvery1Minute # 1분마다 10%씩 트래픽 이동
        Alarms:
          - !Ref DeploymentErrorAlarm     # 에러 감지 시 자동 롤백
        Hooks:
          PreTraffic: !Ref PreTrafficHook # 트래픽 전환 전 검사 함수

  DeploymentErrorAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: my-api-errors
      MetricName: Errors
      Namespace: AWS/Lambda
      Statistic: Sum
      Period: 60
      EvaluationPeriods: 1
      Threshold: 5
      ComparisonOperator: GreaterThanOrEqualToThreshold
      Dimensions:
        - Name: FunctionName
          Value: !Ref MyApiFunction

  PreTrafficHook:
    Type: AWS::Serverless::Function
    Properties:
      Handler: hooks.pre_traffic
      Runtime: python3.12
      CodeUri: src/
      FunctionName: CodeDeployHook_pre_traffic
      Policies:
        - Version: '2012-10-17'
          Statement:
            - Effect: Allow
              Action:
                - codedeploy:PutLifecycleEventHookExecutionStatus
              Resource: '*'
```

훅 함수 예시 (`src/hooks.py`):

```python
import boto3
import os

codedeploy = boto3.client('codedeploy')


def pre_traffic(event, context):
    """트래픽 전환 전 새 버전을 간단히 검사합니다."""
    deployment_id = event['DeploymentId']
    hook_id = event['LifecycleEventHookExecutionId']

    try:
        # 새 버전에 헬스체크 요청
        lambda_client = boto3.client('lambda')
        response = lambda_client.invoke(
            FunctionName=os.environ['NewVersion'],  # SAM이 자동으로 주입
            InvocationType='RequestResponse',
            Payload=b'{"httpMethod": "GET", "path": "/health"}',
        )

        if response['StatusCode'] == 200:
            status = 'Succeeded'
        else:
            status = 'Failed'

    except Exception as e:
        print(f"헬스체크 실패: {e}")
        status = 'Failed'

    codedeploy.put_lifecycle_event_hook_execution_status(
        deploymentId=deployment_id,
        lifecycleEventHookExecutionId=hook_id,
        status=status,
    )
```

배포 명령어:

```bash
# 빌드 후 배포
sam build
sam deploy --guided  # 처음 한 번만 --guided 사용
```

---

## 5. CodeDeploy 배포 전략 종류

SAM `DeploymentPreference.Type`에 넣을 수 있는 전략들입니다.

| 전략 이름 | 설명 | 언제 사용? |
|---|---|---|
| `AllAtOnce` | 한 번에 100% 전환 | 개발/테스트 환경 |
| `Linear10PercentEvery1Minute` | 1분마다 10%씩 증가 | 일반 운영 배포 |
| `Linear10PercentEvery10Minutes` | 10분마다 10%씩 증가 | 중요 서비스 배포 |
| `Canary10Percent5Minutes` | 10%를 5분간 유지 후 100% | 빠른 카나리 테스트 |
| `Canary10Percent30Minutes` | 10%를 30분간 유지 후 100% | 긴 카나리 테스트 |

```yaml
# template.yaml 에서 전략 바꾸기
DeploymentPreference:
  Type: Canary10Percent5Minutes  # 여기만 바꾸면 됩니다
```

---

## 따라 하기 실습

### 실습 1 — 기존 Lambda 함수에 버전과 별칭 달기

이미 `my-api`라는 함수가 있다고 가정합니다.

```bash
# 1. 현재 $LATEST를 버전으로 게시
aws lambda publish-version \
  --function-name my-api \
  --description "Blue: 안정 버전"

# 출력에서 "Version": "1" 확인

# 2. production 별칭을 버전 1에 연결
aws lambda create-alias \
  --function-name my-api \
  --name production \
  --function-version 1

# 3. 별칭 ARN 확인
aws lambda get-alias \
  --function-name my-api \
  --name production
```

예상 출력:

```json
{
  "AliasArn": "arn:aws:lambda:ap-northeast-2:123456789:function:my-api:production",
  "Name": "production",
  "FunctionVersion": "1"
}
```

---

### 실습 2 — 새 코드 배포 후 10% 트래픽만 전환하기

```bash
# 1. src/app.py 를 수정한 뒤 Lambda에 업로드
zip -r function.zip src/
aws lambda update-function-code \
  --function-name my-api \
  --zip-file fileb://function.zip

# 2. 새 코드를 버전 2로 게시
aws lambda publish-version \
  --function-name my-api \
  --description "Green: 신규 기능"

# 3. production 별칭에서 10%를 버전 2로 보내기
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 1 \
  --routing-config '{"AdditionalVersionWeights": {"2": 0.10}}'

# 4. CloudWatch에서 에러율 모니터링 (1~2분 대기)
aws cloudwatch get-metric-statistics \
  --namespace AWS/Lambda \
  --metric-name Errors \
  --dimensions Name=FunctionName,Value=my-api Name=Resource,Value=my-api:production \
  --start-time $(date -u -v-5M +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ) \
  --period 60 \
  --statistics Sum
```

에러가 없으면 실습 3으로 넘어갑니다.

---

### 실습 3 — 완전 전환 또는 롤백

**시나리오 A: 정상 — 버전 2로 완전 전환**

```bash
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 2 \
  --routing-config '{}'

echo "배포 완료! production → 버전 2 (Green)"
```

**시나리오 B: 문제 발생 — 버전 1로 즉시 롤백**

```bash
aws lambda update-alias \
  --function-name my-api \
  --name production \
  --function-version 1 \
  --routing-config '{}'

echo "롤백 완료! production → 버전 1 (Blue)"
```

---

## 자주 하는 실수

| 실수 | 오류 메시지 / 증상 | 해결 방법 |
|---|---|---|
| `$LATEST`를 별칭에 바로 사용 | `$LATEST is not supported for an alias pointing to more than one version` | 반드시 `publish-version`으로 번호를 먼저 게시한 뒤 사용 |
| 가중치 합계가 1을 초과 | `ValidationException: The sum of all traffic weights must be less than 1` | `AdditionalVersionWeights`의 값은 0.0~0.99 사이여야 함. 주 버전은 자동으로 나머지를 가짐 |
| SAM 훅 함수 이름 규칙 위반 | `CodeDeploy hook function name must start with 'CodeDeployHook_'` | 훅 함수의 `FunctionName`을 `CodeDeployHook_`으로 시작하도록 변경 |
| 버전 게시 전 코드 업로드 누락 | 별칭이 이전 코드를 계속 실행함 | `update-function-code` → `publish-version` 순서를 지킬 것 |
| 알람 없이 자동 롤백 기대 | 에러가 나도 트래픽 전환이 계속 진행됨 | `template.yaml`의 `Alarms` 필드에 CloudWatch Alarm ARN 등록 필수 |
| 롤백 후 `routing-config` 미삭제 | 롤백 후에도 일부 트래픽이 오류 버전으로 계속 전달됨 | 롤백 시 `--routing-config '{}'`를 반드시 함께 지정 |

---

## 확인 체크리스트

- [ ] Lambda 버전(Version)과 `$LATEST`의 차이를 설명할 수 있다
- [ ] Lambda 별칭(Alias)이 왜 버전 번호 대신 쓰이는지 말할 수 있다
- [ ] `publish-version` 명령어를 실행하고 버전 번호를 확인했다
- [ ] `create-alias` / `update-alias` 명령어로 별칭을 만들고 수정했다
- [ ] `routing-config`로 10% 트래픽을 새 버전에 보내는 설정을 해봤다
- [ ] CloudWatch 에러 알람을 `template.yaml`에 연결했다
- [ ] 롤백 명령어를 직접 실행하여 이전 버전으로 돌아왔다
- [ ] SAM `DeploymentPreference.Type`을 최소 두 가지 전략으로 바꿔봤다

---

## 한 번 더 생각해 보기

1. **트래픽을 10%가 아니라 1%부터 시작해야 할 때는 언제일까요?** 사용자 수가 매우 많은 서비스와 적은 서비스에서 각각 어떤 기준으로 초기 비율을 정하면 좋을지 생각해 보세요.

2. **PreTraffic 훅이 실패했을 때 AWS는 어떻게 행동할까요?** 훅이 `Failed`를 반환하면 트래픽 전환이 시작되기 전에 자동 롤백이 발생합니다. 이 덕분에 어떤 위험을 사전에 막을 수 있을까요?

3. **블루-그린 배포가 항상 정답은 아닙니다.** 데이터베이스 스키마를 바꾸는 마이그레이션이 동시에 필요한 경우, 두 버전의 Lambda가 같은 테이블을 동시에 쓴다면 어떤 문제가 생길까요? 어떻게 해결할 수 있을지 아이디어를 적어보세요.

---

## 다음 장

다음 장에서는 Lambda 함수의 동시 실행 수(Concurrency)를 제어하고 예약 동시성(Provisioned Concurrency)을 설정하여 콜드 스타트 문제를 해결하는 방법을 배웁니다.