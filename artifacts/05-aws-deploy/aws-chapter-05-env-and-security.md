# Chapter 05: 환경 변수와 보안 설정

## 이 장에서 배우는 것

- 환경 변수가 왜 필요한지 (코드에 비밀번호를 넣으면 안 되는 이유)
- Lambda 콘솔에서 환경 변수를 설정하는 정확한 위치
- Python에서 `os.environ.get()` 패턴을 사용하는 이유
- Secrets Manager와 환경 변수를 언제 각각 선택하는가
- IAM 최소 권한 원칙 (왜 admin 권한을 주면 안 되는가)
- 실습: API 키를 환경 변수로 분리하기 → 코드 수정 → 동작 확인

---

## 왜 필요한가 — 코드에 비밀번호를 넣으면 안 되는 이유

처음 코드를 짤 때 이렇게 하고 싶어진다.

```python
# 내 코드 - 편하게 그냥 넣어버리자
API_KEY = "sk-abc123def456ghi789"
DB_PASSWORD = "Passw0rd!2026"

def call_weather_api(city):
    response = requests.get(
        f"https://api.weather.com/v1/current",
        headers={"Authorization": f"Bearer {API_KEY}"}
    )
    return response.json()
```

당장은 동작한다. 문제가 없어 보인다.

그런데 GitHub에 올리는 순간 무슨 일이 벌어지는지 알아야 한다.

### GitHub에서 일어나는 일

GitHub는 전 세계에 공개된 코드 저장소다. 악의적인 봇들이 24시간 쉬지 않고 새로 올라오는 코드를 스캔한다. 이 봇들은 `API_KEY =`, `password =`, `secret =`, `token =` 같은 패턴을 찾아낸다.

실제 시간:
- 코드를 GitHub에 push하면 **수 분 안에** 봇이 API 키를 수집한다
- 수집된 키로 외부 API를 호출하거나, AWS 리소스를 생성하거나, 비용을 발생시킨다

실제로 일어난 일들:
- AWS Access Key 노출 → 하루 만에 수백만 원 청구서
- OpenAI API 키 노출 → 대량 API 호출로 한도 초과
- DB 비밀번호 노출 → 데이터 무단 접근

### Private 저장소라도 위험하다

Private 저장소는 공개되지 않는다. 하지만:
- 팀원이 실수로 Public으로 바꿀 수 있다
- 저장소를 Fork하면 코드가 복사된다
- 계정이 해킹당하면 Private 저장소도 노출된다

결론: **비밀값은 코드에 절대 넣지 않는다.**

---

## 1. 환경 변수란 무엇인가

환경 변수는 코드 외부에 저장하는 설정값이다. 코드는 값 자체를 모르고, 실행 시점에 환경에서 주입받는다.

비유: 음식점 레시피에 "소금을 넣는다"고 쓰고, 실제 소금량은 매장별로 다르게 설정한다. 레시피 자체에 "소금 5g"이라고 쓰지 않는다.

### 로컬 개발 vs Lambda 배포 비교

| 상황 | 값을 어디서 읽는가 |
|------|-----------------|
| 로컬 개발 | `.env` 파일 (git에 올리지 않음) |
| Lambda 배포 | Lambda Configuration → Environment variables |

코드는 두 경우 모두 동일하다.

```python
import os

API_KEY = os.environ.get("OPENAI_API_KEY")   # 어디서 실행하든 같은 코드
```

---

## 2. Lambda 환경 변수 설정하는 곳

### 정확한 경로

```
AWS 콘솔 로그인
  → Lambda 서비스 클릭
  → 함수 목록에서 함수 이름 클릭
  → 상단 탭에서 [Configuration] 클릭
  → 왼쪽 메뉴에서 [Environment variables] 클릭
  → 오른쪽 [Edit] 버튼 클릭
  → [Add environment variable] 클릭
  → Key와 Value 입력
  → [Save] 클릭
```

### 화면 예시

Environment variables 편집 화면:

