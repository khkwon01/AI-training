## 이 장에서 배우는 것

- 배포 전에 확인해야 할 항목을 5가지 카테고리로 정리할 수 있다
- 코드 품질, 테스트, 보안, 환경설정, 모니터링 각각의 체크 포인트를 안다
- 체크리스트를 직접 따라가며 실제 배포 준비를 완료할 수 있다
- 배포 직후 서비스가 정상 동작하는지 빠르게 확인하는 방법을 안다
- 초보자가 자주 놓치는 항목과 그 결과를 미리 알 수 있다

---

## 먼저 쉬운 설명

코딩이 끝났다고 바로 배포하면 어떤 일이 생길까?

실제로 이런 일들이 일어난다:

- 로컬에서는 잘 됐는데 서버에서 `DATABASE_URL` 환경변수가 없어서 앱이 시작조차 안 된다
- API 키가 GitHub에 올라가서 하루 만에 요금이 100만 원 청구된다
- 에러가 나도 로그가 없어서 무슨 문제인지 알 수 없다
- 테스트를 안 돌렸더니 회원가입이 안 되는 버그가 있었다

배포는 "코드를 서버에 올리는 것"이 아니다. **"서비스가 사용자에게 안전하고 안정적으로 전달되는 것"**이다.

체크리스트는 이 과정을 빠짐없이 확인하기 위한 도구다. 경험 많은 개발자도 체크리스트를 쓴다. 기억에만 의존하면 반드시 빠뜨리는 항목이 생기기 때문이다.

이 장에서는 실제 서비스를 배포할 때 쓸 수 있는 체크리스트를 카테고리별로 배운다.

---

## 1. 코드 품질 체크리스트

배포 전에 코드 자체가 깨끗한지 확인한다. "동작하면 됐지"라는 생각은 나중에 큰 문제가 된다.

### 왜 코드 품질이 중요한가

나쁜 코드는 버그가 숨기 쉽고, 팀원이 읽기 어렵고, 고치다가 다른 곳이 망가진다. 배포 전에 정리하는 것이 배포 후에 고치는 것보다 훨씬 쉽다.

### 체크 항목

```
✅ 코드 품질 체크리스트

[ ] 1. 린터(linter) 오류가 없다
[ ] 2. 사용하지 않는 import, 변수가 없다
[ ] 3. print() 디버그 출력을 모두 제거했다
[ ] 4. TODO, FIXME 주석이 없다 (있다면 이슈로 옮겼다)
[ ] 5. 함수와 변수 이름이 명확하다 (a, b, temp 같은 이름 없음)
[ ] 6. 중복 코드를 함수로 정리했다
[ ] 7. 코드 리뷰를 받았거나 스스로 다시 읽었다
```

### 실제 코드 예시

배포 전에 이런 코드가 남아 있으면 안 된다:

```python
# ❌ 배포 전에 반드시 정리해야 할 코드
import os
import json
import requests  # 사용하지 않음

def get_user(id):
    # TODO: 나중에 캐싱 추가
    print(f"[DEBUG] get_user called with id={id}")  # 디버그 출력
    x = db.query("SELECT * FROM users WHERE id = ?", id)  # x가 뭔지 불명확
    temp = x[0] if x else None  # temp가 뭔지 불명확
    return temp
```

```python
# ✅ 정리된 코드
import os
import json

def get_user(user_id: int):
    result = db.query("SELECT * FROM users WHERE id = ?", user_id)
    return result[0] if result else None
```

### 린터 실행 방법

```bash
# Python 프로젝트
pip install flake8
flake8 . --max-line-length=100

# 출력 예시 (이런 오류가 없어야 한다)
# app/main.py:15:1: F401 'requests' imported but unused
# app/utils.py:32:5: E501 line too long (105 > 100 characters)
```

```bash
# JavaScript/Node.js 프로젝트
npm run lint

# 또는 직접 실행
npx eslint .
```

---

## 2. 테스트 체크리스트

코드가 실제로 의도대로 동작하는지 자동으로 확인하는 테스트를 실행한다.

### 왜 테스트가 중요한가

