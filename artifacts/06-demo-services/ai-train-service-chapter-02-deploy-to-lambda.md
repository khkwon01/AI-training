# ai-train Demo Services Chapter 02: 메모 서비스를 Lambda에 배포하기

## 이 장에서 배우는 것

- 터미널 기반 서비스를 API로 변환하는 방법
- Lambda에 맞게 코드를 수정하는 방법
- 메모 서비스의 각 기능을 API 엔드포인트로 만드는 방법
- 배포 후 URL로 테스트하는 방법
- 앞에서 배운 Python + GitHub + AI + AWS를 연결하는 전체 흐름

---

## 먼저 쉬운 설명

앞 장에서 만든 메모 서비스는 터미널에서만 실행된다. 인터넷에서 접근하려면 API로 바꿔야 한다.

Lambda는 `lambda_handler(event, context)` 함수 하나를 실행한다. 이 함수가 "어떤 기능을 실행할지"를 URL 경로나 쿼리 파라미터로 구분하면 여러 기능을 하나의 Lambda로 처리할 수 있다.

```
GET  /memos              → 전체 메모 목록
POST /memos?text=내용    → 메모 추가
DELETE /memos?id=1       → 메모 삭제
GET  /memos?search=키워드 → 메모 검색
```

---

## 1. Lambda용 메모 서비스 코드

Lambda에서는 파일 시스템이 임시(`/tmp`)이므로, 재시작하면 데이터가 사라진다. 여기서는 메모리(전역 변수)에 저장하는 방식으로 만든다.

```python
# lambda_function.py
import json

# 메모리에 저장 (Lambda가 재시작되면 초기화됨)
memos = []


def get_memos():
    return {"memos": memos, "count": len(memos)}


def add_memo(text):
    if not text or not text.strip():
        return {"error": "빈 메모는 추가할 수 없습니다"}, 400
    memos.append(text.strip())
    return {"message": f"메모 추가됨: {text}", "total": len(memos)}, 200


def delete_memo(index):
    try:
        idx = int(index) - 1  # 1-based → 0-based
        if idx < 0 or idx >= len(memos):
            return {"error": f"잘못된 번호: 1~{len(memos)} 사이로 입력"}, 400
        removed = memos.pop(idx)
        return {"message": f"삭제됨: {removed}", "total": len(memos)}, 200
    except (ValueError, TypeError):
        return {"error": "번호는 숫자여야 합니다"}, 400


def search_memos(keyword):
    if not keyword:
        return {"memos": memos, "count": len(memos)}
    results = [m for m in memos if keyword.lower() in m.lower()]
    return {"memos": results, "count": len(results), "keyword": keyword}


def lambda_handler(event, context):
    print(f"[INPUT] event: {json.dumps(event)}")

    method = event.get("requestContext", {}).get("http", {}).get("method", "GET")
    path = event.get("rawPath", "/")
    params = event.get("queryStringParameters") or {}

    print(f"[PROCESS] method={method}, path={path}, params={params}")

    # 라우팅
    if path == "/" or path == "/memos":
        if method == "GET":
            search = params.get("search")
            if search:
                body, status = search_memos(search), 200
            else:
                body, status = get_memos(), 200

        elif method == "POST":
            text = params.get("text", "")
            body, status = add_memo(text)

        elif method == "DELETE":
            index = params.get("id")
            body, status = delete_memo(index)

        else:
            body, status = {"error": "지원하지 않는 메서드"}, 405
    else:
        body, status = {"error": "경로를 찾을 수 없습니다"}, 404

    print(f"[OUTPUT] status={status}, body={body}")

    return {
        "statusCode": status,
        "headers": {"Content-Type": "application/json; charset=utf-8"},
        "body": json.dumps(body, ensure_ascii=False)
    }
```

---

## 2. Lambda에 배포하기

### 새 Lambda 함수 만들기

1. AWS Lambda 콘솔 → **Create function**
2. Function name: `memo-service`
3. Runtime: **Python 3.12**
4. **Create function**

### 코드 업로드

함수 생성 후 코드 편집기에 위 코드를 붙여넣고 **Deploy**.

### Function URL 설정

Configuration → Function URL → **Create function URL**
- Auth type: **NONE**
- **Save**

URL 저장해두기:
```
https://xxxxx.lambda-url.ap-northeast-2.on.aws/
```

---

## 3. 테스트하기

### 메모 추가

