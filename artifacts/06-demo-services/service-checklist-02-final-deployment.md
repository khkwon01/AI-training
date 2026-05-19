## 이 장에서 배우는 것

- AI 클라우드 서비스를 배포하기 전에 반드시 확인해야 할 항목들
- 체크리스트를 직접 만들고 활용하는 방법
- 실제 배포 실패 사례와 예방법
- GitHub Actions, AWS, 환경 변수 설정의 최종 점검 방법
- 팀원과 함께 체크리스트를 공유하고 관리하는 방법

---

## 먼저 쉬운 설명

비행기 조종사는 이륙 전에 항상 체크리스트를 확인합니다. 엔진 상태, 연료, 날씨 — 하나라도 빠뜨리면 큰일이 납니다.

AI 서비스 배포도 똑같습니다. 코드를 열심히 만들었는데 막상 서버에 올리면 "왜 안 되지?" 하는 상황이 자주 발생합니다. API 키를 빠뜨렸거나, 의존성 패키지를 설치 안 했거나, 환경 변수 이름이 틀렸거나.

이 장에서는 배포 전에 반드시 확인해야 할 항목들을 체크리스트 형태로 정리하고, 직접 실습해 봅니다. 체크리스트 하나가 몇 시간의 디버깅을 막아 줍니다.

---

## 1. 배포 체크리스트란 무엇인가

체크리스트는 "잊으면 안 되는 것들"을 미리 목록으로 만들어 두는 도구입니다. 프로젝트 루트에 `DEPLOY_CHECKLIST.md` 파일을 만들어 팀과 공유하는 것이 좋습니다.

```markdown
# 배포 체크리스트 — AI 날씨 예측 서비스

## 코드 검토
- [ ] main 브랜치에 최신 코드가 병합되었는가
- [ ] 테스트가 모두 통과했는가 (`pytest` 결과 확인)
- [ ] 민감한 정보(API 키, 비밀번호)가 코드에 직접 적혀 있지 않은가

## 환경 설정
- [ ] `.env.example` 파일이 최신 상태인가
- [ ] AWS Secrets Manager 또는 환경 변수에 실제 값이 설정되었는가
- [ ] `requirements.txt`가 현재 가상환경과 일치하는가

## 인프라
- [ ] AWS IAM 역할 권한이 올바른가
- [ ] S3 버킷 이름과 리전이 코드와 일치하는가
- [ ] Lambda 메모리/타임아웃 설정이 적절한가

## 배포 후 확인
- [ ] 헬스체크 엔드포인트가 200을 반환하는가
- [ ] CloudWatch 로그에 에러가 없는가
- [ ] 실제 AI 요청을 한 번 테스트했는가
```

---

## 2. 환경 변수 최종 점검

배포 실패의 가장 흔한 원인은 환경 변수 누락입니다. 로컬에서는 `.env` 파일에 있어서 잘 되는데, 서버에는 없어서 터지는 경우가 많습니다.

```python
# check_env.py — 배포 전에 실행하는 환경 변수 점검 스크립트

import os
import sys

# 반드시 있어야 하는 환경 변수 목록
REQUIRED_ENV_VARS = [
    "OPENAI_API_KEY",
    "AWS_REGION",
    "S3_BUCKET_NAME",
    "DATABASE_URL",
    "APP_SECRET_KEY",
]

def check_env_vars():
    missing = []
    for var in REQUIRED_ENV_VARS:
        value = os.environ.get(var)
        if not value:
            missing.append(var)
        else:
            # 값이 있어도 플레이스홀더인지 확인
            if value in ("your-key-here", "CHANGE_ME", "TODO"):
                missing.append(f"{var} (플레이스홀더 값이 그대로임)")

    if missing:
        print("❌ 아래 환경 변수가 설정되지 않았습니다:")
        for var in missing:
            print(f"   - {var}")
        sys.exit(1)  # 배포 파이프라인을 여기서 중단시킴
    else:
        print("✅ 모든 환경 변수가 설정되었습니다.")

if __name__ == "__main__":
    check_env_vars()
```