"내가 직접 써봤으니까 괜찮다"는 생각은 위험하다. 사람이 수동으로 테스트하면 모든 경우를 확인할 수 없다. 자동화된 테스트는 빠르고 정확하게 버그를 잡는다.

### 체크 항목

```
✅ 테스트 체크리스트

[ ] 1. 모든 테스트가 통과한다 (실패하는 테스트가 없다)
[ ] 2. 핵심 기능에 테스트가 있다 (회원가입, 로그인, 주요 API 등)
[ ] 3. 에러 케이스도 테스트되어 있다 (잘못된 입력, 없는 데이터 등)
[ ] 4. 테스트 커버리지가 일정 수준 이상이다 (최소 60% 권장)
[ ] 5. 테스트 데이터가 실제 데이터베이스에 영향을 주지 않는다
[ ] 6. 새로 추가한 기능에 테스트를 작성했다
```

### 테스트 실행 예시

```bash
# Python pytest
pytest --tb=short

# 출력 예시
# ========================= test session starts =========================
# collected 24 items
#
# tests/test_auth.py ..........                                   [ 41%]
# tests/test_users.py .........                                   [ 79%]
# tests/test_products.py .....                                    [100%]
#
# ========================= 24 passed in 3.42s =========================
```

```bash
# 커버리지 확인
pytest --cov=app --cov-report=term-missing

# 출력 예시
# Name                    Stmts   Miss  Cover   Missing
# -------------------------------------------------------
# app/main.py                45      2    96%   87-88
# app/auth.py                38      8    79%   45-52
# app/users.py               52      0   100%
# -------------------------------------------------------
# TOTAL                     135     10    93%
```

### 실패하는 테스트 예시

```python
# tests/test_auth.py
def test_login_with_wrong_password():
    response = client.post("/auth/login", json={
        "email": "user@example.com",
        "password": "wrong_password"
    })
    
    # 이 테스트가 실패하면 배포하지 않는다
    assert response.status_code == 401
    assert "error" in response.json()
```

```bash
# 테스트 실패 시 출력
# FAILED tests/test_auth.py::test_login_with_wrong_password
# AssertionError: assert 200 == 401
# → 잘못된 비밀번호인데 200을 반환하고 있다 → 심각한 보안 버그
```

---

## 3. 보안 체크리스트

보안 문제는 배포 후에 발생하면 피해가 크다. 배포 전에 반드시 확인한다.

### 왜 보안이 특히 중요한가

보안 사고는 단순한 버그와 다르다. 사용자 데이터가 유출되거나, 요금이 폭탄처럼 청구되거나, 서비스 자체가 해킹당할 수 있다. 모두 배포 전 체크리스트로 막을 수 있는 문제다.

### 체크 항목

```
✅ 보안 체크리스트

[ ] 1. API 키, 비밀번호, 토큰이 코드에 하드코딩되어 있지 않다
[ ] 2. .env 파일이 .gitignore에 등록되어 있다
[ ] 3. GitHub에 민감한 정보가 올라가 있지 않다 (git log 확인)
[ ] 4. 사용자 입력을 검증하고 있다 (SQL 인젝션, XSS 방지)
[ ] 5. 비밀번호를 평문으로 저장하지 않는다 (bcrypt, argon2 등 사용)
[ ] 6. HTTPS를 사용한다 (HTTP 리다이렉트 설정)
[ ] 7. 불필요한 포트가 열려 있지 않다
[ ] 8. 관리자 엔드포인트에 인증이 있다
```

### 흔한 보안 실수 — API 키 노출

```python
# ❌ 절대 이렇게 하면 안 된다
import openai

openai.api_key = "sk-proj-abc123xyz..."  # 코드에 직접 박아넣음

def ask_ai(question):
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": question}]
    )
    return response.choices[0].message.content
```

```python
# ✅ 환경변수로 관리한다
import os
import openai

openai.api_key = os.environ.get("OPENAI_API_KEY")

if not openai.api_key:
    raise ValueError("OPENAI_API_KEY 환경변수가 설정되지 않았습니다")

def ask_ai(question):
    response = openai.chat.completions.create(
        model="gpt-4",
        messages=[{"role": "user", "content": question}]
    )
    return response.choices[0].message.content
```

