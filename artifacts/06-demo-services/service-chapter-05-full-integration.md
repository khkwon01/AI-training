# Chapter 05: 완성형 서비스 통합 실습

## 이 장에서 배우는 것

- 지금까지 배운 Python, GitHub, AI, AWS를 하나의 흐름으로 연결하기
- DynamoDB를 연결한 메모 서비스를 처음부터 끝까지 혼자 완성하기
- GitHub Actions로 push만 하면 자동 배포되는 파이프라인 구성하기
- 완성된 서비스를 CloudWatch로 모니터링하기
- 다음 서비스를 만들 때 재사용할 수 있는 패턴 정리하기

---

## 이 장의 목표

```
내 컴퓨터 Python 코드
    ↓  GitHub push
GitHub 저장소
    ↓  GitHub Actions 자동 실행
AWS Lambda (Python 코드 배포)
    ↓  Function URL 호출
DynamoDB (데이터 영구 저장)
    ↓  CloudWatch 로그
운영 모니터링
```

이 흐름 전체가 자동화된 서비스를 완성한다.

---

## Part 1. 프로젝트 구조 설계

### 폴더 구조

```
memo-service/
├── lambda_function.py      ← 메인 Lambda 코드
├── requirements.txt        ← 의존 패키지 (boto3는 AWS 기본 포함)
├── tests/
│   └── test_memo.py        ← 단위 테스트
├── RUNBOOK.md              ← 배포 런북
└── .github/
    └── workflows/
        └── deploy.yml      ← GitHub Actions 자동 배포
```

### GitHub 저장소 초기화

```bash
mkdir memo-service && cd memo-service
git init
git remote add origin https://github.com/USERNAME/memo-service.git

# .gitignore 만들기
echo ".env
__pycache__/
*.pyc
.pytest_cache/" > .gitignore
```

---

## Part 2. DynamoDB 연동 Lambda 코드

### lambda_function.py

```python
import json
import uuid
import datetime
import boto3
from boto3.dynamodb.conditions import Attr

# Lambda 환경 변수에서 테이블 이름 읽기
import os
TABLE_NAME = os.environ.get("DYNAMODB_TABLE", "memos")

dynamodb = boto3.resource("dynamodb", region_name="ap-northeast-2")
table = dynamodb.Table(TABLE_NAME)


def add_memo(text):
    if not text or not text.strip():
        return {"error": "빈 메모는 추가할 수 없습니다"}, 400
    item = {
        "id": str(uuid.uuid4()),
        "text": text.strip(),
        "created": datetime.datetime.now().isoformat()
    }
    table.put_item(Item=item)
    return {"message": "저장됨", "item": item}, 200


def get_memos(keyword=None):
    if keyword:
        items = table.scan(
            FilterExpression=Attr("text").contains(keyword)
        ).get("Items", [])
    else:
        items = table.scan().get("Items", [])
    return {"memos": items, "count": len(items)}, 200


def delete_memo(memo_id):
    if not memo_id:
        return {"error": "id가 필요합니다"}, 400
    table.delete_item(Key={"id": memo_id})
    return {"message": f"삭제됨: {memo_id}"}, 200


def lambda_handler(event, context):
    print(f"[INPUT] method={event.get('requestContext',{}).get('http',{}).get('method')} params={event.get('queryStringParameters')}")

    method = event.get("requestContext", {}).get("http", {}).get("method", "GET")
    params = event.get("queryStringParameters") or {}

    try:
        if method == "GET":
            body, status = get_memos(params.get("search"))
        elif method == "POST":
            body, status = add_memo(params.get("text", ""))
        elif method == "DELETE":
            body, status = delete_memo(params.get("id"))
        else:
            body, status = {"error": "지원하지 않는 메서드"}, 405
    except Exception as e:
        print(f"[ERROR] {e}")
        body, status = {"error": "서버 오류가 발생했습니다"}, 500

    print(f"[OUTPUT] status={status}")
    return {
        "statusCode": status,
        "headers": {"Content-Type": "application/json; charset=utf-8"},
        "body": json.dumps(body, ensure_ascii=False, default=str)
    }
```

---

## Part 3. 테스트 작성

### tests/test_memo.py

