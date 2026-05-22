# Chapter 19: SAM CLI로 Lambda 배포하기

---

## 학습 목표

1. SAM CLI가 왜 필요한지 (콘솔 클릭 배포의 한계) 이해한다
2. SAM CLI를 설치하고 `sam init`으로 프로젝트를 만들 수 있다
3. `template.yaml`의 기본 구조를 읽고 수정할 수 있다
4. `sam build` → `sam deploy --guided`로 Lambda를 배포할 수 있다
5. 배포한 함수를 curl로 테스트하고, 수정 후 재배포할 수 있다

---

## 1. 왜 SAM CLI인가 — 콘솔 클릭 배포의 한계

지금까지 AWS 콘솔(웹 브라우저)에서 Lambda 함수를 만들었습니다. 함수 코드를 붙여넣고, 저장 버튼을 누르면 바로 동작했죠. 처음에는 충분합니다.

그런데 실제 프로젝트에서는 문제가 생깁니다.

### 콘솔 클릭 배포의 3가지 문제

**문제 1: 재현이 불가능하다**

어제 잘 동작하던 Lambda가 오늘 이상하게 동작합니다. "내가 어떤 설정을 바꿨더라…" 기억이 나지 않습니다. 콘솔에서 한 작업은 기록이 남지 않기 때문입니다. 처음부터 다시 만들어야 할 수도 있습니다.

**문제 2: 팀원과 공유할 수 없다**

새 팀원이 합류했습니다. "Lambda 환경 셋업해줘"라고 하면 어떻게 전달하나요? "콘솔 들어가서 여기 클릭하고, 저기 클릭하고…" 이렇게 구두로 설명해야 합니다. 화면 캡처 20장짜리 문서를 만들거나, 직접 옆에 앉아서 가르쳐줘야 합니다.

**문제 3: 실수할 위험이 높다**

프로덕션(실제 서비스) 환경과 개발 환경을 각각 유지해야 합니다. 콘솔에서 클릭하다 보면 "어, 지금 프로덕션에 적용한 건가, 개발에 적용한 건가?" 헷갈립니다. 한 번의 실수가 서비스 장애로 이어집니다.

### SAM CLI는 이 문제를 어떻게 해결하나

SAM CLI는 Lambda 배포를 **코드(파일)**로 정의합니다. `template.yaml`이라는 파일 하나에 "이 함수는 이런 설정으로 만든다"고 적어두면, 언제든 같은 결과를 재현할 수 있습니다.

- 파일이 있으니 Git으로 버전 관리가 됩니다
- GitHub에 올리면 팀원 누구나 받아서 똑같은 환경을 만들 수 있습니다
- 어떤 설정을 바꿨는지 `git diff`로 한눈에 확인할 수 있습니다

---

## 2. 콘솔 배포 vs SAM CLI 비교표

| 항목 | 콘솔 클릭 배포 | SAM CLI 배포 |
|------|--------------|-------------|
| **재현성** | 낮음 (설정 기억에 의존) | 높음 (template.yaml로 동일 재현) |
| **버전 관리** | 불가능 | Git으로 전체 이력 관리 |
| **팀 공유** | 화면 캡처 또는 구두 설명 | 파일 하나로 공유 (`git clone` 후 `sam deploy`) |
| **자동화** | 불가능 | CI/CD 파이프라인에 연결 가능 |
| **실수 위험** | 높음 (어느 환경인지 헷갈림) | 낮음 (환경별 파라미터 파일로 분리) |
| **초기 진입 장벽** | 낮음 (클릭만 하면 됨) | 중간 (CLI 설치 필요) |
| **여러 리소스 관리** | 어려움 (따로따로 클릭) | 쉬움 (하나의 파일에 모두 정의) |

처음에는 콘솔이 편합니다. 하지만 함수가 2개, 3개가 되고, 팀이 생기는 순간 SAM CLI가 훨씬 효율적입니다.

---

## 3. SAM CLI 설치 방법

### macOS (Homebrew 사용)

터미널을 열고 다음 명령어를 실행합니다.

```bash
brew install aws-sam-cli
```

Homebrew가 없다면 먼저 설치합니다.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

설치 후 버전을 확인합니다.

```bash
sam --version
# SAM CLI, version 1.x.x
```

### Windows (MSI 설치)

1. 브라우저에서 검색: `AWS SAM CLI install Windows`
2. 공식 AWS 문서 페이지에서 MSI 설치 파일 다운로드
3. 다운로드된 `.msi` 파일을 더블 클릭
4. 설치 마법사의 "Next" 버튼을 따라 진행
5. 설치 완료 후 Command Prompt(명령 프롬프트) 또는 PowerShell을 **새로 열기**
6. 버전 확인