### .gitignore 확인

```bash
# .gitignore 파일에 반드시 있어야 할 항목들
cat .gitignore
```

```gitignore
# .gitignore
.env
.env.local
.env.production
*.pem
*.key
secrets/
__pycache__/
*.pyc
.DS_Store
node_modules/
```

### GitHub에 올라간 민감 정보 확인

```bash
# 과거 커밋에서 민감 정보 검색
git log --all --full-history --source -- .env
git grep -i "api_key" $(git rev-list --all)

# 이미 올라갔다면 → 즉시 키를 폐기하고 새 키를 발급한다
# 커밋을 지워도 이미 노출된 키는 사용 불가 처리해야 한다
```

### SQL 인젝션 방지

```python
# ❌ 위험한 코드 — SQL 인젝션에 취약
def get_user_by_name(name):
    query = f"SELECT * FROM users WHERE name = '{name}'"
    return db.execute(query)

# 공격자가 name에 "' OR '1'='1" 을 입력하면 모든 사용자가 조회된다
```

```python
# ✅ 안전한 코드 — 파라미터 바인딩 사용
def get_user_by_name(name):
    query = "SELECT * FROM users WHERE name = ?"
    return db.execute(query, (name,))  # 라이브러리가 안전하게 처리
```

---

## 4. 환경설정 체크리스트

서버 환경에서 앱이 제대로 동작하려면 환경설정이 정확해야 한다.

### 왜 환경설정이 까다로운가

"내 컴퓨터에서는 됐는데"라는 말이 나오는 가장 큰 이유가 환경설정 차이다. 로컬 개발 환경과 서버 환경은 다르다. 이 차이를 명시적으로 관리해야 한다.

### 체크 항목

```
✅ 환경설정 체크리스트

[ ] 1. 필요한 환경변수 목록이 문서화되어 있다 (.env.example 파일)
[ ] 2. 서버에 모든 환경변수가 설정되어 있다
[ ] 3. DEBUG 모드가 꺼져 있다 (DEBUG=False)
[ ] 4. 데이터베이스 연결이 프로덕션 DB를 가리키고 있다
[ ] 5. 의존성 패키지 버전이 고정되어 있다 (requirements.txt, package-lock.json)
[ ] 6. Python/Node.js 버전이 서버와 일치한다
[ ] 7. 서버 포트와 방화벽 설정이 일치한다
```

### .env.example 파일 만들기

```bash
# .env.example — 실제 값 없이 키 이름만 적은 파일
# 이 파일은 Git에 올린다 (민감 정보 없으므로)
```

```ini
# .env.example
# 데이터베이스 설정
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# 인증 설정
SECRET_KEY=your-secret-key-here
JWT_EXPIRE_MINUTES=30

# 외부 서비스
OPENAI_API_KEY=sk-...
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# 앱 설정
DEBUG=False
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
PORT=8000
```

### requirements.txt 생성 및 확인

```bash
# 현재 설치된 패키지를 requirements.txt로 저장
pip freeze > requirements.txt

# 내용 확인
cat requirements.txt
# fastapi==0.104.1
# uvicorn==0.24.0
# sqlalchemy==2.0.23
# bcrypt==4.0.1
# python-jose==3.3.0
```

```bash
# 서버에서 패키지 설치 확인
pip install -r requirements.txt

# 오류 없이 설치되면 OK
# 오류 예시:
# ERROR: Could not find a version that satisfies the requirement somepackage==1.2.3
# → 버전이 맞지 않거나 패키지가 없음
```

### DEBUG 모드 확인

```python
# config.py
import os

# ❌ 이렇게 하면 프로덕션에서도 DEBUG가 True가 될 수 있다
DEBUG = True

# ✅ 환경변수로 관리 — 기본값은 반드시 False
DEBUG = os.environ.get("DEBUG", "False").lower() == "true"

# DEBUG가 True이면 발생하는 문제:
# - 에러 메시지에 코드, 환경변수, DB 구조가 노출됨
# - 성능이 저하됨
# - 보안 설정이 완화됨
```

