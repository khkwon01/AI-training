## 이 장에서 배우는 것

- AWS Lambda의 **콜드 스타트(Cold Start)** 가 무엇인지 이해한다
- 콜드 스타트가 왜 느린지 원인을 파악한다
- 콜드 스타트를 줄이는 실용적인 코드 패턴을 적용한다
- 핸들러 바깥에 초기화 코드를 배치하는 방법을 익힌다
- Provisioned Concurrency와 Warm-up 전략의 차이를 설명할 수 있다

---

## 먼저 쉬운 설명

편의점에서 알바를 한다고 생각해 보세요.

매번 손님이 오면 **유니폼을 입고 → 금전기를 켜고 → 재고를 확인**해야 한다면 얼마나 느릴까요? 손님은 기다리다 지쳐서 가버릴 수도 있어요.

AWS Lambda도 똑같아요. 오랫동안 사용하지 않으면 Lambda 함수가 "잠"들어요. 그러다 요청이 오면 **컨테이너를 실행하고 → Python 런타임을 올리고 → 라이브러리를 불러오고 → 코드를 초기화**해야 하죠. 이 준비 시간을 **콜드 스타트**라고 합니다.

실제 서비스에서는 이 지연이 **수백 밀리초에서 수 초**까지 걸릴 수 있어요. API에 연결된 Lambda라면 사용자는 "왜 이렇게 느려요?"라고 느끼게 됩니다.

이 장에서는 그 준비 시간을 최대한 줄이는 방법을 배웁니다.

---

## 1. 콜드 스타트란 무엇인가

Lambda 함수가 실행될 때 두 가지 경로가 있습니다.

- **웜 스타트(Warm Start)**: 이전 실행 컨테이너가 살아 있어서 바로 핸들러가 호출됨
- **콜드 스타트(Cold Start)**: 새 컨테이너를 만들어야 해서 초기화 단계 전체가 실행됨

```python
# lambda_explain.py — 콜드 스타트 단계를 눈으로 확인하는 예제
import time
import json

# ── 이 아래 코드는 콜드 스타트 때만 실행됩니다 ──
print("🥶 콜드 스타트: 라이브러리 임포트 중...")
import boto3  # 무거운 라이브러리

print("🥶 콜드 스타트: 클라이언트 초기화 중...")
s3_client = boto3.client("s3")  # 네트워크 연결 포함

COLD_START_TIME = time.time()
print(f"🥶 초기화 완료: {COLD_START_TIME}")

# ── 이 아래 코드는 매 요청마다 실행됩니다 ──
def lambda_handler(event, context):
    request_time = time.time()
    elapsed = round((request_time - COLD_START_TIME) * 1000, 2)
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "안녕하세요!",
            "cold_start_age_ms": elapsed
        })
    }
```

> **핵심 규칙**: 핸들러 **바깥**에 있는 코드는 콜드 스타트 때 한 번만 실행됩니다. 핸들러 **안**에 있는 코드는 매 요청마다 실행됩니다.

---

## 2. 초기화 코드를 핸들러 바깥으로 옮기기

가장 효과가 큰 최적화입니다. 데이터베이스 연결, AWS 클라이언트, 설정 파일 로드 등은 핸들러 밖에서 한 번만 실행하세요.

**나쁜 예 — 매 요청마다 연결을 새로 만듦:**

```python
# bad_lambda.py
import boto3
import json

def lambda_handler(event, context):
    # ❌ 요청마다 새 클라이언트를 만든다 — 느리고 낭비!
    dynamodb = boto3.resource("dynamodb")
    table = dynamodb.Table("사용자테이블")
    
    user_id = event.get("user_id", "unknown")
    response = table.get_item(Key={"userId": user_id})
    
    return {
        "statusCode": 200,
        "body": json.dumps(response.get("Item", {}), ensure_ascii=False)
    }
```

**좋은 예 — 클라이언트를 한 번만 만들고 재사용:**

```python
# good_lambda.py
import boto3
import json
import os

# ✅ 콜드 스타트 때 한 번만 실행 — 이후 요청에서 재사용됨
dynamodb = boto3.resource("dynamodb")
TABLE_NAME = os.environ.get("TABLE_NAME", "사용자테이블")
table = dynamodb.Table(TABLE_NAME)

def lambda_handler(event, context):
    user_id = event.get("user_id", "unknown")
    
    # 이미 연결된 테이블 객체를 바로 사용
    response = table.get_item(Key={"userId": user_id})
    
    return {
        "statusCode": 200,
        "body": json.dumps(response.get("Item", {}), ensure_ascii=False)
    }
```

