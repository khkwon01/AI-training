# Chapter 07: DynamoDB로 데이터 영구 저장하기

## 이 장에서 배우는 것

- Lambda가 재시작되면 데이터가 사라지는 문제와 해결 방법
- DynamoDB가 무엇인지, Lambda와 어떻게 연결하는지
- Python에서 `boto3`로 DynamoDB에 데이터를 저장하고 읽는 방법
- 메모 서비스에 DynamoDB를 연결하는 실습
- 무료 티어 범위와 비용 관리

---

## 먼저 쉬운 설명

앞 장에서 만든 메모 서비스는 Lambda가 재시작되면 모든 메모가 사라진다.

Lambda는 요청이 올 때마다 새로 시작될 수 있기 때문에, **메모리에만 저장하는 방식은 실제 서비스에 쓸 수 없다**.

**DynamoDB**는 AWS에서 제공하는 데이터베이스 서비스다. Lambda와 함께 쓰면:

```
사용자 요청 → Lambda 함수 → DynamoDB에 저장/조회 → 응답 반환
```

Lambda가 재시작되어도 데이터는 DynamoDB에 남아있다.

---

## 1. DynamoDB 기본 개념

DynamoDB는 **NoSQL** 데이터베이스다. 엑셀 표처럼 행과 열이 정해진 형식이 아니라, 딕셔너리처럼 자유로운 구조로 저장한다.

### 핵심 용어

| 용어 | 의미 | 비유 |
|------|------|------|
| **Table** | 데이터를 저장하는 공간 | 엑셀 시트 |
| **Item** | 테이블의 데이터 한 줄 | 엑셀 행 |
| **Attribute** | 아이템의 각 값 | 엑셀 셀 |
| **Partition Key** | 각 아이템을 구분하는 고유 키 | 주민등록번호 |

### 메모 서비스에서의 구조

```
Table: memos
  Item: { "id": "1", "text": "Python 공부", "created": "2026-05-15" }
  Item: { "id": "2", "text": "GitHub 실습", "created": "2026-05-15" }
```

---

## 2. DynamoDB 테이블 만들기

### AWS 콘솔에서 생성

1. AWS 콘솔 → **DynamoDB** 검색 → **Create table**
2. 설정:
   - **Table name**: `memos`
   - **Partition key**: `id` (String)
3. **Settings**: **Default settings** 선택 (무료 티어 포함)
4. **Create table** 클릭

테이블 상태가 **Active**가 되면 준비 완료다.

---

## 3. Lambda에 DynamoDB 권한 추가

Lambda가 DynamoDB에 접근하려면 IAM 권한이 필요하다.

1. Lambda 함수 → **Configuration** 탭 → **Permissions**
2. **Role name** 클릭 (IAM 콘솔로 이동)
3. **Add permissions** → **Attach policies**
4. `AmazonDynamoDBFullAccess` 검색 → 선택 → **Add permissions**

---

## 4. Python에서 DynamoDB 사용하기 (boto3)

Lambda 환경에는 `boto3`가 기본으로 설치되어 있다.

### 기본 연결

```python
import boto3

# DynamoDB 리소스 생성 (Lambda에서는 리전 자동 감지)
dynamodb = boto3.resource("dynamodb", region_name="ap-northeast-2")
table = dynamodb.Table("memos")
```

### 항목 저장 (put_item)

```python
import uuid
import datetime

def save_memo(text):
    """메모를 DynamoDB에 저장한다."""
    item = {
        "id": str(uuid.uuid4()),      # 고유 ID 자동 생성
        "text": text,
        "created": datetime.datetime.now().isoformat()
    }
    table.put_item(Item=item)
    return item
```

### 전체 조회 (scan)

```python
def get_all_memos():
    """저장된 모든 메모를 가져온다."""
    response = table.scan()
    return response.get("Items", [])
```

### 특정 항목 삭제 (delete_item)

```python
def delete_memo(memo_id):
    """ID로 메모를 삭제한다."""
    table.delete_item(Key={"id": memo_id})
```

### 키워드 검색 (scan with filter)

```python
from boto3.dynamodb.conditions import Attr

def search_memos(keyword):
    """키워드가 포함된 메모를 검색한다."""
    response = table.scan(
        FilterExpression=Attr("text").contains(keyword)
    )
    return response.get("Items", [])
```

---

## 5. DynamoDB 연동된 Lambda 함수