---

## 5. 모니터링 체크리스트

배포 후 서비스 상태를 알려면 모니터링이 필요하다. 모니터링 없이는 문제가 생겨도 사용자가 먼저 알게 된다.

### 왜 모니터링을 미리 설정해야 하는가

문제가 생긴 후에 로그를 보려면 이미 늦다. 에러가 쌓이고 있는지, 응답 시간이 느려지고 있는지, 서버가 다운됐는지를 **실시간으로** 알아야 빠르게 대응할 수 있다.

### 체크 항목

```
✅ 모니터링 체크리스트

[ ] 1. 앱 로그가 파일 또는 외부 서비스에 저장된다
[ ] 2. 에러가 발생하면 알림(슬랙, 이메일 등)이 온다
[ ] 3. 헬스체크 엔드포인트가 있다 (GET /health)
[ ] 4. 서버 리소스(CPU, 메모리, 디스크) 모니터링이 설정되어 있다
[ ] 5. 응답 시간(latency) 이 측정되고 있다
[ ] 6. 로그 레벨이 적절히 설정되어 있다 (DEBUG → INFO or WARNING)
[ ] 7. 로그 보존 기간이 정해져 있다
```

### 헬스체크 엔드포인트 구현

```python
# app/health.py
from fastapi import APIRouter
from datetime import datetime
import psutil

router = APIRouter()

@router.get("/health")
async def health_check():
    """서비스가 살아있는지 확인하는 엔드포인트"""
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat(),
        "version": "1.0.0"
    }

@router.get("/health/detail")
async def health_check_detail():
    """상세 상태 확인 — 내부 모니터링용"""
    return {
        "status": "healthy",
        "database": await check_database(),
        "cpu_percent": psutil.cpu_percent(),
        "memory_percent": psutil.virtual_memory().percent,
        "disk_percent": psutil.disk_usage('/').percent
    }

async def check_database():
    try:
        await db.execute("SELECT 1")
        return "connected"
    except Exception:
        return "disconnected"
```

### 로깅 설정

```python
# app/logging_config.py
import logging
import sys

def setup_logging():
    logging.basicConfig(
        level=logging.INFO,       # 프로덕션에서는 DEBUG 말고 INFO
        format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
        handlers=[
            logging.StreamHandler(sys.stdout),  # 콘솔 출력
        ]
    )
    
    # 특정 라이브러리 로그 레벨 조정
    logging.getLogger("sqlalchemy.engine").setLevel(logging.WARNING)
    logging.getLogger("uvicorn.access").setLevel(logging.INFO)

# 사용 예시
logger = logging.getLogger(__name__)

def process_order(order_id: int):
    logger.info(f"주문 처리 시작: order_id={order_id}")
    try:
        # 주문 처리 로직
        logger.info(f"주문 처리 완료: order_id={order_id}")
    except Exception as e:
        logger.error(f"주문 처리 실패: order_id={order_id}, error={e}", exc_info=True)
        raise
```

### 에러 알림 설정 (Slack 웹훅)

```python
# app/notifications.py
import os
import httpx
import logging

logger = logging.getLogger(__name__)

SLACK_WEBHOOK_URL = os.environ.get("SLACK_WEBHOOK_URL")

async def send_error_alert(error_message: str, context: dict = None):
    """에러 발생 시 슬랙으로 알림을 보낸다"""
    if not SLACK_WEBHOOK_URL:
        logger.warning("SLACK_WEBHOOK_URL이 설정되지 않아 알림을 보내지 못했습니다")
        return
    
    payload = {
        "text": f"🚨 *서비스 에러 발생*\n```{error_message}```",
        "attachments": [
            {
                "color": "danger",
                "fields": [
                    {"title": key, "value": str(value), "short": True}
                    for key, value in (context or {}).items()
                ]
            }
        ]
    }
    
    async with httpx.AsyncClient() as client:
        response = await client.post(SLACK_WEBHOOK_URL, json=payload)
        if response.status_code != 200:
            logger.error(f"슬랙 알림 전송 실패: {response.status_code}")
```