---

## 3. 임포트 최적화 — 필요한 것만 가져오기

라이브러리 전체를 임포트하면 시간이 오래 걸립니다. 필요한 부분만 가져오세요.

```python
# import_optimize.py

# ❌ 느린 방법 — 라이브러리 전체 로드
import boto3

# ✅ 빠른 방법 — 필요한 클라이언트만 직접 지정
from boto3 import client as boto3_client

# ❌ 느린 방법 — pandas 전체 (크고 무거움)
import pandas as pd

# ✅ 대안 — 람다에서 pandas가 꼭 필요한지 재검토하거나
#    Lambda Layer로 분리하거나, 가벼운 대안을 사용
# (csv, json 내장 모듈로 충분한 경우가 많음)
import csv
import json

# ✅ 환경변수는 핸들러 밖에서 읽기
import os
REGION = os.environ.get("AWS_REGION", "ap-northeast-2")
API_KEY = os.environ.get("API_KEY")  # Secrets Manager보다 빠른 접근

# 전역 클라이언트 (재사용)
s3 = boto3_client("s3", region_name=REGION)

def lambda_handler(event, context):
    bucket = event.get("bucket")
    key = event.get("key")
    
    if not bucket or not key:
        return {"statusCode": 400, "body": "bucket과 key가 필요합니다"}
    
    obj = s3.get_object(Bucket=bucket, Key=key)
    content = obj["Body"].read().decode("utf-8")
    
    return {"statusCode": 200, "body": content[:500]}
```

---

## 4. 지연 초기화(Lazy Initialization) 패턴

꼭 필요할 때만 초기화하는 방법도 있습니다. 모든 코드 경로가 해당 리소스를 사용하지 않을 때 유용합니다.

```python
# lazy_init_lambda.py
import json
import os

# 전역 변수로 선언만 해두고 None으로 시작
_secrets_client = None
_db_connection = None

def get_secrets_client():
    """Secrets Manager 클라이언트를 처음 사용할 때만 생성"""
    global _secrets_client
    if _secrets_client is None:
        import boto3
        _secrets_client = boto3.client("secretsmanager")
        print("Secrets Manager 클라이언트 초기화 완료")
    return _secrets_client

def get_db_password():
    """시크릿 값을 가져옴 (캐시 포함)"""
    client = get_secrets_client()
    response = client.get_secret_value(
        SecretId=os.environ.get("SECRET_ARN", "my-db-secret")
    )
    return response["SecretString"]

def lambda_handler(event, context):
    action = event.get("action", "read")
    
    if action == "health":
        # 이 경로는 DB가 필요 없음 — 빠르게 응답
        return {"statusCode": 200, "body": "OK"}
    
    if action == "read_secret":
        # 이 경로에서만 Secrets Manager 클라이언트 생성
        password = get_db_password()
        return {
            "statusCode": 200,
            "body": json.dumps({"loaded": True, "length": len(password)})
        }
    
    return {"statusCode": 400, "body": "알 수 없는 action"}
```

---

## 5. Provisioned Concurrency — 항상 따뜻하게 유지하기

콜드 스타트를 **완전히 없애고 싶다면** Provisioned Concurrency를 사용하세요. AWS가 미리 컨테이너를 준비해 놓습니다.

```yaml
# serverless.yml — Serverless Framework 사용 예시
service: my-api

provider:
  name: aws
  runtime: python3.12
  region: ap-northeast-2

functions:
  api:
    handler: handler.lambda_handler
    memorySize: 512
    timeout: 30
    # ✅ Provisioned Concurrency 설정
    provisionedConcurrency: 3  # 항상 3개 인스턴스를 따뜻하게 유지
    environment:
      TABLE_NAME: 사용자테이블
    events:
      - http:
          path: /users/{id}
          method: get
```