```python
import json
import uuid
import datetime
import boto3
from boto3.dynamodb.conditions import Attr

dynamodb = boto3.resource("dynamodb", region_name="ap-northeast-2")
table = dynamodb.Table("memos")

def lambda_handler(event, context):
    print(f"[INPUT] {json.dumps(event)}")

    method = event.get("requestContext", {}).get("http", {}).get("method", "GET")
    params = event.get("queryStringParameters") or {}

    try:
        if method == "GET":
            keyword = params.get("search")
            if keyword:
                items = table.scan(
                    FilterExpression=Attr("text").contains(keyword)
                ).get("Items", [])
            else:
                items = table.scan().get("Items", [])
            body, status = {"memos": items, "count": len(items)}, 200

        elif method == "POST":
            text = params.get("text", "").strip()
            if not text:
                body, status = {"error": "빈 메모는 추가할 수 없습니다"}, 400
            else:
                item = {
                    "id": str(uuid.uuid4()),
                    "text": text,
                    "created": datetime.datetime.now().isoformat()
                }
                table.put_item(Item=item)
                body, status = {"message": "저장됨", "item": item}, 200

        elif method == "DELETE":
            memo_id = params.get("id")
            if not memo_id:
                body, status = {"error": "id가 필요합니다"}, 400
            else:
                table.delete_item(Key={"id": memo_id})
                body, status = {"message": f"삭제됨: {memo_id}"}, 200

        else:
            body, status = {"error": "지원하지 않는 메서드"}, 405

    except Exception as e:
        print(f"[ERROR] {e}")
        body, status = {"error": "Internal Server Error"}, 500

    print(f"[OUTPUT] status={status}")
    return {
        "statusCode": status,
        "headers": {"Content-Type": "application/json; charset=utf-8"},
        "body": json.dumps(body, ensure_ascii=False, default=str)
    }
```

---

## 6. 따라 하기 실습

### 실습 1. DynamoDB 테이블 만들기

1. AWS 콘솔 → DynamoDB → **Create table**
2. Table name: `memos`, Partition key: `id` (String)
3. Default settings → **Create table**
4. Status: **Active** 확인

### 실습 2. Lambda에 권한 추가하고 코드 배포

1. Lambda `memo-service` → Configuration → Permissions
2. IAM 역할에 `AmazonDynamoDBFullAccess` 추가
3. 위 코드를 Lambda 편집기에 붙여넣기 → **Deploy**

### 실습 3. 메모 저장하고 조회하기

```bash
# 메모 추가
curl -X POST "https://xxxxx.lambda-url.ap-northeast-2.on.aws/?text=DynamoDB+테스트"

# 목록 조회
curl "https://xxxxx.lambda-url.ap-northeast-2.on.aws/"

# Lambda를 재배포해도 메모가 남아있는지 확인
```

### 실습 4. DynamoDB 콘솔에서 직접 확인

1. DynamoDB → **Explore items** → `memos` 테이블
2. 저장된 항목 확인

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| 권한 없음 | `AccessDeniedException` | Lambda 실행 역할에 DynamoDB 권한 추가 |
| 리전 불일치 | 테이블을 찾지 못함 | Lambda와 DynamoDB가 같은 리전인지 확인 |
| Partition key 없음 | `ValidationException` | `put_item` 시 반드시 `id` 포함 |
| `scan`이 느림 | 데이터 많을 때 지연 | 소규모 실습에서는 문제없음, 실무는 query 사용 |

---

## 확인 체크리스트

- [ ] DynamoDB 테이블을 만들고 Active 상태를 확인했는가
- [ ] Lambda에 DynamoDB 권한을 추가했는가
- [ ] 메모를 저장하고 Lambda 재배포 후에도 데이터가 남아있는가
- [ ] DynamoDB 콘솔에서 저장된 항목을 직접 확인했는가

---

## 무료 티어 범위

| 항목 | 무료 범위 |
|------|----------|
| 읽기 용량 | 월 25 RCU (Read Capacity Units) |
| 쓰기 용량 | 월 25 WCU (Write Capacity Units) |
| 저장 공간 | 25 GB |

실습 수준의 사용량은 무료 범위를 초과하지 않는다.

---

## 한 번 더 생각해 보기

1. Lambda 메모리 저장과 DynamoDB 저장의 차이는 무엇인가?
2. `scan`이 아닌 `query`는 언제 써야 하는가?
3. DynamoDB가 없던 시절 Lambda는 데이터를 어떻게 저장했을까?

---

## 다음 장

다음 장에서는 지금까지 배운 모든 것을 통합해서 완성형 서비스를 만든다.

---

## 참고 자료

- AWS DynamoDB 개발자 가이드 — https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/
- boto3 DynamoDB — https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb.html
- DynamoDB 무료 티어 — https://aws.amazon.com/dynamodb/pricing/