```bash
# 실행 방법
python check_env.py

# 성공 시 출력
✅ 모든 환경 변수가 설정되었습니다.

# 실패 시 출력
❌ 아래 환경 변수가 설정되지 않았습니다:
   - OPENAI_API_KEY
   - DATABASE_URL
```

---

## 3. 의존성 패키지 점검

로컬에서 pip install로 추가한 패키지를 `requirements.txt`에 업데이트하는 것을 깜빡하는 경우가 많습니다.

```bash
# requirements.txt 자동 생성 (현재 가상환경 기준)
pip freeze > requirements.txt

# 생성된 파일 예시 (requirements.txt)
# anthropic==0.28.0
# boto3==1.34.0
# fastapi==0.111.0
# uvicorn==0.30.0
# pydantic==2.7.0
# python-dotenv==1.0.1
```

```python
# verify_requirements.py — requirements.txt와 실제 설치된 패키지가 일치하는지 확인

import subprocess
import sys

def verify_requirements(requirements_file="requirements.txt"):
    print(f"📦 {requirements_file} 검증 중...")

    # pip check: 의존성 충돌 확인
    result = subprocess.run(
        ["pip", "check"],
        capture_output=True,
        text=True
    )

    if result.returncode != 0:
        print("❌ 의존성 충돌이 발견되었습니다:")
        print(result.stdout)
        sys.exit(1)

    # requirements.txt에 있는 패키지가 모두 설치되었는지 확인
    with open(requirements_file) as f:
        packages = [
            line.strip()
            for line in f
            if line.strip() and not line.startswith("#")
        ]

    missing_packages = []
    for package in packages:
        name = package.split("==")[0].split(">=")[0]
        check = subprocess.run(
            ["pip", "show", name],
            capture_output=True,
            text=True
        )
        if check.returncode != 0:
            missing_packages.append(package)

    if missing_packages:
        print("❌ 설치되지 않은 패키지:")
        for pkg in missing_packages:
            print(f"   - {pkg}")
        print("\n다음 명령으로 설치하세요: pip install -r requirements.txt")
        sys.exit(1)

    print("✅ 모든 패키지가 올바르게 설치되었습니다.")

if __name__ == "__main__":
    verify_requirements()
```

---

## 4. GitHub Actions CI 최종 점검

배포 파이프라인이 실제로 잘 동작하는지 확인하는 것도 체크리스트의 중요한 부분입니다.

```yaml
# .github/workflows/deploy-checklist.yml
# 배포 전 자동 체크리스트를 실행하는 워크플로우

name: 배포 전 자동 체크리스트

on:
  pull_request:
    branches: [main]
  workflow_dispatch:  # 수동 실행 가능

jobs:
  pre-deploy-check:
    name: 배포 준비 점검
    runs-on: ubuntu-latest

    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: Python 설치
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: 패키지 설치
        run: pip install -r requirements.txt

      - name: 환경 변수 점검
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          AWS_REGION: ${{ secrets.AWS_REGION }}
          S3_BUCKET_NAME: ${{ secrets.S3_BUCKET_NAME }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          APP_SECRET_KEY: ${{ secrets.APP_SECRET_KEY }}
        run: python check_env.py

      - name: 의존성 검증
        run: python verify_requirements.py

      - name: 테스트 실행
        run: pytest tests/ -v --tb=short

      - name: 보안 취약점 스캔
        run: |
          pip install bandit
          bandit -r . -x ./tests --severity-level medium

      - name: 모든 점검 통과
        run: echo "✅ 배포 준비 완료! 모든 체크리스트를 통과했습니다."
```

---

## 5. 배포 후 헬스체크 자동화

배포가 끝났다고 끝이 아닙니다. 서비스가 실제로 살아있는지 확인해야 합니다.