```cmd
sam --version
```

> **주의**: 설치 전에 열려 있던 터미널은 새 명령어를 인식하지 못합니다. 반드시 터미널을 새로 열어야 합니다.

### Linux

```bash
# 64비트 Linux
pip3 install aws-sam-cli

# 또는 공식 패키지 설치
curl -Lo sam-installation.sh https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
```

---

## 4. AWS 자격증명 설정

SAM CLI가 AWS에 배포하려면 "내가 누구인지" 알려줘야 합니다. AWS에서 발급한 키가 필요합니다.

### AWS Access Key 발급 (처음인 경우)

1. AWS 콘솔 로그인
2. 오른쪽 상단 계정 이름 클릭 → **Security credentials**
3. **Access keys** 섹션 → **Create access key**
4. **Access key ID**와 **Secret access key** 복사 (Secret은 이 화면에서만 볼 수 있습니다)

### aws configure 명령어

터미널에서 실행합니다.

```bash
aws configure
```

4가지 질문이 차례로 나옵니다.

```
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

- **Access Key ID**: 발급받은 키 ID 입력
- **Secret Access Key**: 발급받은 시크릿 키 입력
- **Default region name**: `ap-northeast-2` (서울 리전)
- **Default output format**: `json` 입력

설정이 저장됐는지 확인합니다.

```bash
aws sts get-caller-identity
```

다음과 비슷한 출력이 나오면 성공입니다.

```json
{
    "UserId": "AIDAIOSFODNN7EXAMPLE",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/myname"
}
```

---

## 5. sam init — 새 프로젝트 만들기

작업할 폴더로 이동한 뒤 `sam init`을 실행합니다.

```bash
cd ~/projects
sam init
```

여러 질문이 나옵니다. 처음 사용자는 아래와 같이 선택합니다.

### 단계별 선택 안내

**1단계: 템플릿 소스 선택**

```
Which template source would you like to use?
        1 - AWS Quick Start Templates
        2 - Custom Template Location
Choice: 1
```

→ `1` 입력 (AWS 공식 템플릿 사용)

**2단계: 템플릿 선택**

```
Choose an AWS Quick Start application template
        1 - Hello World Example
        2 - Data processing
        ...
Template: 1
```

→ `1` 입력 (Hello World 예제)

**3단계: Python 버전 사용 여부**

```
Use the most popular runtime and package type? (Python and zip) [y/N]: y
```

→ `y` 입력

**4단계: X-Ray 트레이싱**

```
Would you like to enable X-Ray tracing on the function(s) in your application? [y/N]: N
```

→ `N` 입력 (일단 넘어갑니다)

**5단계: CloudWatch Application Insights**

```
Would you like to enable monitoring using CloudWatch Application Insights? [y/N]: N
```

→ `N` 입력

**6단계: 프로젝트 이름**

```
Project name [sam-app]: my-lambda-app
```

→ `my-lambda-app` 입력 (원하는 이름으로 변경 가능)

완료되면 다음 메시지가 나옵니다.

```
    -----------------------
    Generating application:
    -----------------------
    Name: my-lambda-app
    ...
    ✓ Application created successfully

    Next steps can be found in the README file at my-lambda-app/README.md
```

---

## 6. 생성된 파일 구조

`my-lambda-app` 폴더가 만들어집니다.

```
my-lambda-app/
├── template.yaml          ← Lambda 설정 파일 (핵심)
├── hello_world/
│   ├── app.py             ← Lambda 함수 코드
│   ├── requirements.txt   ← Python 패키지 목록
│   └── __init__.py
├── tests/
│   ├── unit/
│   │   └── test_handler.py
│   └── integration/
│       └── test_api_gateway.py
├── samconfig.toml         ← sam deploy 설정 저장 파일
├── events/
│   └── event.json         ← 로컬 테스트용 이벤트 샘플
└── README.md
```

### 중요 파일 설명

| 파일 | 역할 |
|------|------|
| `template.yaml` | 어떤 Lambda 함수를, 어떤 설정으로 만들지 정의 |
| `hello_world/app.py` | 실제 Lambda 함수 코드 |
| `hello_world/requirements.txt` | 함수가 사용할 Python 라이브러리 목록 |
| `samconfig.toml` | `sam deploy --guided` 후 설정이 저장되는 파일 |
| `events/event.json` | 로컬에서 테스트할 때 사용하는 가짜 이벤트 데이터 |

---

## 7. template.yaml 주요 섹션 상세 설명

`template.yaml` 파일을 에디터로 열어보면 크게 4개 섹션으로 나뉩니다.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  my-lambda-app
  Sample SAM Template for my-lambda-app

Globals:
  Function:
    Timeout: 3

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.12
      Architectures:
        - x86_64
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL for Prod stage for Hello World function"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  HelloWorldFunction:
    Description: "Hello World Lambda Function ARN"
    Value: !GetAtt HelloWorldFunction.Arn
```