```
┌─────────────────────────────────────┐
│  Edit environment variables         │
│                                     │
│  Environment variables              │
│  ┌──────────────┬──────────────┐   │
│  │ Key          │ Value        │   │
│  ├──────────────┼──────────────┤   │
│  │ GREETING_PREFIX │ 안녕하세요 │   │
│  ├──────────────┼──────────────┤   │
│  │ MAX_MEMOS    │ 10           │   │
│  └──────────────┴──────────────┘   │
│                                     │
│  [Add environment variable]         │
│                                     │
│  [Cancel]              [Save]       │
└─────────────────────────────────────┘
```

Key 이름 규칙:
- 대문자와 언더스코어 사용 (관례)
- 예: `API_KEY`, `DB_HOST`, `MAX_RETRIES`
- 공백, 특수문자 피하기

### 주의사항

환경 변수는 저장 후 반드시 **Deploy를 다시 할 필요가 없다**. Save만 해도 즉시 적용된다.

단, 코드를 수정했다면 Deploy가 필요하다.

---

## 3. Python에서 os.environ.get() 패턴

### 기본 패턴

```python
import os

# 패턴 1: 기본값 없이 읽기 (환경변수가 없으면 None 반환)
api_key = os.environ.get("API_KEY")

# 패턴 2: 기본값과 함께 읽기 (환경변수가 없으면 기본값 사용)
prefix = os.environ.get("GREETING_PREFIX", "안녕하세요")

# 패턴 3: 숫자로 변환
max_memos = int(os.environ.get("MAX_MEMOS", "100"))

# 패턴 4: 불리언으로 변환 (환경변수는 항상 문자열)
debug_mode = os.environ.get("DEBUG_MODE", "false").lower() == "true"
```

### os.environ["KEY"] vs os.environ.get("KEY") 차이

| 방법 | 환경변수 없을 때 | 언제 사용하는가 |
|------|---------------|--------------|
| `os.environ["KEY"]` | `KeyError` 발생 → Lambda 500 오류 | 절대 없어서는 안 되는 필수값 |
| `os.environ.get("KEY")` | `None` 반환 → 코드가 계속 실행 | 대부분의 경우 |
| `os.environ.get("KEY", "기본값")` | 기본값 반환 | 기본값이 있는 설정값 |

초보자는 항상 `os.environ.get("KEY", "기본값")` 패턴을 쓰는 것이 안전하다.

### 실제 사용 예시

```python
import json
import os

def lambda_handler(event, context):
    # 환경 변수 읽기
    greeting = os.environ.get("GREETING_PREFIX", "안녕하세요")
    max_memos = int(os.environ.get("MAX_MEMOS", "100"))
    service_name = os.environ.get("SERVICE_NAME", "메모 서비스")

    params = event.get("queryStringParameters") or {}
    name = params.get("name", "World")

    return {
        "statusCode": 200,
        "body": json.dumps({
            "service": service_name,
            "message": f"{greeting}, {name}님!",
            "limit": max_memos
        }, ensure_ascii=False)
    }
```

Lambda Configuration → Environment variables에 다음을 설정:
```
GREETING_PREFIX = 반갑습니다
MAX_MEMOS = 50
SERVICE_NAME = 나의 첫 서비스
```

코드 변경 없이 인사말과 제한을 바꿀 수 있다.

---

## 4. 코드에 넣으면 안 되는 값들

| 종류 | 예시 | 위험 |
|------|------|------|
| AI/외부 API 키 | `sk-abc123...`, `Bearer xyz...` | 타인이 내 계정으로 API 호출 → 요금 폭탄 |
| 데이터베이스 비밀번호 | `myPassword123` | DB 무단 접근 → 데이터 유출 |
| AWS Access Key | `AKIAIOSFODNN7EXAMPLE` | AWS 리소스 무단 사용 → 수백만 원 청구 |
| JWT Secret | `my-jwt-secret-key` | 인증 우회 가능 → 모든 계정 탈취 |
| 개인 이메일/전화번호 | `user@email.com` | 개인정보 보호법 위반 가능 |
| 카드 번호, 주민번호 | 코드에 테스트용으로 넣는 경우 | 법적 문제 |

### .gitignore로 실수 방지

로컬 개발 시 환경 변수를 `.env` 파일에 저장한다.

