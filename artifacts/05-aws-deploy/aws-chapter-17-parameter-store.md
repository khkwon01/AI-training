## 이 장에서 배우는 것

- AWS Systems Manager Parameter Store가 무엇인지 이해한다
- 비밀값(시크릿)을 환경변수 파일 대신 Parameter Store에 저장하는 이유를 설명할 수 있다
- AWS CLI와 Python boto3로 파라미터를 읽고 쓸 수 있다
- SecureString 타입으로 값을 암호화해서 저장할 수 있다
- 애플리케이션 코드에서 Parameter Store 값을 불러와 사용할 수 있다

---

## 먼저 쉬운 설명

코드를 짜다 보면 데이터베이스 비밀번호, API 키, 토큰 같은 민감한 값을 어딘가에 저장해야 할 순간이 옵니다.

가장 쉬운 방법은 `.env` 파일에 적는 것이지만, 이 파일을 GitHub에 올리면 전 세계 누구나 볼 수 있습니다. 실제로 이 실수로 회사 시스템이 해킹당하는 사고가 매년 수없이 일어납니다.

**AWS Parameter Store**는 이 문제를 해결하는 AWS의 공식 비밀 보관함입니다. 비밀값을 AWS 클라우드 서버에 안전하게 저장하고, 허가된 서버와 사람만 읽을 수 있게 합니다. 코드에는 비밀값 자체가 아니라 "어디서 가져오면 되는지"만 적으면 됩니다.

쉽게 비유하면 이렇습니다: 집 열쇠를 문 앞 화분 아래 두는 것(`.env` 파일)과 은행 금고에 맡기는 것(Parameter Store)의 차이입니다.

---

## 1. Parameter Store 기본 개념

Parameter Store에는 세 가지 타입의 파라미터가 있습니다.

| 타입 | 용도 | 암호화 |
|------|------|--------|
| `String` | 일반 설정값 (예: 리전, URL) | 없음 |
| `StringList` | 콤마로 구분된 목록 | 없음 |
| `SecureString` | 비밀번호, API 키 등 민감 정보 | AWS KMS로 암호화 |

파라미터 이름은 슬래시(`/`)로 계층 구조를 만들 수 있습니다.

```
/myapp/production/db-password
/myapp/production/api-key
/myapp/staging/db-password
```

이 구조 덕분에 환경별로 설정을 깔끔하게 분리할 수 있습니다.

---

## 2. AWS CLI로 파라미터 만들고 읽기

터미널에서 직접 파라미터를 만들고 읽는 연습을 합니다.

**파라미터 저장하기**

```bash
# 일반 문자열 파라미터
aws ssm put-parameter \
  --name "/myapp/production/db-host" \
  --value "mydb.ap-northeast-2.rds.amazonaws.com" \
  --type String

# 암호화된 SecureString 파라미터
aws ssm put-parameter \
  --name "/myapp/production/db-password" \
  --value "super-secret-password-123" \
  --type SecureString
```

**파라미터 읽기**

```bash
# 일반 파라미터 읽기
aws ssm get-parameter \
  --name "/myapp/production/db-host"

# SecureString 복호화해서 읽기 (--with-decryption 필수!)
aws ssm get-parameter \
  --name "/myapp/production/db-password" \
  --with-decryption
```

`--with-decryption` 옵션을 빠뜨리면 암호화된 값 그대로 나옵니다. 비밀번호가 필요할 때는 반드시 이 옵션을 붙여야 합니다.

**특정 경로 아래 파라미터 한 번에 읽기**

```bash
aws ssm get-parameters-by-path \
  --path "/myapp/production/" \
  --with-decryption
```

---

## 3. Python boto3로 파라미터 읽기

실제 애플리케이션 코드에서는 boto3 라이브러리를 사용합니다.

**기본 읽기 예제** (`config_loader.py`)

```python
import boto3

def get_parameter(name: str, with_decryption: bool = True) -> str:
    """Parameter Store에서 값을 읽어 반환한다."""
    client = boto3.client("ssm", region_name="ap-northeast-2")
    response = client.get_parameter(
        Name=name,
        WithDecryption=with_decryption
    )
    return response["Parameter"]["Value"]


# 사용 예시
db_host = get_parameter("/myapp/production/db-host")
db_password = get_parameter("/myapp/production/db-password")

print(f"DB 호스트: {db_host}")
# DB 호스트: mydb.ap-northeast-2.rds.amazonaws.com
```

**경로 아래 파라미터 전체 읽기** (`config_loader.py` 계속)