### Globals 섹션

```yaml
Globals:
  Function:
    Timeout: 3
```

모든 Lambda 함수에 공통으로 적용할 설정입니다. `Timeout: 3`은 함수가 3초 안에 실행되지 않으면 강제 종료한다는 의미입니다. 여기서 설정하면 Resources의 각 함수마다 반복할 필요가 없습니다.

### Resources 섹션

가장 핵심적인 섹션입니다. 만들 AWS 리소스를 정의합니다.

```yaml
Resources:
  HelloWorldFunction:           # 리소스 이름 (자유롭게 지정)
    Type: AWS::Serverless::Function   # Lambda 함수 타입
    Properties:
      CodeUri: hello_world/     # 코드가 있는 폴더
      Handler: app.lambda_handler     # 파일명.함수명
      Runtime: python3.12       # Python 버전
      Events:
        HelloWorld:
          Type: Api             # API Gateway 연결
          Properties:
            Path: /hello        # URL 경로
            Method: get         # HTTP 메서드
```

- `CodeUri`: Lambda 코드가 있는 로컬 폴더 경로
- `Handler`: `파일명.함수명` 형식. `app.lambda_handler`는 `app.py` 파일의 `lambda_handler` 함수를 실행하라는 의미
- `Runtime`: `python3.12`, `nodejs20.x` 등 실행 환경
- `Events`: 이 함수를 어떻게 트리거할지 정의. `Type: Api`는 HTTP 요청으로 호출한다는 의미

### Outputs 섹션

배포 후 확인이 필요한 값을 출력합니다.

```yaml
Outputs:
  HelloWorldApi:
    Description: "API Gateway endpoint URL"
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
```

`sam deploy` 완료 후 터미널에 이 URL이 출력됩니다. 이 URL로 curl 테스트를 할 수 있습니다.

---

## 8. Python Lambda 핸들러 코드 예시

`hello_world/app.py` 파일을 열면 기본 코드가 있습니다.

```python
import json


def lambda_handler(event, context):
    """Lambda 함수의 진입점(entry point).

    Parameters
    ----------
    event : dict
        API Gateway에서 전달된 요청 정보.
        - event['httpMethod']: GET, POST 등
        - event['queryStringParameters']: ?name=value 형태의 파라미터
        - event['body']: POST body
    context : object
        Lambda 실행 환경 정보 (메모리, 타임아웃 등).

    Returns
    -------
    dict
        API Gateway가 클라이언트에 돌려줄 HTTP 응답.
    """
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world",
        }),
    }
```

### 응답 구조 이해

Lambda가 API Gateway와 연결되면, 반드시 이 형식으로 응답해야 합니다.

```python
{
    "statusCode": 200,        # HTTP 상태 코드 (200 = 성공)
    "headers": {              # 선택사항: 응답 헤더
        "Content-Type": "application/json"
    },
    "body": json.dumps({...}) # 응답 본문 (반드시 문자열)
}
```

`body`는 **문자열**이어야 합니다. dict를 그대로 반환하면 오류가 납니다. `json.dumps()`로 변환해야 합니다.

### 커스텀 응답 예시

```python
import json


def lambda_handler(event, context):
    name = "World"

    # 쿼리 파라미터 읽기 (?name=Alice)
    query_params = event.get("queryStringParameters") or {}
    if "name" in query_params:
        name = query_params["name"]

    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json"
        },
        "body": json.dumps({
            "message": f"Hello, {name}!",
            "timestamp": "2026-01-01T00:00:00Z"
        }),
    }
```

---

## 9. sam build — 빌드하기

프로젝트 폴더로 이동해서 빌드합니다.

```bash
cd my-lambda-app
sam build
```

### 빌드 중에 무슨 일이 일어나나

1. `requirements.txt`에 적힌 Python 패키지를 설치합니다
2. 코드와 패키지를 `.aws-sam/build/` 폴더에 모읍니다
3. AWS에 올릴 수 있는 형태로 패키징합니다