```bash
# .env 파일 (절대 git에 올리지 않음)
OPENAI_API_KEY=sk-abc123def456ghi789
DB_PASSWORD=Passw0rd!2026
GREETING_PREFIX=안녕하세요
```

`.gitignore`에 반드시 추가:

```
# .gitignore
.env
*.env
.env.local
.env.production
```

확인 방법:
```bash
git status   # .env 파일이 "Untracked files"에 보이지 않아야 함
```

`.env` 파일이 `git status`에 보이지 않으면 정상적으로 무시되고 있다.

---

## 5. Secrets Manager vs 환경 변수 선택 기준

### Lambda 환경 변수가 적합한 경우

- 개발/테스트/운영 환경 구분 (`ENVIRONMENT=dev`)
- 서비스 URL, 도메인 주소
- 기능 플래그 (`FEATURE_NEW_UI=true`)
- 개수 제한, 타임아웃 같은 설정값
- 혼자 하는 개인 프로젝트의 API 키

장점: 간단하고 빠르다.
단점: Lambda 콘솔에서 값을 볼 수 있다 (관리자 권한 있는 사람이면 누구나).

### AWS Secrets Manager가 적합한 경우

- 여러 Lambda 함수가 같은 비밀값을 공유할 때
- 정기적으로 값을 교체해야 할 때 (비밀번호 rotation)
- 값에 접근한 기록(audit log)이 필요할 때
- 팀 단위 프로젝트, 실제 서비스

장점: 값 교체 이력 관리, 접근 권한 세밀하게 제어, 자동 rotation 지원.
단점: 설정이 복잡하고, 추가 비용 발생, Lambda 시작 시 API 호출이 필요.

### 초보자 판단 기준

```
혼자 하는 프로젝트 또는 학습 목적
  → Lambda 환경 변수로 충분

팀이 함께 쓰는 서비스 또는 실제 고객이 있는 서비스
  → Secrets Manager 검토
```

### Secrets Manager 사용 예시 (참고용)

**Secrets Manager에 값 저장:**
```
AWS 콘솔 → Secrets Manager → Store a new secret
→ Secret type: Other type of secret
→ Key/Value: api_key / sk-abc123def456
→ Secret name: my-service/openai-key
→ Store 클릭
```

**Lambda에서 읽기:**
```python
import boto3
import json
import os

# Lambda 시작 시 한 번만 로드 (핸들러 함수 밖)
def get_secret(secret_name):
    client = boto3.client(
        "secretsmanager",
        region_name=os.environ.get("AWS_REGION", "ap-northeast-2")
    )
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response["SecretString"])

# 콜드 스타트 시 한 번만 실행
_secrets = None

def get_cached_secrets():
    global _secrets
    if _secrets is None:
        _secrets = get_secret("my-service/openai-key")
    return _secrets

def lambda_handler(event, context):
    secrets = get_cached_secrets()
    api_key = secrets["api_key"]
    # api_key 사용
```

주의: Secrets Manager를 Lambda에서 읽으려면 실행 역할에 `SecretsManagerReadWrite` 또는 세부 권한 정책이 필요하다.

---

## 6. IAM 최소 권한 원칙

### IAM이란

AWS의 모든 서비스는 "누가 무엇을 할 수 있는가"를 IAM(Identity and Access Management)으로 관리한다.

Lambda 함수도 실행될 때 특정 IAM 역할(Role)로 동작한다. 이 역할에 어떤 권한이 있느냐에 따라 Lambda가 할 수 있는 작업이 결정된다.

### 왜 admin 권한을 주면 안 되는가

`AdministratorAccess` 정책은 AWS의 모든 서비스, 모든 리소스에 대해 모든 작업을 허용한다.

Lambda에 이 권한을 주면 어떤 일이 생길 수 있는가:

**시나리오 1: Lambda 코드 취약점 악용**

Lambda 코드에 버그가 있어서 공격자가 임의 코드를 실행할 수 있게 됐다. Lambda에 admin 권한이 있다면:
- 공격자가 새 EC2 인스턴스를 대량 생성 → 수천만 원 청구
- 다른 Lambda 함수의 환경 변수(비밀값) 열람
- S3 버킷의 모든 파일 삭제
- IAM 사용자 생성으로 지속적인 접근 확보