```python
# post_deploy_check.py — 배포 후 서비스 상태 확인

import time
import sys
import requests  # pip install requests 필요

# 배포된 서비스 URL (실제 URL로 교체하세요)
SERVICE_URL = "https://your-service.amazonaws.com"

def health_check(url: str, max_retries: int = 5, wait_seconds: int = 10) -> bool:
    """
    서비스가 정상 응답할 때까지 재시도합니다.
    배포 직후에는 서비스가 뜨는 데 시간이 걸릴 수 있습니다.
    """
    print(f"🔍 헬스체크 시작: {url}/health")

    for attempt in range(1, max_retries + 1):
        try:
            response = requests.get(f"{url}/health", timeout=5)

            if response.status_code == 200:
                data = response.json()
                print(f"✅ 서비스 정상 작동 중!")
                print(f"   상태: {data.get('status', 'ok')}")
                print(f"   버전: {data.get('version', 'unknown')}")
                return True
            else:
                print(f"⚠️  시도 {attempt}/{max_retries}: HTTP {response.status_code}")

        except requests.exceptions.ConnectionError:
            print(f"⚠️  시도 {attempt}/{max_retries}: 연결 실패 (서비스 시작 중일 수 있음)")

        except requests.exceptions.Timeout:
            print(f"⚠️  시도 {attempt}/{max_retries}: 타임아웃")

        if attempt < max_retries:
            print(f"   {wait_seconds}초 후 재시도...")
            time.sleep(wait_seconds)

    print("❌ 헬스체크 실패: 서비스가 응답하지 않습니다.")
    return False


def smoke_test(url: str) -> bool:
    """실제 AI 기능이 동작하는지 간단히 테스트합니다."""
    print("\n🔥 스모크 테스트 시작...")

    try:
        response = requests.post(
            f"{url}/api/predict",
            json={"input": "테스트 입력"},
            timeout=30
        )

        if response.status_code == 200:
            print("✅ AI 예측 엔드포인트 정상 작동")
            return True
        else:
            print(f"❌ 스모크 테스트 실패: HTTP {response.status_code}")
            print(f"   응답: {response.text[:200]}")
            return False

    except Exception as e:
        print(f"❌ 스모크 테스트 오류: {e}")
        return False


if __name__ == "__main__":
    if not health_check(SERVICE_URL):
        sys.exit(1)

    if not smoke_test(SERVICE_URL):
        sys.exit(1)

    print("\n🎉 배포 완료! 서비스가 정상적으로 운영 중입니다.")
```

---

## 따라 하기 실습

### 실습 1 — 나만의 체크리스트 파일 만들기

프로젝트 루트에 `DEPLOY_CHECKLIST.md`를 만들고, 본인 프로젝트에 맞게 항목을 채워 봅니다.

```bash
# 1. 프로젝트 폴더로 이동
cd ~/my-ai-project

# 2. 체크리스트 파일 생성
touch DEPLOY_CHECKLIST.md

# 3. 아래 내용을 붙여넣고 프로젝트에 맞게 수정
cat > DEPLOY_CHECKLIST.md << 'EOF'
# 배포 체크리스트

## 코드
- [ ] 모든 테스트 통과 (`pytest` 확인)
- [ ] 코드 리뷰 완료
- [ ] 불필요한 print() 제거

## 환경
- [ ] .env.example 업데이트
- [ ] GitHub Secrets에 모든 키 등록
- [ ] requirements.txt 최신화

## 배포 후
- [ ] 헬스체크 통과
- [ ] 스모크 테스트 통과
- [ ] CloudWatch 로그 확인
EOF

# 4. Git에 추가
git add DEPLOY_CHECKLIST.md
git commit -m "docs: 배포 체크리스트 추가"
```

---

### 실습 2 — 환경 변수 점검 스크립트 실행하기

위에서 만든 `check_env.py`를 프로젝트에 추가하고 실행해 봅니다.

```bash
# 1. 스크립트 저장 (위의 check_env.py 내용 붙여넣기)
touch check_env.py
# (내용 편집 후 저장)

# 2. 일부러 환경 변수 없이 실행해서 오류 확인
python check_env.py
# 예상 출력:
# ❌ 아래 환경 변수가 설정되지 않았습니다:
#    - OPENAI_API_KEY
#    - AWS_REGION
#    ...

# 3. 환경 변수를 설정하고 다시 실행
export OPENAI_API_KEY="sk-테스트키"
export AWS_REGION="ap-northeast-2"
export S3_BUCKET_NAME="my-ai-bucket"
export DATABASE_URL="postgresql://localhost/mydb"
export APP_SECRET_KEY="super-secret-key-1234"

python check_env.py
# 예상 출력:
# ✅ 모든 환경 변수가 설정되었습니다.
```

