## 이 장에서 배우는 것

- 가상 환경(virtual environment)이 무엇인지, 왜 팀 프로젝트에서 반드시 필요한지 이해한다
- `venv`로 가상 환경을 만들고 활성화·비활성화하는 방법을 익힌다
- `requirements.txt`와 `pyproject.toml`로 팀원과 의존성을 공유하는 방법을 배운다
- `.gitignore`에 가상 환경 폴더를 올바르게 등록하는 방법을 안다
- 팀 협업 시 자주 발생하는 "내 컴퓨터에서만 된다" 문제를 예방하는 습관을 기른다

---

## 먼저 쉬운 설명

여러분이 요리를 한다고 상상해 보세요. A 요리사는 소금을 "조금" 넣고, B 요리사는 소금을 "한 스푼" 넣습니다. 같은 레시피를 봐도 결과물이 달라지죠.

파이썬 패키지도 똑같습니다. 팀원 A는 `requests` 버전 2.28을 쓰고, 팀원 B는 버전 2.31을 쓰면 코드가 서로 다르게 동작할 수 있습니다. 배포 서버에서는 아예 실행이 안 될 수도 있고요.

**가상 환경**은 프로젝트마다 "독립된 파이썬 부엌"을 만들어 주는 도구입니다. 각 부엌에는 그 프로젝트에 필요한 재료(패키지)만 정확한 양(버전)으로 담겨 있습니다. 덕분에 "내 컴퓨터에서는 됐는데 왜 서버에서 안 되죠?"라는 말을 팀에서 영원히 없앨 수 있습니다.

---

## 1. 가상 환경 만들기와 활성화

파이썬 3.3 이상이면 별도 설치 없이 `venv` 모듈을 바로 쓸 수 있습니다.

```bash
# 프로젝트 폴더로 이동
cd ~/projects/my-team-app

# 가상 환경 생성 (.venv 라는 이름을 권장)
python -m venv .venv
```

생성 후에는 반드시 **활성화**해야 합니다. 활성화 방법은 운영체제마다 다릅니다.

```bash
# macOS / Linux
source .venv/bin/activate

# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# Windows (명령 프롬프트)
.venv\Scripts\activate.bat
```

활성화되면 터미널 앞에 `(.venv)` 표시가 나타납니다.

```
(.venv) kihyuk@mac my-team-app %
```

작업이 끝나면 아래 명령으로 비활성화합니다.

```bash
deactivate
```

> **팀 규칙 추천:** 가상 환경 폴더 이름을 `.venv`로 통일하면 팀원 모두가 동일한 명령어를 사용할 수 있습니다.

---

## 2. `.gitignore`에 가상 환경 등록하기

가상 환경 폴더(`.venv`)는 절대 Git에 올리면 안 됩니다. 수백 MB에 달하는 파일이 올라가고, 운영체제마다 경로가 달라서 충돌이 생깁니다.

프로젝트 루트에 `.gitignore` 파일을 만들거나 열어서 아래 내용을 추가하세요.

```gitignore
# 가상 환경
.venv/
venv/
env/

# 파이썬 캐시
__pycache__/
*.pyc
*.pyo

# 환경 변수 파일 (비밀 키 포함 가능)
.env
.env.local
```

```bash
# .gitignore가 제대로 적용되는지 확인
git status
```

`.venv/` 폴더가 목록에 나타나지 않으면 정상입니다.

---

## 3. `requirements.txt`로 의존성 공유하기

팀원과 패키지 목록을 공유하는 가장 간단한 방법은 `requirements.txt`입니다.

```bash
# 가상 환경이 활성화된 상태에서 패키지 설치
pip install requests==2.31.0
pip install fastapi==0.111.0
pip install pytest==8.2.0

# 현재 설치된 패키지를 파일로 저장
pip freeze > requirements.txt
```

생성된 `requirements.txt` 예시:

```text
annotated-types==0.7.0
anyio==4.4.0
fastapi==0.111.0
pytest==8.2.0
requests==2.31.0
starlette==0.37.2
```