Lambda에 최소 권한만 있었다면 피해 범위가 극도로 제한됐을 것이다.

**시나리오 2: 실수로 잘못된 코드 배포**

코드 실수로 DynamoDB 테이블을 삭제하는 코드가 배포됐다. admin 권한이 있으면 실행된다. 최소 권한(해당 테이블 읽기만 허용)이었다면 실행조차 안 된다.

### 최소 권한 원칙

Lambda가 실제로 필요한 권한만 부여한다.

| 작업 | 부여해야 하는 권한 |
|------|-----------------|
| CloudWatch 로그 쓰기 | `AWSLambdaBasicExecutionRole` (Lambda 생성 시 기본 포함) |
| 특정 DynamoDB 테이블 읽기/쓰기 | 해당 테이블에 대한 `dynamodb:GetItem`, `dynamodb:PutItem` 등 |
| Secrets Manager 특정 비밀 읽기 | 해당 비밀에 대한 `secretsmanager:GetSecretValue` |
| S3 특정 버킷 읽기 | 해당 버킷에 대한 `s3:GetObject` |

### Lambda 실행 역할 확인 방법

```
Lambda 함수 페이지
  → [Configuration] 탭
  → 왼쪽 메뉴: [Permissions]
  → [Execution role] 섹션
  → 역할 이름 링크 클릭 → IAM 콘솔로 이동
  → [Permissions policies] 탭에서 부여된 정책 목록 확인
```

### 기본 실행 역할에 포함된 권한

Lambda를 처음 만들 때 자동으로 생성되는 역할에는 `AWSLambdaBasicExecutionRole`이 포함된다. 이 정책은 CloudWatch Logs에 쓰는 권한만 포함한다.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        }
    ]
}
```

DynamoDB, S3, Secrets Manager 같은 다른 서비스에 접근하려면 추가 권한이 필요하다.

---

## 7. 따라 하기 실습

### 실습 1. 인사말을 환경 변수로 분리하기

**목표**: 코드에 고정된 값을 환경 변수로 분리하고, 코드 변경 없이 값을 바꾸는 경험을 한다.

**Step 1. 현재 코드 확인**

현재 함수가 다음과 같이 인사말이 코드에 하드코딩되어 있다고 가정한다.

```python
import json

def lambda_handler(event, context):
    params = event.get("queryStringParameters") or {}
    name = params.get("name", "World")

    # 인사말이 코드에 직접 박혀 있음
    message = f"안녕하세요, {name}님! 오늘도 좋은 하루 되세요."

    return {
        "statusCode": 200,
        "body": json.dumps({"message": message}, ensure_ascii=False)
    }
```

**Step 2. 코드를 환경 변수를 읽도록 수정**

```python
import json
import os

def lambda_handler(event, context):
    params = event.get("queryStringParameters") or {}
    name = params.get("name", "World")

    # 환경 변수에서 읽기 (없으면 기본값 사용)
    greeting = os.environ.get("GREETING_MESSAGE", "안녕하세요")
    suffix = os.environ.get("GREETING_SUFFIX", "오늘도 좋은 하루 되세요.")

    message = f"{greeting}, {name}님! {suffix}"

    return {
        "statusCode": 200,
        "body": json.dumps({"message": message}, ensure_ascii=False)
    }
```

**Step 3. Lambda 환경 변수 설정**

```
Lambda 함수 → Configuration → Environment variables → Edit → Add environment variable
```

다음 두 개 추가:
```
Key: GREETING_MESSAGE    Value: 반갑습니다
Key: GREETING_SUFFIX     Value: 즐거운 하루 보내세요!
```

Save 클릭.

**Step 4. Deploy 후 테스트**

```bash
curl "https://함수URL/?name=철수"
```

응답:
```json
{"message": "반갑습니다, 철수님! 즐거운 하루 보내세요!"}
```

**Step 5. 코드 변경 없이 값 바꾸기**

Configuration → Environment variables → Edit에서:
```
GREETING_MESSAGE = Hello
GREETING_SUFFIX  = Have a great day!
```

Save 후 다시 호출하면 코드 변경 없이 다른 메시지가 나온다.

---

### 실습 2. API 키를 환경 변수로 분리하기

**목표**: 코드에 직접 쓴 API 키를 환경 변수로 옮기고, 실제 동작을 확인한다.

**Step 1. 잘못된 코드 (절대 이렇게 하면 안 됨, 예시용)**

```python
import json
import requests  # requests 패키지가 있다고 가정

