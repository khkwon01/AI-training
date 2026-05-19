## 이 장에서 배우는 것

- GitHub, AWS, DynamoDB를 하나의 흐름으로 연결하는 방법
- AI 도구(바이브 코딩)를 활용해 전체 서비스를 빠르게 만드는 방법
- 실제 서비스처럼 코드를 작성하고, 배포하고, 데이터를 저장하는 흐름 이해
- 각 도구가 왜 필요한지, 서로 어떻게 연결되는지 파악
- 막힐 때 AI에게 어떻게 물어봐야 하는지 알기

---

## 먼저 쉬운 설명

지금까지 배운 것들을 떠올려 보세요.

- **GitHub**: 코드를 저장하고 버전을 관리하는 곳
- **AWS**: 내 서비스를 인터넷에 올려 실제로 실행되게 하는 곳
- **DynamoDB**: 사용자 데이터를 저장하는 데이터베이스

하지만 이것들을 따로따로 배우는 것과, **하나의 서비스로 연결**하는 것은 다릅니다.

마치 재료를 각각 아는 것과, 요리를 완성하는 것의 차이예요.

이 장에서는 처음부터 끝까지 — 코드 작성 → GitHub에 저장 → AWS에 배포 → DynamoDB에 데이터 저장 — 이 전체 흐름을 **바이브 코딩** 방식으로 직접 만들어 봅니다.

바이브 코딩이란, AI에게 "이런 걸 만들고 싶어"라고 말하고, AI가 만들어 준 코드를 이해하면서 수정하는 방식입니다. 완벽하게 외울 필요 없이, 흐름을 이해하는 것이 목표입니다.

---

## 1. 전체 구조 이해하기

우리가 만들 서비스의 구조는 다음과 같습니다.

```
[사용자]
   ↓  HTTP 요청
[AWS Lambda 함수]  ← 코드가 실행되는 곳
   ↓  데이터 읽기/쓰기
[DynamoDB]         ← 데이터가 저장되는 곳
   ↑
[GitHub]           ← 코드를 관리하는 곳
```

실제 파일 구조를 미리 확인해 봅시다.

```
my-vibe-service/
├── handler.py          ← Lambda 함수 코드
├── requirements.txt    ← 필요한 라이브러리 목록
└── .github/
    └── workflows/
        └── deploy.yml  ← 자동 배포 설정
```

AI에게 이렇게 물어보세요:

```
"AWS Lambda에서 DynamoDB에 데이터를 저장하는
가장 간단한 Python 함수를 만들어 줘.
테이블 이름은 'UserVisits'이고,
사용자 ID와 방문 시간을 저장하고 싶어."
```

---

## 2. Lambda 함수 코드 작성하기

AI가 만들어 준 코드를 `handler.py`에 저장합니다.

```python
# handler.py
import json
import boto3
from datetime import datetime

# DynamoDB 연결
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('UserVisits')

def lambda_handler(event, context):
    # 요청에서 사용자 ID 가져오기
    user_id = event.get('user_id', 'unknown')
    
    # 현재 시간 기록
    visit_time = datetime.now().isoformat()
    
    # DynamoDB에 저장
    table.put_item(
        Item={
            'user_id': user_id,
            'visit_time': visit_time
        }
    )
    
    # 성공 응답 반환
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': f'{user_id}님의 방문이 기록되었습니다.',
            'time': visit_time
        })
    }
```

코드를 이해하는 핵심 포인트:

| 코드 줄 | 의미 |
|---|---|
| `boto3.resource('dynamodb')` | AWS DynamoDB에 연결 |
| `dynamodb.Table('UserVisits')` | 'UserVisits' 테이블 선택 |
| `table.put_item(...)` | 테이블에 데이터 저장 |
| `return { 'statusCode': 200 ... }` | 성공했다고 응답 |

---

## 3. GitHub에 코드 올리기

코드를 작성했으면 GitHub에 저장합니다. 터미널에서 아래 명령어를 순서대로 입력하세요.

```bash
# 1. 새 폴더 만들고 이동
mkdir my-vibe-service
cd my-vibe-service

# 2. Git 저장소 초기화
git init

# 3. handler.py 파일 생성 후 (위의 코드 붙여넣기)

# 4. 변경 사항 추가
git add handler.py

# 5. 커밋 (저장)
git commit -m "첫 번째 Lambda 함수 추가"

# 6. GitHub에 올리기
git remote add origin https://github.com/내아이디/my-vibe-service.git
git push -u origin main
```

GitHub에 올라간 코드를 확인하는 방법:
브라우저에서 `github.com/내아이디/my-vibe-service`를 열면 `handler.py` 파일이 보여야 합니다.

---

## 4. DynamoDB 테이블 만들기

AWS 콘솔에서 직접 테이블을 만들거나, AI에게 CLI 명령어를 요청할 수 있습니다.

```bash
# AWS CLI로 DynamoDB 테이블 만들기
aws dynamodb create-table \
    --table-name UserVisits \
    --attribute-definitions \
        AttributeName=user_id,AttributeType=S \
    --key-schema \
        AttributeName=user_id,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region ap-northeast-2
```

테이블이 만들어졌는지 확인:

```bash
aws dynamodb describe-table \
    --table-name UserVisits \
    --query 'Table.TableStatus'
```

결과가 `"ACTIVE"`이면 성공입니다.

---

## 5. Lambda 함수 배포하고 테스트하기