```python
# handler.py — Provisioned Concurrency와 함께 사용
import boto3
import json
import os
import time

# 초기화 시간 측정 (모니터링용)
_init_start = time.time()

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table(os.environ["TABLE_NAME"])

_init_end = time.time()
print(f"초기화 시간: {(_init_end - _init_start) * 1000:.0f}ms")

def lambda_handler(event, context):
    user_id = event["pathParameters"]["id"]
    
    try:
        result = table.get_item(Key={"userId": user_id})
        item = result.get("Item")
        
        if not item:
            return {
                "statusCode": 404,
                "body": json.dumps({"error": "사용자를 찾을 수 없습니다"}, ensure_ascii=False)
            }
        
        return {
            "statusCode": 200,
            "body": json.dumps(item, ensure_ascii=False)
        }
    
    except Exception as e:
        print(f"오류 발생: {e}")
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "서버 오류"}, ensure_ascii=False)
        }
```

> **비용 주의**: Provisioned Concurrency는 사용하지 않아도 비용이 발생합니다. 트래픽이 예측 가능한 프로덕션 환경에서만 사용하세요.

---

## 따라 하기 실습

### 실습 1 — 콜드 스타트 시간 측정하기

`cold_start_measure.py` 파일을 만들고 AWS Lambda에 배포해 보세요.

```python
# cold_start_measure.py
import json
import time
import os

# 초기화 타임스탬프 기록
_INIT_TIME = time.perf_counter()
_CONTAINER_ID = os.environ.get("AWS_REQUEST_ID", "local")

# 무거운 임포트 시뮬레이션
import boto3
_BOTO3_LOADED = time.perf_counter()

s3 = boto3.client("s3")
_S3_READY = time.perf_counter()

def lambda_handler(event, context):
    now = time.perf_counter()
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "boto3_import_ms": round((_BOTO3_LOADED - _INIT_TIME) * 1000, 2),
            "s3_client_ms": round((_S3_READY - _BOTO3_LOADED) * 1000, 2),
            "total_init_ms": round((_S3_READY - _INIT_TIME) * 1000, 2),
            "handler_call_ms": round((now - _S3_READY) * 1000, 2),
            "request_id": context.aws_request_id
        })
    }
```

Lambda 콘솔에서 **테스트 → 실행**하고 결과를 확인하세요. 첫 번째 실행과 두 번째 실행의 `total_init_ms` 값을 비교해 보세요.

---

### 실습 2 — 나쁜 코드를 좋은 코드로 리팩터링하기

아래 `before_refactor.py`를 `after_refactor.py`로 개선해 보세요.

```python
# before_refactor.py — 리팩터링 전
import json

def lambda_handler(event, context):
    import boto3                          # ❌ 핸들러 안에서 임포트
    import os
    
    ssm = boto3.client("ssm")            # ❌ 매 요청마다 클라이언트 생성
    
    param = ssm.get_parameter(
        Name="/my-app/db-host",
        WithDecryption=True
    )
    db_host = param["Parameter"]["Value"]
    
    dynamodb = boto3.resource("dynamodb") # ❌ 매 요청마다 리소스 생성
    table_name = os.environ.get("TABLE_NAME")
    table = dynamodb.Table(table_name)
    
    result = table.scan(Limit=10)
    return {
        "statusCode": 200,
        "body": json.dumps(result["Items"], ensure_ascii=False)
    }
```

```python
# after_refactor.py — 여기에 개선된 코드를 작성하세요
import json
import boto3
import os

# TODO: 전역 클라이언트와 캐시된 파라미터를 여기에 초기화하세요

def lambda_handler(event, context):
    # TODO: 전역 변수를 사용하도록 핸들러를 수정하세요
    pass
```

**확인 방법**: `after_refactor.py`에서 `import` 문과 `boto3.client()` 호출이 핸들러 **바깥**에 있어야 합니다.

---

### 실습 3 — 메모리 크기와 콜드 스타트 관계 실험하기

Lambda 메모리를 다르게 설정하고 초기화 시간을 비교해 보세요.

```python
# memory_experiment.py
import json
import time
import boto3
import os

_INIT_START = time.perf_counter()

# 의도적으로 여러 클라이언트 생성 (무거운 초기화 시뮬레이션)
s3 = boto3.client("s3")
dynamodb = boto3.resource("dynamodb")
ssm = boto3.client("ssm")

_INIT_END = time.perf_counter()
MEMORY_MB = os.environ.get("AWS_LAMBDA_FUNCTION_MEMORY_SIZE", "알 수 없음")

print(f"메모리: {MEMORY_MB}MB, 초기화: {(_INIT_END - _INIT_START)*1000:.0f}ms")

def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps({
            "memory_mb": MEMORY_MB,
            "init_time_ms": round((_INIT_END - _INIT_START) * 1000, 2)
        })
    }
```