# 나쁜 예: API 키가 코드에 노출
WEATHER_API_KEY = "abc123def456"

def lambda_handler(event, context):
    city = (event.get("queryStringParameters") or {}).get("city", "Seoul")

    # 실제 외부 API 대신 시뮬레이션
    print(f"[INFO] API 키: {WEATHER_API_KEY}")  # 이것도 절대 하면 안 됨
    print(f"[INFO] 요청 도시: {city}")

    return {
        "statusCode": 200,
        "body": json.dumps({"city": city, "weather": "맑음"}, ensure_ascii=False)
    }
```

**Step 2. 환경 변수로 분리한 올바른 코드**

```python
import json
import os

def lambda_handler(event, context):
    city = (event.get("queryStringParameters") or {}).get("city", "Seoul")

    # 환경 변수에서 API 키 읽기
    api_key = os.environ.get("WEATHER_API_KEY")

    if not api_key:
        print("[ERROR] WEATHER_API_KEY 환경 변수가 설정되지 않음")
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "설정 오류"})
        }

    # API 키를 로그에 출력하면 절대 안 됨
    print(f"[INFO] 요청 도시: {city}")
    print("[INFO] API 키 로드 완료 (값은 출력하지 않음)")

    # 실제 API 호출 (여기서는 시뮬레이션)
    return {
        "statusCode": 200,
        "body": json.dumps({"city": city, "weather": "맑음"}, ensure_ascii=False)
    }
```

**Step 3. 환경 변수 설정**

```
Configuration → Environment variables → Edit → Add environment variable
Key: WEATHER_API_KEY
Value: (실제 API 키 또는 테스트용 가짜 값 입력)
```

**Step 4. 배포 및 테스트**

```bash
# API 키가 있는 경우
curl "https://함수URL/?city=Seoul"

# 환경변수 없이 테스트 (오류 확인용 - 환경변수 삭제 후 테스트)
# → 500 오류와 "[ERROR] WEATHER_API_KEY 환경 변수가 설정되지 않음" 로그 확인
```

---

### 실습 3. .gitignore 설정 확인하기

**목표**: 로컬 프로젝트에서 `.env` 파일이 git에 올라가지 않도록 설정한다.

**Step 1. 로컬 프로젝트 폴더에서 .env 파일 생성**

```bash
# 프로젝트 폴더로 이동
cd ~/내프로젝트폴더

# .env 파일 생성 (값은 실제 키로)
# (텍스트 편집기로 직접 생성)
```

`.env` 파일 내용:
```
WEATHER_API_KEY=abc123def456
GREETING_MESSAGE=안녕하세요
MAX_MEMOS=50
```

**Step 2. .gitignore에 추가**

프로젝트 루트에 `.gitignore` 파일을 편집기로 열어서 다음 추가:

```
# 환경 변수 파일 (절대 git에 올리지 않음)
.env
*.env
.env.local
.env.development
.env.production

# Python 캐시
__pycache__/
*.pyc
*.pyo