코드를 Lambda에 올리고 실제로 실행해 봅니다.

```bash
# 코드를 zip 파일로 압축
zip function.zip handler.py

# Lambda 함수 생성 (처음 배포할 때)
aws lambda create-function \
    --function-name my-vibe-service \
    --zip-file fileb://function.zip \
    --handler handler.lambda_handler \
    --runtime python3.12 \
    --role arn:aws:iam::내계정번호:role/lambda-role

# Lambda 함수 직접 실행 테스트
aws lambda invoke \
    --function-name my-vibe-service \
    --payload '{"user_id": "kim123"}' \
    response.json

# 결과 확인
cat response.json
```

성공하면 이런 결과가 나옵니다:

```json
{
    "statusCode": 200,
    "body": "{\"message\": \"kim123님의 방문이 기록되었습니다.\", \"time\": \"2026-05-15T10:30:00\"}"
}
```

---

## 따라 하기 실습

### 실습 1: 방문 기록 서비스 만들기

아래 파일을 직접 만들어 보세요.

**파일명:** `handler.py`

1. 위의 코드를 복사해서 `handler.py`에 저장합니다.
2. AI에게 이렇게 물어보세요: `"handler.py 코드에서 user_id가 없을 때 에러 메시지를 한국어로 반환하도록 수정해 줘"`
3. AI가 수정해 준 코드와 원래 코드의 차이를 찾아보세요.

---

### 실습 2: 데이터 조회 기능 추가하기

실습 1에서 만든 `handler.py`에 조회 기능을 추가합니다.

**AI에게 이렇게 요청해 보세요:**

```
"handler.py에 user_id로 방문 기록을 조회하는
get_visits 함수를 추가해 줘.
event에 'action': 'get'이 오면 조회하고,
'action': 'put'이 오면 저장하도록 해줘."
```

AI가 만들어 준 코드를 `handler.py`에 반영하고, GitHub에 커밋합니다:

```bash
git add handler.py
git commit -m "방문 기록 조회 기능 추가"
git push
```

---

### 실습 3: 전체 흐름 확인하기

실습 2까지 완성했으면, 전체 흐름을 직접 테스트합니다.

**저장 테스트:**

```bash
aws lambda invoke \
    --function-name my-vibe-service \
    --payload '{"action": "put", "user_id": "park456"}' \
    response.json
cat response.json
```

**조회 테스트:**

```bash
aws lambda invoke \
    --function-name my-vibe-service \
    --payload '{"action": "get", "user_id": "park456"}' \
    response.json
cat response.json
```

두 테스트 모두 성공하면, GitHub → Lambda → DynamoDB 전체 연결이 완성된 것입니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|---|---|---|
| 테이블 이름 오타 | `ResourceNotFoundException: Requested resource not found` | `aws dynamodb list-tables`로 테이블 이름 확인 |
| IAM 권한 없음 | `AccessDeniedException: User is not authorized to perform: dynamodb:PutItem` | Lambda 실행 역할에 DynamoDB 권한 추가 |
| 리전 불일치 | `ResourceNotFoundException` (권한은 있는데 테이블을 못 찾음) | Lambda와 DynamoDB 리전이 같은지 확인 (`ap-northeast-2`) |
| JSON 형식 오류 | `JSONDecodeError: Expecting value` | `payload` 값의 따옴표 확인, 작은따옴표 대신 큰따옴표 사용 |
| zip 파일 누락 | `InvalidParameterValueException: Could not unzip uploaded file` | `handler.py`가 zip 최상단에 있는지 확인 |
| 함수 이름 오타 | `ResourceNotFoundException: Function not found` | `aws lambda list-functions`로 함수 이름 확인 |
| Python 핸들러 경로 오류 | `Runtime.HandlerNotFound: handler.lambda_handler is undefined` | `--handler` 값이 `파일명.함수명` 형식인지 확인 |

---

## 확인 체크리스트

- [ ] `handler.py` 파일을 직접 만들었다
- [ ] AI에게 코드 수정을 요청하고, 수정된 내용을 이해했다
- [ ] `git add`, `git commit`, `git push` 순서를 기억한다
- [ ] DynamoDB 테이블 `UserVisits`가 `ACTIVE` 상태이다
- [ ] Lambda 함수가 정상적으로 배포되었다
- [ ] 저장(`put`) 테스트에서 `statusCode: 200`을 받았다
- [ ] 조회(`get`) 테스트에서 저장한 데이터가 돌아왔다
- [ ] GitHub 저장소에 최신 코드가 올라가 있다

---

## 한 번 더 생각해 보기

1. **Lambda 함수가 DynamoDB에 접근하려면 왜 IAM 권한이 필요할까요?** 만약 모든 함수가 모든 테이블에 접근할 수 있다면 어떤 문제가 생길까요?

2. **코드를 GitHub에 올리는 것과 Lambda에 배포하는 것은 다른 행동입니다.** 현재 실습에서 GitHub에 코드를 올려도 Lambda는 자동으로 업데이트되지 않습니다. 이 두 단계를 자동으로 연결하려면 무엇이 필요할까요?

3. **바이브 코딩으로 만든 코드를 그냥 쓰면 될까요?** AI가 만들어 준 코드에서 반드시 확인해야 할 것이 있다면 무엇일까요?

---

## 다음 장

다음 장에서는 GitHub Actions를 이용해 코드를 푸시하면 Lambda가 자동으로 배포되는 **CI/CD 파이프라인**을 만들어 봅니다.