```
Building codeuri: /Users/.../my-lambda-app/hello_world runtime: python3.12 ...
 - Installing dependencies from: requirements.txt
Build Succeeded

Built Artifacts  : .aws-sam/build
Built Template   : .aws-sam/build/template.yaml

Commands you can use next
=========================
[*] Validate SAM template: sam validate
[*] Invoke Function: sam local invoke
[*] Test Function in the Cloud: sam sync --stack-name {{stack-name}} --watch
[*] Deploy: sam deploy --guided
```

`Build Succeeded` 메시지가 나오면 성공입니다.

### 빌드가 필요한 경우

- `app.py` 코드를 수정했을 때
- `requirements.txt`에 새 패키지를 추가했을 때
- **코드를 바꿀 때마다** `sam build`를 먼저 실행해야 합니다

---

## 10. sam deploy --guided — 배포하기

빌드 후 배포합니다. `--guided` 옵션은 처음 배포할 때 필요한 설정을 물어봅니다.

```bash
sam deploy --guided
```

### 각 질문에 어떻게 답하는지

```
Configuring SAM deploy
======================

        Looking for config file [samconfig.toml] :  Not found

        Setting default arguments for 'sam deploy'
        =========================================
        Stack Name [sam-app]: my-lambda-app
        AWS Region [us-east-1]: ap-northeast-2
        #Shows you resources changes to be deployed and require a 'Y' to initiate deploy
        Confirm changes before deploy [y/N]: y
        #SAM needs permission to be able to create roles to connect to the resources in your template
        Allow SAM CLI IAM role creation [Y/n]: Y
        #Preserves the state of previously provisioned resources when an operation fails
        Disable rollback [y/N]: N
        HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
        Save arguments to configuration file [Y/n]: Y
        SAM configuration file [samconfig.toml]: (Enter 그냥 누름)
        SAM configuration environment [default]: (Enter 그냥 누름)
```

| 질문 | 권장 답변 | 이유 |
|------|----------|------|
| Stack Name | `my-lambda-app` | CloudFormation 스택 이름. 원하는 이름 입력 |
| AWS Region | `ap-northeast-2` | 서울 리전 |
| Confirm changes before deploy | `y` | 배포 전 변경사항 확인 (안전) |
| Allow SAM CLI IAM role creation | `Y` | Lambda 실행 권한 자동 생성 허용 |
| Disable rollback | `N` | 오류 시 이전 상태로 자동 복구 |
| HelloWorldFunction authorization | `y` | 인증 없이 공개 접근 허용 (테스트용) |
| Save arguments | `Y` | 다음에 `sam deploy`만 해도 동일 설정 적용 |

### 배포 진행 화면

설정 입력 후 배포가 시작됩니다.

```
Deploying with following values
================================
Stack name                   : my-lambda-app
Region                       : ap-northeast-2
Confirm changeset            : True
...

Initiating deployment
=====================

        Uploading to my-lambda-app/...  12345 / 12345  (100.00%)

Waiting for changeset to be created..

CloudFormation stack changeset
-------------------------------------------------------------------------------------------------
Operation  LogicalResourceId         ResourceType                    Replacement
-------------------------------------------------------------------------------------------------
+ Add      HelloWorldFunctionRole    AWS::IAM::Role                  N/A
+ Add      HelloWorldFunction        AWS::Lambda::Function           N/A
+ Add      ServerlessRestApi         AWS::ApiGateway::RestApi        N/A
...
-------------------------------------------------------------------------------------------------

Changeset created successfully. arn:aws:cloudformation:...

Previewing CloudFormation changeset before deployment
======================================================
Deploy this changeset? [y/N]: y
```

`y`를 입력하면 실제 배포가 시작됩니다.

```
2026-01-01 00:00:00 - Waiting for stack create/update to complete

CloudFormation events from stack operations (refresh every 5.0 seconds)
-------------------------------------------------------------------------------------------------
ResourceStatus       ResourceType                    LogicalResourceId
-------------------------------------------------------------------------------------------------
CREATE_IN_PROGRESS   AWS::IAM::Role                  HelloWorldFunctionRole
CREATE_COMPLETE      AWS::IAM::Role                  HelloWorldFunctionRole
CREATE_IN_PROGRESS   AWS::Lambda::Function           HelloWorldFunction
CREATE_COMPLETE      AWS::Lambda::Function           HelloWorldFunction
...
CREATE_COMPLETE      AWS::CloudFormation::Stack      my-lambda-app
-------------------------------------------------------------------------------------------------

CloudFormation outputs from deployed stack
-------------------------------------------------------------------------------------------------
Outputs
-------------------------------------------------------------------------------------------------
Key                 HelloWorldApi
Description         API Gateway endpoint URL for Prod stage
Value               https://abcd1234.execute-api.ap-northeast-2.amazonaws.com/Prod/hello/
-------------------------------------------------------------------------------------------------

Successfully created/updated stack - my-lambda-app in ap-northeast-2
```