---

### 실습 3 — GitHub Actions에 체크리스트 워크플로우 연결하기

위에서 만든 파일들을 GitHub Actions와 연결합니다.

```bash
# 1. 워크플로우 디렉토리 생성
mkdir -p .github/workflows

# 2. 워크플로우 파일 생성 (위의 deploy-checklist.yml 내용 붙여넣기)
touch .github/workflows/deploy-checklist.yml
# (내용 편집 후 저장)

# 3. GitHub Secrets 등록 (GitHub 웹사이트에서)
# Settings → Secrets and variables → Actions → New repository secret
# 이름: OPENAI_API_KEY
# 값: 실제 API 키

# 4. 변경사항 커밋 및 Push
git add .github/ check_env.py verify_requirements.py
git commit -m "ci: 배포 전 자동 체크리스트 워크플로우 추가"
git push origin main

# 5. GitHub → Actions 탭에서 워크플로우 실행 확인
# 모든 단계가 초록색 체크 표시면 성공!
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 해결 방법 |
|------|------------|-----------|
| `.env` 파일을 GitHub에 올림 | 경고 없음 (보안 사고 발생) | `.gitignore`에 `.env` 추가, `git rm --cached .env` 실행 |
| `requirements.txt` 업데이트 안 함 | `ModuleNotFoundError: No module named 'anthropic'` | `pip freeze > requirements.txt` 후 커밋 |
| GitHub Secrets 이름 오타 | `KeyError: 'OPENAI_API_KEY'` 또는 빈 문자열 | Secrets 이름과 코드의 `os.environ.get()` 이름 일치 확인 |
| 테스트 없이 배포 | 런타임에 예상치 못한 500 오류 | `pytest` 통과 후에만 배포하도록 CI에 gate 추가 |
| 헬스체크 엔드포인트 미구현 | 배포 성공인데 서비스 사망 상태 | `/health` 엔드포인트를 반드시 구현 |
| 로컬과 다른 Python 버전 | `SyntaxError` 또는 패키지 설치 실패 | `python --version` 확인, `pyproject.toml`에 버전 명시 |
| AWS 리전 불일치 | `NoSuchBucket` 또는 `EndpointResolutionError` | 코드, 환경 변수, AWS 콘솔 리전이 모두 동일한지 확인 |

---

## 확인 체크리스트

- [ ] `DEPLOY_CHECKLIST.md` 파일을 프로젝트에 만들었다
- [ ] `check_env.py`를 실행해서 모든 환경 변수가 설정되었음을 확인했다
- [ ] `requirements.txt`가 최신 상태임을 `pip freeze`로 확인했다
- [ ] `.gitignore`에 `.env`가 포함되어 있다
- [ ] GitHub Secrets에 필요한 키가 모두 등록되어 있다
- [ ] `pytest`를 실행해서 테스트가 모두 통과함을 확인했다
- [ ] GitHub Actions 워크플로우가 성공(초록 체크)으로 완료되었다
- [ ] 배포 후 헬스체크 URL에서 200 응답을 받았다
- [ ] CloudWatch 또는 로그에서 에러가 없음을 확인했다

---

## 한 번 더 생각해 보기

1. **체크리스트가 없다면 어떤 일이 벌어질까요?** 경험했던 (또는 상상해 볼 수 있는) 배포 실패 상황을 하나 떠올리고, 어떤 체크리스트 항목이 그것을 막을 수 있었을지 적어 보세요.

2. **자동화와 수동 점검은 어떻게 균형을 맞춰야 할까요?** GitHub Actions로 자동화할 수 있는 것과, 사람이 직접 눈으로 확인해야 하는 것을 구분해 본다면 각각 어떤 항목이 들어가야 할까요?

3. **서비스가 배포되었는데 헬스체크는 통과했지만 실제 AI 응답이 이상하다면?** 이런 상황을 사전에 잡아낼 수 있는 스모크 테스트를 어떻게 설계할 수 있을까요?

---

## 다음 장

다음 장에서는 실제로 배포된 AI 서비스의 운영 중 모니터링과 장애 대응 방법을 배웁니다.