---

## 6. 배포 후 즉시 확인 체크리스트

배포가 완료된 직후, 5분 안에 이 항목들을 확인한다.

```
✅ 배포 후 즉시 확인 (5분 이내)

[ ] 1. 헬스체크 엔드포인트가 200을 반환한다
       curl https://yourdomain.com/health

[ ] 2. 주요 페이지(홈, 로그인)가 정상적으로 로드된다
       직접 브라우저에서 접속해본다

[ ] 3. 로그에 에러가 없다
       kubectl logs / docker logs / journalctl 확인

[ ] 4. 핵심 기능이 동작한다 (로그인, 데이터 조회 등)
       Smoke test: 가장 중요한 기능 1~2개를 직접 사용해본다

[ ] 5. 응답 시간이 정상 범위다
       첫 요청이 3초 이내에 응답하는지 확인
```

### 헬스체크 명령어

```bash
# 기본 헬스체크
curl -s https://yourdomain.com/health | python3 -m json.tool

# 예상 응답
# {
#     "status": "healthy",
#     "timestamp": "2026-05-22T09:00:00.000Z",
#     "version": "1.0.0"
# }

# 응답이 없거나 status가 "healthy"가 아니면 배포 실패
```

```bash
# 로그 확인 (Docker)
docker logs --tail 100 --follow my-app

# 로그 확인 (systemd)
journalctl -u my-app -n 100 -f

# 이런 로그가 보이면 정상
# 2026-05-22 09:00:05 [INFO] Application startup complete
# 2026-05-22 09:00:10 [INFO] GET /health 200 OK 12ms
```

---

## 따라 하기 실습

### 실습 1 — 배포 준비 상태 점검 스크립트 만들기

프로젝트 루트에 `scripts/pre-deploy-check.sh` 파일을 만든다.

```bash
mkdir -p scripts
```

```bash
#!/bin/bash
# scripts/pre-deploy-check.sh

set -e  # 오류 발생 시 즉시 중단

echo "===== 배포 전 점검 시작 ====="
PASS=0
FAIL=0

check() {
    local name="$1"
    local result="$2"
    if [ "$result" = "0" ]; then
        echo "✅ PASS: $name"
        PASS=$((PASS + 1))
    else
        echo "❌ FAIL: $name"
        FAIL=$((FAIL + 1))
    fi
}

# 1. 린터 확인
echo ""
echo "--- 코드 품질 ---"
flake8 app/ --max-line-length=100 --quiet
check "Flake8 린터" $?

# 2. 테스트 확인
echo ""
echo "--- 테스트 ---"
pytest --tb=short -q
check "전체 테스트 통과" $?

# 3. .env.example 존재 확인
echo ""
echo "--- 환경설정 ---"
[ -f ".env.example" ]
check ".env.example 파일 존재" $?

# 4. .env가 .gitignore에 있는지 확인
grep -q "^\.env$" .gitignore 2>/dev/null
check ".env가 .gitignore에 등록됨" $?

# 5. DEBUG=False 확인
grep -q "DEBUG=False" .env 2>/dev/null || grep -q 'DEBUG.*False' app/config.py 2>/dev/null
check "DEBUG=False 설정됨" $?

# 결과 요약
echo ""
echo "===== 점검 결과 ====="
echo "통과: $PASS개 / 실패: $FAIL개"

if [ $FAIL -gt 0 ]; then
    echo "❌ 배포 준비가 되지 않았습니다. 실패 항목을 수정하세요."
    exit 1
else
    echo "✅ 모든 점검 통과! 배포 준비 완료."
    exit 0
fi
```

```bash
# 스크립트 실행 권한 부여 후 실행
chmod +x scripts/pre-deploy-check.sh
./scripts/pre-deploy-check.sh
```

실행 결과 예시:

