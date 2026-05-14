# ai-train AWS Chapter 02: AWS 계정 만들기와 Lambda 첫 배포

## 이 장에서 배우는 것

- AWS 계정을 만들고 보안 설정을 하는 방법
- IAM에서 최소 권한 사용자를 만드는 이유와 방법
- AWS Lambda가 무엇인지, 왜 초보자에게 적합한지
- Python 함수를 Lambda에 올리는 단계별 방법
- 배포 후 테스트하고 로그를 확인하는 방법

---

## 먼저 쉬운 설명

서버를 직접 사서 운영하면 비용도 크고 관리도 복잡하다.

**AWS Lambda**는 코드만 올리면 AWS가 서버를 대신 관리해 주는 서비스다. 코드가 실행될 때만 비용이 발생하고, 월 100만 회 실행까지는 무료다.

초보자가 "내가 만든 Python 코드를 인터넷에 올린다"는 경험을 하기에 가장 빠른 방법이다.

---

## 1. AWS 계정 만들기

### 가입 절차

1. https://aws.amazon.com 접속
2. 오른쪽 위 **Create an AWS Account** 클릭
3. 이메일 주소, 비밀번호, 계정 이름 입력
4. 연락처 정보 입력 (Personal 선택)
5. 신용카드 등록 (무료 티어 사용 시 청구 없음, 본인 확인용)
6. 전화 인증 완료
7. **Free** 플랜 선택 → 가입 완료

### 가입 후 반드시 할 일: MFA 설정

루트 계정(가입 시 이메일)에 MFA(다중 인증)를 설정해야 한다. MFA가 없으면 계정이 탈취될 경우 큰 요금이 발생할 수 있다.

1. AWS 콘솔 오른쪽 위 계정 이름 클릭 → **Security credentials**
2. **Multi-factor authentication (MFA)** → **Assign MFA device**
3. 스마트폰에 **Google Authenticator** 또는 **Authy** 앱 설치
4. QR 코드 스캔 → 인증 코드 2번 입력

---

## 2. IAM 사용자 만들기

루트 계정으로 직접 작업하면 위험하다. 작업용 IAM 사용자를 따로 만들어서 쓴다.

### IAM 사용자 생성

1. AWS 콘솔 검색창에 `IAM` 입력 → **IAM** 서비스 클릭
2. 왼쪽 메뉴 **Users** → **Create user**
3. 사용자 이름 입력: 예) `my-dev-user`
4. **Provide user access to the AWS Management Console** 체크
5. **I want to create an IAM user** 선택 → 비밀번호 설정
6. **Next**

### 권한 설정

**Attach policies directly** 선택 후 아래 정책 추가:

| 정책 이름 | 용도 |
|-----------|------|
| `AWSLambda_FullAccess` | Lambda 함수 생성/실행 |
| `CloudWatchLogsFullAccess` | 로그 확인 |
| `IAMReadOnlyAccess` | IAM 정보 읽기 |

7. **Next** → **Create user**

### IAM 사용자로 로그인

계정 ID (12자리 숫자)와 IAM 사용자 이름, 비밀번호로 로그인:

```
https://계정ID.signin.aws.amazon.com/console
```

---

## 3. AWS Lambda 이해하기

Lambda는 **함수(Function) 단위**로 코드를 올리는 서비스다.

```
내 Python 코드 → Lambda 함수 → 실행 요청이 올 때만 실행
```

### Lambda의 특징

| 항목 | 내용 |
|------|------|
| 서버 관리 | AWS가 자동으로 처리 |
| 비용 | 실행 시간만큼만 청구 |
| 무료 범위 | 월 100만 요청, 400,000 GB-초 |
| 실행 제한 | 최대 15분, 메모리 최대 10GB |
| 지원 언어 | Python, Node.js, Java 등 |

초보자 실습에는 무료 범위를 초과하지 않는다.

---

## 4. Python 함수를 Lambda에 올리기

### 4-1. 함수 코드 준비

아래 코드를 `lambda_function.py` 파일로 만든다.

```python
def lambda_handler(event, context):
    """
    Lambda가 실행할 때 자동으로 호출하는 함수.
    event: 입력 데이터 (딕셔너리)
    context: 실행 환경 정보 (자동 전달)
    """
    name = event.get("name", "World")
    message = f"안녕하세요, {name}님!"
    
    print(f"실행됨: {message}")  # CloudWatch Logs에 기록됨
    
    return {
        "statusCode": 200,
        "body": message
    }
```