```python
import unittest
from unittest.mock import MagicMock, patch
import sys, os
sys.path.insert(0, os.path.dirname(os.path.dirname(os.path.abspath(__file__))))

# DynamoDB를 mock으로 대체해서 테스트
with patch("boto3.resource") as mock_resource:
    mock_table = MagicMock()
    mock_resource.return_value.Table.return_value = mock_table
    from lambda_function import add_memo, delete_memo


class TestAddMemo(unittest.TestCase):

    def setUp(self):
        mock_table.reset_mock()

    def test_normal(self):
        body, status = add_memo("테스트 메모")
        self.assertEqual(status, 200)
        self.assertIn("item", body)
        mock_table.put_item.assert_called_once()

    def test_empty(self):
        body, status = add_memo("")
        self.assertEqual(status, 400)
        mock_table.put_item.assert_not_called()

    def test_whitespace_only(self):
        body, status = add_memo("   ")
        self.assertEqual(status, 400)


class TestDeleteMemo(unittest.TestCase):

    def setUp(self):
        mock_table.reset_mock()

    def test_normal(self):
        body, status = delete_memo("some-uuid")
        self.assertEqual(status, 200)
        mock_table.delete_item.assert_called_once()

    def test_no_id(self):
        body, status = delete_memo(None)
        self.assertEqual(status, 400)
        mock_table.delete_item.assert_not_called()


if __name__ == "__main__":
    unittest.main()
```

로컬에서 실행:

```bash
python3 -m pytest tests/ -v
```

---

## Part 4. GitHub Actions 자동 배포

### .github/workflows/deploy.yml

```yaml
name: Deploy to AWS Lambda

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Run tests
        run: python -m pytest tests/ -v

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: Deploy to Lambda
        run: |
          zip deployment.zip lambda_function.py
          aws lambda update-function-code \
            --function-name memo-service \
            --zip-file fileb://deployment.zip
          echo "✅ 배포 완료"

      - name: Set environment variable
        run: |
          aws lambda update-function-configuration \
            --function-name memo-service \
            --environment Variables="{DYNAMODB_TABLE=memos}"
```

---

## Part 5. 전체 흐름 실습

### 1단계. 로컬 개발

```bash
# 새 기능 브랜치 만들기
git checkout -b feature/search-by-date

# lambda_function.py에 날짜 검색 기능 추가 (AI 활용)
# "날짜 범위로 메모를 검색하는 search_by_date(start, end) 함수를 추가해줘"
```

### 2단계. 테스트 통과

```bash
python3 -m pytest tests/ -v
# 새 기능의 테스트도 추가하고 통과시키기
```

### 3단계. GitHub PR

```bash
git add .
git commit -m "feat: 날짜 범위 검색 기능 추가 (#5)"
git push origin feature/search-by-date
```

GitHub에서 PR 생성 → Files changed 확인 → Merge

### 4단계. 자동 배포 확인

GitHub → Actions 탭에서 배포 진행 상황 확인:
```
✅ test
✅ deploy
```

### 5단계. 동작 검증

```bash
# 새 기능 테스트
curl "https://xxxxx.lambda-url.ap-northeast-2.on.aws/?search=Python"

# CloudWatch Logs 확인
```

---

## 완성 체크리스트

```
□ DynamoDB 테이블 생성 및 Lambda 권한 연결
□ lambda_function.py 작성 및 로컬 실행 확인
□ 단위 테스트 작성 및 통과
□ GitHub 저장소 생성 및 코드 push
□ GitHub Secrets에 AWS 인증 정보 저장
□ GitHub Actions workflow 파일 생성
□ main 브랜치 push → 자동 배포 확인
□ Lambda URL로 기능 테스트 (CRUD 전체)
□ CloudWatch Logs에서 로그 확인
□ RUNBOOK.md 작성
```

---

## 이 서비스를 기반으로 다음에 만들 수 있는 것

| 아이디어 | 추가 기술 |
|---------|----------|
| 사용자별 메모 관리 | 인증(JWT), DynamoDB GSI |
| 메모 공유 기능 | S3 파일 저장, CloudFront |
| 자동 분류 AI | Lambda + Claude API |
| 모바일 앱 연동 | API Gateway, CORS 설정 |

---

## 참고 자료

- 이 장의 전체 코드: GitHub 저장소에서 관리
- AWS Lambda + DynamoDB 아키텍처 — https://docs.aws.amazon.com/lambda/latest/dg/services-dynamodb.html