```bash
curl -X POST "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos?text=Python+공부하기"
```

```json
{"message": "메모 추가됨: Python 공부하기", "total": 1}
```

### 메모 목록 조회

```bash
curl "https://xxxxx.lambda-url.ap-northeast-2.on.aws/memos"
```

```json
{"memos": ["Python 공부하기"], "count": 1}
```

### Python 코드로 테스트

```python
import urllib.request
import json

BASE_URL = "https://xxxxx.lambda-url.ap-northeast-2.on.aws"

def call_api(path, method="GET"):
    url = BASE_URL + path
    req = urllib.request.Request(url, method=method)
    with urllib.request.urlopen(req) as res:
        return json.loads(res.read())

# 메모 추가
import urllib.parse
text = urllib.parse.quote("GitHub 실습하기")
print(call_api(f"/memos?text={text}", method="POST"))

# 목록 조회
print(call_api("/memos"))

# 검색
keyword = urllib.parse.quote("Python")
print(call_api(f"/memos?search={keyword}"))
```

---

## 4. GitHub에 코드 올리기

```bash
# 저장소 초기화 (없다면)
git init
git remote add origin https://github.com/username/memo-service.git

# 코드 추가
git add lambda_function.py
git commit -m "Lambda용 메모 서비스 API 구현"
git push origin main
```

---

## 5. 전체 흐름 정리

지금까지 배운 것들이 어떻게 연결됐는지 확인한다.

```
Python 기초 (챕터 1~15)
  → 조건문, 반복문, 함수, class, module로 서비스 구현

GitHub (챕터 1~7)
  → 코드를 저장소에 올리고 버전 관리

AI 도구 (챕터 1~5)
  → 프롬프트로 코드 요청, 검증, repair loop로 개선

Vibe coding (챕터 1~3)
  → AI와 함께 기능 단계별 추가

AWS (챕터 1~4)
  → Lambda + API Gateway + CloudWatch로 배포 및 모니터링

최종 결과: URL로 접근 가능한 메모 서비스 완성
```

---

## 6. 따라 하기 실습

### 실습 1. Lambda에 코드 배포하고 URL 테스트

1. `memo-service` Lambda 함수 생성
2. 코드 붙여넣기 → Deploy
3. Function URL 생성
4. `curl` 또는 브라우저로 각 기능 테스트

### 실습 2. AI로 기능 추가하기

AI에게 요청:
```
현재 Lambda 메모 서비스에 메모 수정 기능을 추가하고 싶어.
PUT /memos?id=1&text=새내용 형태로 호출하면 해당 번호 메모를 수정해야 해.
기존 lambda_handler 코드에 어떻게 추가하면 될까?
```

받은 코드를 검증하고 Deploy.

### 실습 3. CloudWatch로 로그 확인

각 API 호출 후 CloudWatch Logs에서:
- `[INPUT]`, `[PROCESS]`, `[OUTPUT]` 로그 확인
- 오류가 났을 때 traceback 확인

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| Lambda 재시작 후 메모 사라짐 | 데이터가 초기화됨 | 정상 동작 (메모리 저장 방식의 한계) |
| URL에 한글 포함 | 인코딩 오류 | `urllib.parse.quote()` 로 URL 인코딩 |
| CORS 오류 | 브라우저에서 API 호출 실패 | Function URL 설정에서 CORS 허용 |
| Deploy 안 하고 테스트 | 수정 전 코드가 실행됨 | 코드 수정 후 반드시 **Deploy** |

---

## 확인 체크리스트

- [ ] Lambda에 메모 서비스 코드를 배포할 수 있는가
- [ ] Function URL로 메모 추가/조회/삭제/검색을 테스트할 수 있는가
- [ ] CloudWatch Logs에서 API 호출 로그를 확인할 수 있는가
- [ ] GitHub에 코드를 commit하고 push했는가

---

## 한 번 더 생각해 보기

1. Lambda가 재시작되면 메모가 사라지는 문제를 해결하려면 어떻게 해야 할까?
2. 여러 사람이 같은 API를 동시에 사용하면 메모 데이터에 어떤 문제가 생길까?
3. 이 서비스를 실제로 사용하려면 무엇이 더 필요할까?

---

## 참고 자료

- AWS Lambda Function URL — https://docs.aws.amazon.com/lambda/latest/dg/lambda-urls.html
- Python urllib.request — https://docs.python.org/3/library/urllib.request.html