`Successfully created/updated stack` 메시지가 보이면 배포 성공입니다.

---

## 11. 배포 후 URL 확인 및 curl 테스트

Outputs 섹션에 나온 URL을 복사해서 테스트합니다.

### curl로 테스트

```bash
curl https://abcd1234.execute-api.ap-northeast-2.amazonaws.com/Prod/hello/
```

성공하면 다음과 같은 응답이 옵니다.

```json
{"message": "hello world"}
```

### 브라우저로 테스트

URL을 브라우저 주소창에 붙여넣어도 됩니다. JSON 형식의 응답이 보이면 성공입니다.

### 응답 코드 확인

```bash
curl -v https://abcd1234.execute-api.ap-northeast-2.amazonaws.com/Prod/hello/
```

`-v` 옵션을 추가하면 HTTP 상태 코드(200 OK 등)도 확인할 수 있습니다.

---

## 12. 코드 수정 후 재배포

코드를 변경하면 **build → deploy** 순서로 다시 실행합니다.

### 예시: 응답 메시지 변경

`hello_world/app.py`를 수정합니다.

```python
import json


def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "안녕하세요, Lambda!",
            "version": "2.0"
        }),
    }
```

저장 후 재배포합니다.

```bash
# 1단계: 빌드
sam build

# 2단계: 배포 (--guided 없이 실행 — samconfig.toml에 설정 저장됨)
sam deploy
```

두 번째 배포부터는 `--guided`가 필요 없습니다. `samconfig.toml`에 저장된 설정을 자동으로 사용합니다.

```
Uploading to my-lambda-app/... 12345 / 12345 (100.00%)

CloudFormation stack changeset
-------------------------------------------------------------------------------------------------
Operation  LogicalResourceId   ResourceType              Replacement
-------------------------------------------------------------------------------------------------
~ Update   HelloWorldFunction  AWS::Lambda::Function     False
-------------------------------------------------------------------------------------------------

Deploy this changeset? [y/N]: y

Successfully created/updated stack - my-lambda-app in ap-northeast-2
```

`Update`로 표시되며 기존 함수가 업데이트됩니다.

재배포 후 curl로 변경된 응답을 확인합니다.

```bash
curl https://abcd1234.execute-api.ap-northeast-2.amazonaws.com/Prod/hello/
# {"message": "안녕하세요, Lambda!", "version": "2.0"}
```

---

## 13. 정리 (sam delete)

실습이 끝났거나 리소스를 삭제하고 싶을 때 사용합니다. AWS는 사용한 만큼 비용이 청구됩니다. 실습 후에는 삭제합니다.

```bash
sam delete
```

확인 질문이 나옵니다.

```
        Are you sure you want to delete the stack my-lambda-app in the region ap-northeast-2 ? [y/N]: y
        Are you sure you want to delete the folder my-lambda-app in S3 which contains the artifacts? [y/N]: y
```

둘 다 `y`를 입력하면 Lambda 함수, API Gateway, S3에 올린 코드 등 관련 리소스가 모두 삭제됩니다.

```
        - Deleting S3 object with key my-lambda-app/...
        - Deleting Cloudformation stack my-lambda-app

Deleted successfully
```

> **주의**: `sam delete`는 되돌릴 수 없습니다. 프로덕션 환경에서 실행할 때는 반드시 스택 이름을 확인하세요.

---

## 14. 실습

### 실습 1: Hello World 배포

**목표**: SAM CLI를 이용해 Hello World Lambda를 배포하고, curl로 응답을 확인한다.

**순서**:

```bash
# 1. 새 프로젝트 생성
sam init
# → AWS Quick Start Templates → Hello World Example → Python → y

# 2. 프로젝트 폴더로 이동
cd my-lambda-app

# 3. 빌드
sam build

# 4. 배포
sam deploy --guided
# → 각 질문에 위의 가이드대로 답변

# 5. 출력된 URL로 테스트
curl https://[출력된URL]/Prod/hello/
```