Lambda 함수는 반드시 `lambda_handler(event, context)` 형태여야 한다.

### 4-2. Lambda 함수 생성

1. AWS 콘솔 검색창에 `Lambda` 입력 → **Lambda** 서비스 클릭
2. **Create function** 클릭
3. 설정:
   - **Author from scratch** 선택
   - Function name: `hello-python`
   - Runtime: **Python 3.12**
   - Architecture: `x86_64`
4. **Create function** 클릭

### 4-3. 코드 업로드

함수 생성 후 나타나는 코드 편집기에 `lambda_function.py` 코드를 붙여넣는다.

기존 내용을 전부 지우고 위 코드를 그대로 입력.

**Deploy** 버튼 클릭 → `Changes deployed` 메시지 확인

---

## 5. 테스트 실행하기

### 테스트 이벤트 만들기

1. 코드 편집기 위 **Test** 탭 클릭
2. **Create new test event** 선택
3. Event name: `test-hello`
4. Event JSON 입력:

```json
{
  "name": "Mina"
}
```

5. **Save** → **Test** 클릭

### 결과 확인

성공하면 아래처럼 표시된다.

```
Response
{
  "statusCode": 200,
  "body": "안녕하세요, Mina님!"
}

Function logs
START RequestId: ...
실행됨: 안녕하세요, Mina님!
END RequestId: ...
REPORT Duration: 1.23 ms
```

---

## 6. CloudWatch Logs에서 로그 확인하기

Lambda가 실행될 때마다 로그가 CloudWatch Logs에 저장된다.

1. Lambda 함수 페이지 → **Monitor** 탭
2. **View CloudWatch logs** 클릭
3. 가장 최근 **Log stream** 클릭
4. `print()` 로 출력한 내용이 보임

---

## 따라 하기 실습

### 실습 1. AWS 계정 만들고 IAM 사용자 생성

1. AWS 계정 가입 (루트 계정에 MFA 설정 포함)
2. IAM 사용자 생성 (`my-dev-user`)
3. Lambda 권한 정책 추가
4. IAM 사용자로 로그인

### 실습 2. Lambda 함수 만들고 배포하기

1. Lambda 콘솔에서 `hello-python` 함수 생성
2. 위 코드 붙여넣기
3. **Deploy** 클릭

### 실습 3. 테스트 이벤트 실행하고 로그 확인

1. 테스트 이벤트 `{"name": "내 이름"}` 으로 실행
2. Response에서 메시지 확인
3. CloudWatch Logs에서 `print()` 출력 확인

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| 루트 계정으로 작업 | 보안 위험 | IAM 사용자로 전환 |
| MFA 미설정 | 계정 탈취 위험 | Security credentials → MFA 설정 |
| 함수 이름이 `lambda_handler` 아님 | 실행 오류 | Handler 설정 확인 (기본값: `lambda_function.lambda_handler`) |
| Deploy 없이 Test 실행 | 수정 전 코드가 실행됨 | 코드 수정 후 반드시 **Deploy** 클릭 |
| 리전이 달라서 함수 안 보임 | 만든 함수가 없어 보임 | 콘솔 오른쪽 위 리전 확인 (서울: `ap-northeast-2`) |

---

## 확인 체크리스트

- [ ] AWS 계정이 생성되어 있고 MFA가 설정되어 있는가
- [ ] IAM 사용자로 로그인할 수 있는가
- [ ] Lambda 함수를 생성하고 코드를 배포할 수 있는가
- [ ] 테스트 이벤트를 만들고 실행할 수 있는가
- [ ] CloudWatch Logs에서 실행 로그를 찾을 수 있는가

---

## 한 번 더 생각해 보기

1. 루트 계정 대신 IAM 사용자를 쓰는 이유는 무엇인가?
2. Lambda 함수가 실행될 때마다 비용이 발생한다면, 월 무료 범위를 초과하려면 몇 번이나 실행해야 할까?
3. `print()` 로 출력한 내용이 CloudWatch Logs에 저장되는 것이 왜 유용한가?

---

## 다음 장

다음 장에서는 Lambda 함수를 API Gateway와 연결해서 URL로 호출할 수 있게 만든다. URL이 생기면 브라우저나 다른 프로그램에서 내 함수를 불러올 수 있다.

---

## 참고 자료

- AWS Lambda 시작 가이드 — https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html
- AWS 무료 티어 — https://aws.amazon.com/free
- IAM 사용자 생성 — https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html
- CloudWatch Logs — https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html