```
===== 배포 전 점검 시작 =====

--- 코드 품질 ---
✅ PASS: Flake8 린터

--- 테스트 ---
✅ PASS: 전체 테스트 통과

--- 환경설정 ---
✅ PASS: .env.example 파일 존재
✅ PASS: .env가 .gitignore에 등록됨
❌ FAIL: DEBUG=False 설정됨

===== 점검 결과 =====
통과: 4개 / 실패: 1개
❌ 배포 준비가 되지 않았습니다. 실패 항목을 수정하세요.
```

---

### 실습 2 — 환경변수 검증 코드 추가하기

앱이 시작될 때 필요한 환경변수가 모두 있는지 자동으로 확인하는 코드를 추가한다.

```python
# app/config.py
import os
from typing import List

# 반드시 있어야 하는 환경변수 목록
REQUIRED_ENV_VARS: List[str] = [
    "DATABASE_URL",
    "SECRET_KEY",
    "OPENAI_API_KEY",
]

# 있으면 좋은 환경변수 목록 (없어도 실행은 됨)
OPTIONAL_ENV_VARS: List[str] = [
    "SLACK_WEBHOOK_URL",
    "SMTP_HOST",
]

def validate_environment():
    """앱 시작 시 환경변수를 검증한다"""
    missing = []
    
    for var in REQUIRED_ENV_VARS:
        if not os.environ.get(var):
            missing.append(var)
    
    if missing:
        raise EnvironmentError(
            f"필수 환경변수가 설정되지 않았습니다: {', '.join(missing)}\n"
            f".env.example을 참고하여 .env 파일을 작성하세요."
        )
    
    for var in OPTIONAL_ENV_VARS:
        if not os.environ.get(var):
            print(f"⚠️  선택 환경변수 미설정: {var} (일부 기능이 동작하지 않을 수 있습니다)")

# 설정값 (환경변수에서 읽어온다)
DATABASE_URL: str = os.environ.get("DATABASE_URL", "")
SECRET_KEY: str = os.environ.get("SECRET_KEY", "")
DEBUG: bool = os.environ.get("DEBUG", "False").lower() == "true"
PORT: int = int(os.environ.get("PORT", "8000"))
```

```python
# app/main.py
from fastapi import FastAPI
from app.config import validate_environment
from app.health import router as health_router

# 앱 시작 시 가장 먼저 환경변수 검증
validate_environment()

app = FastAPI()
app.include_router(health_router)

@app.on_event("startup")
async def startup_event():
    print("✅ 서버 시작 완료")
```

```bash
# DATABASE_URL 없이 실행하면
python -m uvicorn app.main:app

# 출력
# EnvironmentError: 필수 환경변수가 설정되지 않았습니다: DATABASE_URL, SECRET_KEY
# .env.example을 참고하여 .env 파일을 작성하세요.
```

---

### 실습 3 — GitHub Actions로 배포 전 자동 점검 설정하기

Push할 때마다 자동으로 테스트와 린터를 실행하도록 GitHub Actions를 설정한다.

```bash
mkdir -p .github/workflows
```

```yaml
# .github/workflows/pre-deploy-check.yml
name: 배포 전 점검

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  quality-check:
    runs-on: ubuntu-latest
    
    steps:
      - name: 코드 가져오기
        uses: actions/checkout@v4
      
      - name: Python 설정
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: 패키지 설치
        run: pip install -r requirements.txt
      
      - name: 린터 실행
        run: flake8 app/ --max-line-length=100
      
      - name: 테스트 실행
        env:
          DATABASE_URL: sqlite:///./test.db
          SECRET_KEY: test-secret-key-for-ci
          DEBUG: "False"
        run: pytest --tb=short -v
      
      - name: 테스트 커버리지 확인
        env:
          DATABASE_URL: sqlite:///./test.db
          SECRET_KEY: test-secret-key-for-ci
        run: |
          pytest --cov=app --cov-fail-under=60
          echo "커버리지 60% 이상 통과"
```

Push 후 GitHub 저장소의 Actions 탭에서 결과를 확인할 수 있다. 빨간 X가 뜨면 배포하지 않는다.

---

## 자주 하는 실수