**실험 순서**:
1. Lambda 메모리를 **128MB**로 설정 → 배포 → 콜드 스타트 시간 기록
2. 메모리를 **512MB**로 변경 → 재배포 → 콜드 스타트 시간 기록
3. 메모리를 **1024MB**로 변경 → 재배포 → 콜드 스타트 시간 기록

> **팁**: 메모리를 늘리면 CPU도 비례해서 늘어납니다. 콜드 스타트가 눈에 띄게 줄어드는 것을 확인할 수 있어요.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| 핸들러 안에서 `boto3.client()` 반복 호출 | 에러는 없지만 매 요청이 느림 (200~500ms 추가) | 전역 변수로 클라이언트를 핸들러 밖에서 선언 |
| 환경변수를 핸들러 안에서 `os.environ` 반복 호출 | 에러 없음, 미미한 성능 저하 | `_TABLE_NAME = os.environ["TABLE_NAME"]` 으로 전역에서 한 번만 읽기 |
| 전역 변수에 민감 정보 직접 하드코딩 | `AccessDeniedException` 또는 보안 감사 실패 | 환경변수 또는 Secrets Manager 사용 |
| Provisioned Concurrency 설정 후 배포 별칭(alias) 지정 안 함 | `ProvisionedConcurrencyConfigNotFoundException` | 함수 버전 또는 alias에 설정해야 함 (`$LATEST`에는 불가) |
| `import` 안에 무거운 ML 라이브러리 포함 (numpy, scipy 등) | `Init Duration: 8000ms` — 콜드 스타트 8초 이상 | Lambda Layer로 분리하거나 컨테이너 이미지 방식으로 전환 |
| 전역 DB 연결이 타임아웃 후 끊어졌는데 재연결 처리 없음 | `connection was closed in the middle of the query` | 연결 상태 확인 후 재연결하는 `ping()` 패턴 추가 |
| 메모리 128MB로 계속 사용 | `Runtime.ExitError` 또는 함수가 중간에 죽음 | 메모리 부족 → CloudWatch에서 `Max Memory Used` 확인 후 증설 |

---

## 확인 체크리스트

- [ ] `boto3.client()` 또는 `boto3.resource()` 호출이 핸들러 함수 **바깥**에 있다
- [ ] `import` 문이 모두 파일 **최상단**에 있다 (핸들러 안이 아님)
- [ ] 환경변수(`os.environ`)는 전역에서 한 번만 읽는다
- [ ] CloudWatch Logs에서 `Init Duration`을 확인할 수 있다
- [ ] 콜드 스타트와 웜 스타트의 응답 시간 차이를 측정해 보았다
- [ ] 메모리 크기를 바꿔가며 `Init Duration` 변화를 관찰했다
- [ ] Provisioned Concurrency의 장점과 비용 발생 조건을 설명할 수 있다
- [ ] 지연 초기화(Lazy Initialization) 패턴이 언제 유리한지 설명할 수 있다

---

## 한 번 더 생각해 보기

1. **전역 변수에 데이터베이스 연결을 저장하면** Lambda 컨테이너가 재사용될 때마다 같은 연결을 쓰게 됩니다. 그런데 연결이 10분 후 타임아웃으로 끊겼다면 어떻게 될까요? 이 문제를 어떻게 처리하면 좋을까요?

2. Provisioned Concurrency를 24시간 내내 유지하는 것이 항상 좋은 선택일까요? **트래픽이 낮은 새벽 시간대**에는 어떤 전략이 더 효율적일지 생각해 보세요. AWS EventBridge와 함께 사용하면 어떤 조합이 가능할까요?

3. Lambda 함수의 메모리를 늘리면 비용도 올라갑니다. 그런데 메모리를 2배로 늘렸더니 실행 시간이 절반으로 줄었다면, **전체 비용은 늘었을까요 줄었을까요?** Lambda 요금 계산 방식(`GB-초`)을 기반으로 추론해 보세요.

---

## 다음 장

다음 장에서는 Lambda 함수의 실제 성능을 모니터링하는 방법으로 **CloudWatch Metrics와 AWS X-Ray를 활용한 트레이싱**을 배웁니다.