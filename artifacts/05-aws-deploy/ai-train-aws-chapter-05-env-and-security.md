# ai-train AWS Chapter 05: 환경 변수와 보안 설정

## 이 장에서 배우는 것

- Lambda에서 환경 변수를 설정하고 읽는 방법
- 코드에 직접 넣으면 안 되는 값들
- AWS Secrets Manager로 민감한 값을 안전하게 관리하는 방법
- IAM 권한을 최소화하는 이유와 방법
- 초보자가 자주 하는 보안 실수와 예방법

---

## 먼저 쉬운 설명

코드에 이런 값을 직접 쓰면 안 된다.

```python
# 절대 이렇게 하지 말 것
API_KEY = "sk-abc123def456"
DB_PASSWORD = "my_secret_password"
```

GitHub에 올리는 순간 전 세계에 공개된다. 악의적인 봇이 수분 안에 이 값을 수집해서 악용할 수 있다.

**환경 변수**는 코드 외부에 설정값을 저장하는 방법이다. 코드는 값을 직접 알지 못하고, 실행 환경에서 주입받는다.

---

## 1. Lambda 환경 변수 설정

### 콘솔에서 설정

1. Lambda 함수 → **Configuration** 탭 → **Environment variables**
2. **Edit** 클릭
3. **Add environment variable**:
   - Key: `GREETING_PREFIX`
   - Value: `안녕하세요`
4. **Save**

### 코드에서 읽기

```python
import os

def lambda_handler(event, context):
    # 환경 변수 읽기
    prefix = os.environ.get("GREETING_PREFIX", "Hello")  # 기본값 "Hello"
    name = event.get("queryStringParameters", {}).get("name", "World")

    return {
        "statusCode": 200,
        "body": f"{prefix}, {name}님!"
    }
```

`os.environ.get("KEY", "기본값")` 패턴을 쓰면 환경 변수가 없을 때도 안전하게 동작한다.

---

## 2. 코드에 넣으면 안 되는 값들

| 종류 | 예시 | 위험 |
|------|------|------|
| API 키 | `sk-abc123...` | 타인이 내 계정으로 API 호출 |
| 데이터베이스 비밀번호 | `myPassword123` | DB 무단 접근 |
| AWS Access Key | `AKIAIOSFODNN7EXAMPLE` | AWS 리소스 무단 사용, 요금 폭탄 |
| JWT Secret | `my-jwt-secret` | 인증 우회 가능 |
| 개인 이메일/전화번호 | `user@email.com` | 개인정보 노출 |

### .gitignore로 실수 방지

로컬 개발 시 환경 변수를 `.env` 파일에 저장하고, `.gitignore`에 추가한다.

```bash
# .env 파일 (절대 git에 올리지 않음)
API_KEY=sk-abc123def456
DB_PASSWORD=my_secret_password
```

```
# .gitignore
.env
*.env
```

---

## 3. Lambda에서 민감한 값 안전하게 관리하기

### 방법 1: Lambda 환경 변수 (단순 설정값)

개발/테스트 환경 구분, 서비스 URL, 기능 플래그 등에 적합.

```python
ENVIRONMENT = os.environ.get("ENVIRONMENT", "dev")
MAX_MEMOS = int(os.environ.get("MAX_MEMOS", "100"))
```

### 방법 2: AWS Secrets Manager (비밀값)

API 키, 비밀번호처럼 노출되면 안 되는 값에 사용한다.

**Secrets Manager에 값 저장:**
1. AWS 콘솔 → **Secrets Manager** → **Store a new secret**
2. Secret type: **Other type of secret**
3. Key/Value: `api_key` / `sk-abc123def456`
4. Secret name: `memo-service/api-key`

**Lambda에서 읽기:**
```python
import boto3
import json

def get_secret(secret_name):
    client = boto3.client("secretsmanager", region_name="ap-northeast-2")
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response["SecretString"])

# Lambda 시작 시 한 번만 로드 (핸들러 밖)
secrets = get_secret("memo-service/api-key")

def lambda_handler(event, context):
    api_key = secrets["api_key"]
    # ...
```

> 초보자는 우선 Lambda 환경 변수로 충분하다. Secrets Manager는 팀 프로젝트나 실제 서비스에서 사용한다.

---

## 4. IAM 권한 최소화

Lambda 함수에는 실행 역할(execution role)이 있다. 이 역할에 필요한 권한만 부여해야 한다.

### 왜 최소 권한인가

Lambda가 탈취되거나 버그가 있을 때, 권한이 적을수록 피해가 작다.

### 확인 방법

Lambda → Configuration → **Permissions** → Role name 클릭

현재 부여된 정책 목록이 표시된다. 불필요한 정책은 제거한다.

### 초보자 실습 수준에서 필요한 권한

| 작업 | 필요한 정책 |
|------|-----------|
| CloudWatch 로그 쓰기 | `AWSLambdaBasicExecutionRole` (기본 포함) |
| DynamoDB 읽기/쓰기 | `AmazonDynamoDBFullAccess` (또는 세부 지정) |
| Secrets Manager 읽기 | `SecretsManagerReadWrite` (또는 세부 지정) |

`AdministratorAccess`는 절대 Lambda에 부여하지 않는다.

---

## 5. 따라 하기 실습

### 실습 1. 환경 변수로 서비스 설정 관리하기

`memo-service` Lambda에 환경 변수 추가:

| Key | Value |
|-----|-------|
| `MAX_MEMOS` | `10` |
| `SERVICE_NAME` | `My Memo Service` |

코드에서 읽어서 메모 개수 제한 기능 추가:

```python
MAX_MEMOS = int(os.environ.get("MAX_MEMOS", "100"))

def add_memo(text):
    if len(memos) >= MAX_MEMOS:
        return {"error": f"메모는 최대 {MAX_MEMOS}개까지 저장할 수 있습니다"}, 400
    # ... 기존 로직
```

Deploy 후 테스트.

### 실습 2. .gitignore 설정

로컬 프로젝트에 `.env` 파일과 `.gitignore` 파일 만들기:

```bash
# .gitignore에 추가
.env
__pycache__/
*.pyc
```

```bash
git add .gitignore
git commit -m "chore: .gitignore에 .env와 캐시 파일 추가"
```

---

## 자주 하는 실수

| 실수 | 위험 | 예방법 |
|------|------|--------|
| API 키를 코드에 직접 작성 | GitHub 노출 → 악용 | 환경 변수 또는 Secrets Manager 사용 |
| `.env` 파일을 git에 commit | 비밀값 공개 | `.gitignore`에 `.env` 추가 |
| `os.environ["KEY"]` 사용 | 환경 변수 없으면 KeyError | `os.environ.get("KEY", "기본값")` 사용 |
| Lambda에 과도한 권한 부여 | 보안 취약점 확대 | 최소 필요 권한만 부여 |

---

## 확인 체크리스트

- [ ] Lambda 환경 변수를 설정하고 코드에서 `os.environ.get()`으로 읽을 수 있는가
- [ ] 어떤 값을 코드에 넣으면 안 되는지 말할 수 있는가
- [ ] `.gitignore`에 `.env` 파일을 추가했는가
- [ ] Lambda의 실행 역할에 어떤 권한이 있는지 확인할 수 있는가

---

## 참고 자료

- AWS Lambda 환경 변수 — https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html
- AWS Secrets Manager — https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html