팀원이 처음 프로젝트를 받았을 때 실행할 명령:

```bash
# 저장소 클론 후
git clone https://github.com/my-team/my-team-app.git
cd my-team-app

# 가상 환경 생성 및 활성화
python -m venv .venv
source .venv/bin/activate   # macOS/Linux

# 패키지 한 번에 설치
pip install -r requirements.txt
```

---

## 4. `pyproject.toml`로 더 체계적으로 관리하기 (권장)

요즘 팀 프로젝트에서는 `requirements.txt` 대신 `pyproject.toml`을 많이 씁니다. 프로젝트 메타정보(이름, 버전, 작성자)와 의존성을 한 파일에 정리할 수 있기 때문입니다.

```toml
# pyproject.toml

[project]
name = "my-team-app"
version = "0.1.0"
description = "팀 협업 예제 프로젝트"
requires-python = ">=3.11"

dependencies = [
    "fastapi>=0.111.0",
    "requests>=2.31.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.2.0",
    "ruff>=0.4.0",
]
```

설치 방법:

```bash
# 운영 의존성만 설치
pip install .

# 개발 도구(pytest, ruff 등)까지 설치
pip install ".[dev]"
```

> **팁:** `pyproject.toml`을 쓰면 버전 범위(`>=`)로 지정할 수 있어서 보안 패치가 나왔을 때 유연하게 대응할 수 있습니다.

---

## 5. 팀 온보딩 스크립트 만들기

새 팀원이 합류할 때마다 "이거 실행하면 끝"이 되도록 셋업 스크립트를 만들어 두면 편합니다.

```bash
#!/bin/bash
# scripts/setup.sh

set -e  # 오류 발생 시 즉시 중단

echo "=== 개발 환경 설정 시작 ==="

# 파이썬 버전 확인
python_version=$(python3 --version 2>&1)
echo "파이썬 버전: $python_version"

# 가상 환경 생성
if [ ! -d ".venv" ]; then
    echo ".venv 가상 환경 생성 중..."
    python3 -m venv .venv
fi

# 활성화 및 패키지 설치
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt

echo "=== 설정 완료! ==="
echo "다음 명령으로 가상 환경을 활성화하세요:"
echo "  source .venv/bin/activate"
```

```bash
# 스크립트 실행 권한 부여 후 실행
chmod +x scripts/setup.sh
./scripts/setup.sh
```

---

## 따라 하기 실습

### 실습 1 — 팀 프로젝트 기본 구조 만들기

아래 명령을 순서대로 따라 하면서 실제 팀 프로젝트 구조를 만들어 보세요.

```bash
# 1. 프로젝트 폴더 생성
mkdir team-todo-app
cd team-todo-app

# 2. 가상 환경 생성 및 활성화
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate.bat

# 3. 필요한 패키지 설치
pip install requests pytest

# 4. requirements.txt 생성
pip freeze > requirements.txt

# 5. 생성된 내용 확인
cat requirements.txt
```

**확인 포인트:** `requirements.txt` 안에 `requests`와 `pytest`가 버전 번호와 함께 있으면 성공입니다.

---

### 실습 2 — `.gitignore` 설정 및 Git 등록

실습 1에 이어서 진행합니다.

```bash
# 1. .gitignore 파일 생성
cat > .gitignore << 'EOF'
.venv/
__pycache__/
*.pyc
.env
EOF

# 2. Git 초기화 및 파일 등록
git init
git add .gitignore requirements.txt
git status
```

`git status` 결과에서 `.venv/` 폴더가 **보이지 않으면** 정상입니다. 만약 보인다면 `.gitignore` 파일이 저장됐는지 다시 확인하세요.

```bash
# 3. 첫 커밋
git commit -m "chore: 프로젝트 초기 설정 및 가상 환경 구성"
```

---

### 실습 3 — 팀원 합류 시나리오 시뮬레이션

실습 2에 이어서, 새 팀원이 합류하는 상황을 시뮬레이션합니다.