**완료 기준**: curl 요청 시 `{"message": "hello world"}` 응답이 오면 성공

---

### 실습 2: 커스텀 응답 반환

**목표**: `app.py`를 수정해서 자신의 이름을 포함한 응답을 반환한다.

`hello_world/app.py`를 다음과 같이 수정합니다.

```python
import json
import datetime


def lambda_handler(event, context):
    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json"
        },
        "body": json.dumps({
            "message": "안녕하세요! 저는 Lambda입니다.",
            "author": "여기에_자신의_이름",
            "deployed_at": datetime.datetime.utcnow().isoformat()
        }, ensure_ascii=False),
    }
```

재배포하고 결과를 확인합니다.

```bash
sam build && sam deploy
curl https://[URL]/Prod/hello/
```

**완료 기준**: 응답에 자신의 이름과 현재 시각이 포함되어 있으면 성공

---

### 실습 3: 쿼리 파라미터 읽기

**목표**: URL에 `?name=이름` 을 붙이면 개인화된 인사말을 반환하는 함수를 만든다.

`hello_world/app.py`를 수정합니다.

```python
import json


def lambda_handler(event, context):
    # 쿼리 파라미터 읽기
    query_params = event.get("queryStringParameters") or {}
    name = query_params.get("name", "World")

    # 언어 파라미터 읽기
    lang = query_params.get("lang", "en")

    if lang == "ko":
        message = f"안녕하세요, {name}님!"
    elif lang == "ja":
        message = f"こんにちは、{name}さん！"
    else:
        message = f"Hello, {name}!"

    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json; charset=utf-8"
        },
        "body": json.dumps({
            "message": message,
            "name": name,
            "lang": lang
        }, ensure_ascii=False),
    }
```

재배포 후 다양한 파라미터로 테스트합니다.

```bash
sam build && sam deploy

# 기본 (영어)
curl "https://[URL]/Prod/hello/"

# 이름 지정
curl "https://[URL]/Prod/hello/?name=Alice"

# 한국어
curl "https://[URL]/Prod/hello/?name=철수&lang=ko"

# 일본어
curl "https://[URL]/Prod/hello/?name=田中&lang=ja"
```

**완료 기준**: 파라미터에 따라 다른 언어의 인사말이 반환되면 성공

---

## 자주 하는 실수

### 실수 1: build 없이 deploy

코드를 수정했는데 `sam build` 없이 `sam deploy`하면 이전 코드가 배포됩니다.

```bash
# 잘못된 순서
sam deploy  # 코드 수정 후 build 안 함

# 올바른 순서
sam build   # 반드시 먼저
sam deploy
```

### 실수 2: body를 dict로 반환

```python
# 오류 발생
return {
    "statusCode": 200,
    "body": {"message": "hello"}  # dict를 직접 반환하면 오류
}

# 올바른 방법
return {
    "statusCode": 200,
    "body": json.dumps({"message": "hello"})  # 문자열로 변환
}
```

### 실수 3: 리전 불일치

`aws configure`에서 설정한 리전과 `sam deploy`에서 입력한 리전이 다르면 권한 오류가 납니다. 항상 `ap-northeast-2` (서울)로 통일합니다.

### 실수 4: queryStringParameters가 None

쿼리 파라미터가 없을 때 `event["queryStringParameters"]`는 `None`입니다. `.get()` 전에 `or {}`로 처리합니다.

```python
# 오류 발생 가능
params = event["queryStringParameters"]
name = params.get("name")  # params가 None이면 오류

# 안전한 방법
params = event.get("queryStringParameters") or {}
name = params.get("name", "World")
```

---

## 복습 체크리스트

- [ ] 콘솔 클릭 배포의 3가지 문제를 설명할 수 있다
- [ ] `sam init`으로 새 프로젝트를 만들 수 있다
- [ ] `template.yaml`의 `Resources` 섹션에서 함수 이름, 코드 경로, 핸들러, 런타임을 찾을 수 있다
- [ ] `sam build`를 언제 실행해야 하는지 안다
- [ ] `sam deploy --guided`의 각 질문에 답할 수 있다
- [ ] 배포 후 URL로 curl 테스트를 할 수 있다
- [ ] 코드 수정 후 `sam build && sam deploy`로 재배포할 수 있다
- [ ] `sam delete`로 리소스를 정리할 수 있다

---

## 참고 자료

- artifacts/05-aws-deploy/aws-chapter-02-account-and-lambda-setup.md
- artifacts/_internal/research-aws-lambda-patterns-2026.md