| 실수 | 발생하는 에러 / 결과 | 해결 방법 |
|------|---------------------|-----------|
| `.env` 파일을 GitHub에 올림 | API 키 노출, 요금 폭탄 | `.gitignore`에 `.env` 추가, 노출된 키 즉시 폐기 |
| `DEBUG=True`로 배포 | 에러 발생 시 코드와 환경변수가 외부에 노출 | `DEBUG=False` 설정 확인 |
| `requirements.txt` 없이 배포 | `ModuleNotFoundError: No module named 'fastapi'` | `pip freeze > requirements.txt` 후 커밋 |
| 환경변수 이름 오타 | `KeyError: 'DATBASE_URL'` (오타) 또는 `None` 반환 | `.env.example`과 코드의 변수 이름 일치 확인 |
| 테스트 없이 배포 | 회원가입, 결제 등 핵심 기능 오류를 배포 후 발견 | 배포 전 `pytest` 실행 의무화 |
| 헬스체크 엔드포인트 없음 | 서버 다운 여부를 알 수 없음 | `GET /health` 엔드포인트 추가 |
| 로그 없이 배포 | 에러가 나도 원인을 알 수 없음 | `logging` 설정 후 배포 |
| 포트 충돌 | `OSError: [Errno 98] Address already in use` | `lsof -i :8000`으로 사용 중인 프로세스 확인 후 종료 |
| 데이터베이스 마이그레이션 누락 | `sqlalchemy.exc.OperationalError: no such column` | 배포 전 `alembic upgrade head` 실행 |
| 시간대(timezone) 설정 누락 | 로그 시간이 UTC로 기록되어 디버깅 어려움 | `TZ=Asia/Seoul` 환경변수 설정 |

---

## 확인 체크리스트

이 장을 마친 후 스스로 확인해보자.

```
[ ] 1. 코드 품질 체크리스트 5개 항목을 말할 수 있다
[ ] 2. 린터(flake8 또는 eslint)를 실행하고 오류를 수정할 수 있다
[ ] 3. pytest를 실행하고 모든 테스트가 통과하는 것을 확인했다
[ ] 4. .env 파일이 .gitignore에 등록되어 있다
[ ] 5. .env.example 파일을 만들어 필요한 환경변수를 문서화했다
[ ] 6. DEBUG=False가 프로덕션 설정에 되어 있다
[ ] 7. GET /health 엔드포인트를 구현했다
[ ] 8. 로깅이 설정되어 INFO 레벨로 로그가 출력된다
[ ] 9. pre-deploy-check.sh 스크립트를 실행해서 모든 항목이 통과했다
[ ] 10. 배포 후 즉시 확인할 5가지 항목을 말할 수 있다
```

---

## 한 번 더 생각해 보기

**1.** 체크리스트를 항상 모두 통과해야만 배포할 수 있을까? 급한 버그 수정이 필요할 때는 어떻게 해야 할까?

> 힌트: 항목마다 "필수"와 "권장"을 구분할 수 있다. 긴급 배포(hotfix)에는 최소 필수 항목만 통과하면 배포하고, 나머지는 배포 후 빠르게 보완하는 방식을 쓴다. 중요한 것은 어떤 항목을 건너뛰었는지 기록해두는 것이다.

**2.** 혼자 개발하는 프로젝트라도 체크리스트가 필요할까?

> 힌트: 혼자라도 "2주 후의 나"는 지금의 나와 다르다. 배포 체크리스트는 팀을 위한 것이 아니라 **미래의 나를 위한 것**이기도 하다. 잊어버리는 것을 막는 도구다.

**3.** 배포 후 헬스체크가 성공했는데도 사용자가 버그를 신고한다. 어떻게 대응해야 할까?

> 힌트: 헬스체크는 "서버가 켜져 있다"는 것만 확인한다. 실제 비즈니스 로직이 정상인지는 확인하지 않는다. 이 때문에 핵심 기능을 직접 테스트하는 Smoke Test와 로그 모니터링이 함께 필요하다.

---

## 다음 장

다음 장에서는 배포한 서비스의 성능을 측정하고 병목 지점을 찾아 최적화하는 방법을 배운다.