```bash
# 1. 현재 가상 환경 비활성화
deactivate

# 2. 다른 팀원 역할: 프로젝트 폴더를 새로 받은 것처럼 .venv 삭제
rm -rf .venv

# 3. 새 가상 환경 생성 및 패키지 복원
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# 4. 설치 확인
pip list
```

**확인 포인트:** `pip list`에 `requests`와 `pytest`가 나타나면 팀원 온보딩 시나리오 성공입니다.

---

## 자주 하는 실수

| 실수 | 실제 에러 메시지 | 해결 방법 |
|------|-----------------|-----------|
| 가상 환경을 활성화하지 않고 패키지 설치 | `WARNING: pip is configured with locations that require TLS/SSL...` 또는 전역 환경에 설치됨 | `source .venv/bin/activate` 먼저 실행. 터미널에 `(.venv)` 표시 확인 |
| `requirements.txt` 없이 커밋 | 팀원 컴퓨터에서 `ModuleNotFoundError: No module named 'requests'` | `pip freeze > requirements.txt` 후 커밋에 포함 |
| `.venv` 폴더를 Git에 올림 | `git push` 매우 느려지거나 수백 MB 파일 업로드 | `.gitignore`에 `.venv/` 추가. 이미 올렸다면 `git rm -r --cached .venv` |
| `requirements.txt` 업데이트 잊기 | 새 팀원이 `pip install -r requirements.txt` 해도 최신 패키지 없음 | 패키지 추가할 때마다 `pip freeze > requirements.txt` 실행 후 커밋 |
| Windows/Mac 경로 혼용 | `.venv\Scripts\activate` 를 Mac에서 실행 → `No such file or directory` | OS에 맞는 활성화 명령 사용. `setup.sh`에 OS 분기 처리 추가 |
| 파이썬 버전 불일치 | `SyntaxError` 또는 `requires Python >=3.11` | `python --version` 으로 확인. `pyproject.toml`에 `requires-python` 명시 |

---

## 확인 체크리스트

프로젝트를 팀에 공유하기 전에 아래 항목을 모두 체크하세요.

- [ ] `.venv/` 폴더가 `.gitignore`에 등록되어 있다
- [ ] `git status`에서 `.venv/` 폴더가 보이지 않는다
- [ ] `requirements.txt` 또는 `pyproject.toml`이 최신 상태로 커밋되어 있다
- [ ] 가상 환경을 삭제하고 `pip install -r requirements.txt`로 복원했을 때 프로젝트가 정상 실행된다
- [ ] README 또는 온보딩 문서에 가상 환경 설치 방법이 적혀 있다
- [ ] 팀원이 다른 OS(Windows/Mac)를 쓰는 경우 각각의 활성화 명령이 문서에 있다
- [ ] `.env` 파일(API 키, 비밀번호 등)도 `.gitignore`에 포함되어 있다

---

## 한 번 더 생각해 보기

1. **"이 파일은 올려야 할까, 말아야 할까?"** — `requirements.txt`는 Git에 올리고 `.venv/`는 올리지 않는 이유를 자신의 말로 설명해 보세요. 어떤 기준으로 나누면 좋을까요?

2. **버전을 고정하는 것의 장단점은?** — `requests==2.31.0`처럼 정확한 버전을 고정하면 안정성이 높아지지만, 보안 패치가 나왔을 때 수동으로 업데이트해야 합니다. 팀에서 어떤 방식을 선택할지 이유와 함께 생각해 보세요.

3. **신입 팀원의 입장에서 생각해 보기** — 오늘 처음 팀에 합류한 동료가 README만 보고 30분 안에 개발 환경을 세팅할 수 있을까요? 지금 여러분 프로젝트의 온보딩 문서를 직접 따라해 보고, 막히는 곳이 있다면 어떻게 개선할 수 있을지 적어 보세요.

---

## 다음 장

다음 장에서는 `pytest`를 활용해 팀 프로젝트에서 자동화된 테스트를 작성하고 CI 파이프라인에 연결하는 방법을 배웁니다.