```python
def get_parameters_by_path(path: str) -> dict[str, str]:
    """특정 경로 아래 모든 파라미터를 딕셔너리로 반환한다."""
    client = boto3.client("ssm", region_name="ap-northeast-2")
    result = {}

    paginator = client.get_paginator("get_parameters_by_path")
    for page in paginator.paginate(
        Path=path,
        WithDecryption=True,
        Recursive=True
    ):
        for param in page["Parameters"]:
            # "/myapp/production/db-host" → "db-host"
            short_key = param["Name"].split("/")[-1]
            result[short_key] = param["Value"]

    return result


# 사용 예시
config = get_parameters_by_path("/myapp/production/")
print(config)
# {'db-host': 'mydb...', 'db-password': 'super-secret...'}
```

---

## 4. 실제 앱에 적용하기 — .env 파일 없애기

`.env` 파일을 사용하던 코드를 Parameter Store로 전환하는 패턴입니다.

**기존 방식 (위험)** (`app_old.py`)

```python
import os
from dotenv import load_dotenv

load_dotenv()  # .env 파일을 읽음 (GitHub에 올리면 위험!)

DB_PASSWORD = os.getenv("DB_PASSWORD")
API_KEY = os.getenv("API_KEY")
```

**Parameter Store 방식 (안전)** (`app.py`)

```python
import boto3

def load_secrets(env: str = "production") -> dict[str, str]:
    """앱 시작 시 필요한 시크릿을 한 번에 로드한다."""
    client = boto3.client("ssm", region_name="ap-northeast-2")
    paginator = client.get_paginator("get_parameters_by_path")

    secrets = {}
    for page in paginator.paginate(
        Path=f"/myapp/{env}/",
        WithDecryption=True
    ):
        for param in page["Parameters"]:
            key = param["Name"].split("/")[-1].upper().replace("-", "_")
            secrets[key] = param["Value"]

    return secrets


# 앱 시작 시 한 번만 호출
config = load_secrets(env="production")

DB_PASSWORD = config["DB_PASSWORD"]
API_KEY = config["API_KEY"]
```

이제 `.env` 파일이 없어도 되고, 코드 어디에도 비밀값이 직접 적혀 있지 않습니다.

---

## 따라 하기 실습

### 실습 1: 첫 번째 파라미터 만들기

프로젝트 폴더를 만들고 AWS CLI로 파라미터를 저장합니다.

```bash
mkdir ~/aws-ssm-practice && cd ~/aws-ssm-practice

# 1. 일반 파라미터 저장
aws ssm put-parameter \
  --name "/practice/myapp/api-url" \
  --value "https://api.example.com/v1" \
  --type String

# 2. 암호화 파라미터 저장
aws ssm put-parameter \
  --name "/practice/myapp/api-key" \
  --value "sk-abcdef1234567890" \
  --type SecureString

# 3. 저장 확인
aws ssm get-parameters-by-path \
  --path "/practice/myapp/" \
  --with-decryption \
  --query "Parameters[*].{이름:Name,값:Value}"
```

저장한 값이 터미널에 출력되면 성공입니다.

---

### 실습 2: Python으로 시크릿 읽는 스크립트 작성

실습 1에서 저장한 값을 Python으로 읽어옵니다.

```bash
pip install boto3
```

`read_secrets.py` 파일을 만들고 아래 코드를 작성합니다.

```python
# read_secrets.py
import boto3


def load_app_config() -> dict[str, str]:
    client = boto3.client("ssm", region_name="ap-northeast-2")
    paginator = client.get_paginator("get_parameters_by_path")

    config = {}
    for page in paginator.paginate(
        Path="/practice/myapp/",
        WithDecryption=True
    ):
        for param in page["Parameters"]:
            key = param["Name"].split("/")[-1]
            config[key] = param["Value"]

    return config


if __name__ == "__main__":
    app_config = load_app_config()
    for key, value in app_config.items():
        # 민감 정보는 마스킹해서 출력
        masked = value[:4] + "****" if len(value) > 4 else "****"
        print(f"{key}: {masked}")
```

```bash
python read_secrets.py
# api-key: sk-a****
# api-url: http****
```

---

### 실습 3: .env 파일과 Parameter Store를 함께 쓰는 안전한 전환 패턴

팀 환경에서 로컬 개발은 `.env`, 서버 배포는 Parameter Store를 쓰는 패턴을 구현합니다.

`config.py` 파일을 만듭니다.