# macOS
.DS_Store
```

**Step 3. git이 .env를 무시하는지 확인**

```bash
git status
```

`.env` 파일이 "Untracked files"에 나오면 안 된다. 보이지 않으면 정상이다.

만약 보인다면:
```bash
# .gitignore가 제대로 설정됐는지 확인
cat .gitignore   # .env가 있는지 확인
```

**Step 4. .gitignore 커밋**

```bash
git add .gitignore
git commit -m "chore: .gitignore에 .env와 Python 캐시 파일 추가"
git push
```

---

## 자주 막히는 지점

### 막히는 지점 1. "Configuration 탭이 어디 있는지 모르겠어요"

Lambda 함수 페이지에서 코드 편집기 **위쪽**에 탭들이 있다:
```
[Code] [Test] [Monitor] [Configuration] [Aliases] [Versions]
```
여기서 [Configuration]을 클릭한다.

### 막히는 지점 2. "환경 변수를 설정했는데 코드에서 None이 나와요"

원인 1: Save를 클릭하지 않았다 → Save 버튼 다시 확인.

원인 2: Key 이름을 잘못 입력했다 → 콘솔의 Key 이름과 코드의 `os.environ.get("KEY")` 이름이 정확히 일치하는지 확인 (대소문자 구분).

원인 3: `os.environ["KEY"]` 사용 시 KeyError → `os.environ.get("KEY", "기본값")`으로 변경.

### 막히는 지점 3. "os 모듈을 import 안 했어요"

```python
import os   # 파일 맨 위에 이 줄이 있어야 함
```

`import os`가 없으면 `NameError: name 'os' is not defined` 오류 발생.

### 막히는 지점 4. "숫자를 환경 변수로 저장했는데 비교가 안 돼요"

환경 변수는 항상 **문자열**로 저장된다.

```python
# 잘못된 사용
max_memos = os.environ.get("MAX_MEMOS", "100")
if len(memos) >= max_memos:    # 문자열 "100"과 숫자를 비교 → TypeError

# 올바른 사용
max_memos = int(os.environ.get("MAX_MEMOS", "100"))
if len(memos) >= max_memos:    # 정수끼리 비교 → 정상
```

---

## 자주 하는 실수

| 실수 | 위험 | 예방법 |
|------|------|--------|
| API 키를 코드에 직접 작성 | GitHub 노출 → 즉시 악용 | 환경 변수 또는 Secrets Manager 사용 |
| `.env` 파일을 git에 commit | 비밀값 공개 | `.gitignore`에 `.env` 추가 |
| `os.environ["KEY"]` 사용 | 환경 변수 없으면 KeyError → 500 오류 | `os.environ.get("KEY", "기본값")` 사용 |
| Lambda에 AdministratorAccess 부여 | 취약점 악용 시 전체 AWS 계정 탈취 | 최소 필요 권한만 부여 |
| 환경 변수값을 로그에 출력 | 비밀값이 CloudWatch에 노출 | 민감한 값은 절대 print()로 출력하지 않음 |
| 숫자를 int()로 변환하지 않음 | 비교 오류, 계산 오류 | `int(os.environ.get("N", "기본값"))` 사용 |

---

## 확인 체크리스트

- [ ] Lambda Configuration → Environment variables 경로를 직접 찾아갈 수 있는가
- [ ] `os.environ.get("KEY", "기본값")` 패턴을 사용하고 있는가
- [ ] 숫자형 환경 변수를 `int()`, 불리언을 `.lower() == "true"`로 변환하는가
- [ ] `.gitignore`에 `.env`가 포함되어 있는가
- [ ] Lambda 실행 역할의 권한 목록을 확인하는 방법을 아는가
- [ ] 코드, git 저장소에 API 키나 비밀번호가 없는가

---

## 한 번 더 생각해 보기

1. 같은 API 키를 10개의 Lambda 함수에서 사용한다면 Lambda 환경 변수와 Secrets Manager 중 어떤 게 더 나을까?
2. 개발 환경과 운영 환경에서 다른 값을 쓰고 싶을 때 환경 변수를 어떻게 활용하면 될까?
3. Lambda에 S3 전체 접근 권한이 있는데, 실제로는 특정 버킷 하나만 읽으면 된다면 어떻게 바꿔야 할까?

---

## 다음 장

다음 장에서는 지금까지 배운 내용(배포, 로깅, 환경 변수)을 종합해서 실제 배포할 때 따르는 런북과 운영 체크리스트를 배운다.

---

## 참고 자료

- AWS Lambda 환경 변수 공식 문서 — https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html
- AWS Secrets Manager 공식 문서 — https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html
- IAM 최소 권한 원칙 — https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege
- Python os.environ 문서 — https://docs.python.org/3/library/os.html#os.environ