```python
# config.py
import os
import boto3
from pathlib import Path


def load_config() -> dict[str, str]:
    """
    로컬 환경(APP_ENV=local)이면 .env.local 파일을 읽고,
    그 외(staging, production)이면 Parameter Store를 읽는다.
    """
    env = os.getenv("APP_ENV", "local")

    if env == "local":
        # 로컬 개발: .env.local 파일 사용 (Git에 커밋하지 않음)
        config = {}
        env_file = Path(".env.local")
        if env_file.exists():
            for line in env_file.read_text().splitlines():
                if "=" in line and not line.startswith("#"):
                    key, _, value = line.partition("=")
                    config[key.strip()] = value.strip()
        return config
    else:
        # 서버 환경: Parameter Store 사용
        client = boto3.client("ssm", region_name="ap-northeast-2")
        paginator = client.get_paginator("get_parameters_by_path")
        config = {}
        for page in paginator.paginate(
            Path=f"/myapp/{env}/",
            WithDecryption=True
        ):
            for param in page["Parameters"]:
                key = param["Name"].split("/")[-1].upper().replace("-", "_")
                config[key] = param["Value"]
        return config


# 앱 전체에서 이 딕셔너리를 임포트해서 사용
CONFIG = load_config()
```

`.env.local` 파일을 만들어 로컬 테스트용 값을 넣고 실행합니다.

```bash
# .env.local
API_KEY=local-test-key-000
API_URL=http://localhost:8000
```

```bash
APP_ENV=local python -c "from config import CONFIG; print(CONFIG)"
# {'API_KEY': 'local-test-key-000', 'API_URL': 'http://localhost:8000'}
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|------|------------|-----------|
| `--with-decryption` 빠뜨림 | 값이 `kms:...` 같은 암호화 문자열로 나옴 | CLI와 boto3 모두 `WithDecryption=True` 추가 |
| IAM 권한 없음 | `AccessDeniedException: User is not authorized to perform ssm:GetParameter` | IAM 정책에 `ssm:GetParameter`, `ssm:GetParametersByPath`, `kms:Decrypt` 권한 추가 |
| 리전 불일치 | `ParameterNotFound` | boto3 클라이언트의 `region_name`이 파라미터를 만든 리전과 같은지 확인 |
| 파라미터 덮어쓰기 실패 | `ParameterAlreadyExists` | `put-parameter`에 `--overwrite` 플래그 추가 |
| 경로 끝 슬래시 빠짐 | 빈 결과 반환 | `get_parameters_by_path`의 `Path`는 `/myapp/production/`처럼 끝에 `/` 포함 |
| `.env` 파일을 Git에 커밋 | 경고 없이 푸시됨 | `.gitignore`에 `.env`, `.env.*`, `.env.local` 추가 |
| SecureString인데 KMS 키 없음 | `InvalidKeyId` | KMS 키 ARN을 `--key-id`로 지정하거나 기본 AWS 관리형 키(`alias/aws/ssm`) 사용 |

---

## 확인 체크리스트

- [ ] `.env` 파일이 `.gitignore`에 등록되어 있다
- [ ] AWS CLI로 `put-parameter` 명령어를 실행해 파라미터를 만들어 봤다
- [ ] `SecureString` 타입 파라미터를 저장하고 `--with-decryption`으로 읽어봤다
- [ ] `get_parameters_by_path`로 경로 아래 파라미터를 한 번에 읽는 Python 코드를 작성했다
- [ ] IAM 정책에 `ssm:GetParameter`와 `kms:Decrypt` 권한이 있는지 확인했다
- [ ] 코드 어디에도 비밀번호나 API 키가 하드코딩되어 있지 않다
- [ ] 로컬 환경과 서버 환경에서 설정을 다르게 로드하는 패턴을 이해했다

---

## 한 번 더 생각해 보기

1. 팀원 A는 "Parameter Store 대신 그냥 서버 환경변수에 넣으면 되지 않나요?"라고 말합니다. 환경변수와 Parameter Store 중 어떤 방식이 더 안전하고 관리하기 쉬운지, 이유와 함께 설명해 보세요.

2. 앱이 시작될 때마다 Parameter Store를 호출하면 API 요청이 반복됩니다. 요청 횟수를 줄이려면 어떻게 설계하면 좋을까요? 모듈 수준에서 한 번만 로드하는 방식의 장단점은 무엇일까요?

3. `SecureString`이 아닌 `String` 타입으로 비밀번호를 저장했다면 어떤 위험이 생길까요? AWS CloudTrail 로그와 연결해서 생각해 보세요.

---

## 다음 장

다음 장에서는 Parameter Store와 함께 자주 쓰이는 **AWS Secrets Manager**를 살펴보고, 두 서비스의 차이점과 데이터베이스 자격증명 자동 교체(rotation) 기능을 배웁니다.