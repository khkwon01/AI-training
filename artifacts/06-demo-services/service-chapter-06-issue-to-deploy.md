## 이 장에서 배우는 것

- GitHub Issue를 작성하는 방법과 그 목적
- AI(Claude)를 활용해 Issue → 코드 작성 → PR → 배포까지 이어지는 전체 흐름
- GitHub Actions를 사용한 자동 배포(CD) 파이프라인 구성
- AI와 협업해서 실제 작동하는 기능을 처음부터 끝까지 완성하는 경험

---

## 먼저 쉬운 설명

여러분이 앱을 만들 때 이런 경험 해본 적 있나요?

> "이 기능 추가해야 하는데… 어디서부터 시작하지?"

실무에서 개발자들은 **Issue → 코드 → PR → 배포** 순서로 일합니다. 마치 요리처럼 — 메뉴를 정하고(Issue), 요리하고(코드), 시식하고(PR 리뷰), 손님에게 내놓는(배포) 과정이죠.

이 장에서는 AI를 요리 보조 셰프처럼 활용해서, 이 전체 흐름을 **혼자서도** 처음부터 끝까지 완성하는 방법을 배웁니다.

---

## 1. GitHub Issue 작성하기 — "무엇을 만들 것인가" 명확히 하기

Issue는 **할 일 메모**입니다. 하지만 좋은 Issue는 단순한 메모가 아니라, 팀원(또는 AI)이 바로 작업을 시작할 수 있을 만큼 구체적입니다.

### 나쁜 Issue 예시

```
제목: 버그 수정
내용: 뭔가 안 됨
```

### 좋은 Issue 예시

```markdown
## 제목: 사용자 이름 표시 기능 추가

### 배경
현재 로그인 후 화면에 "안녕하세요" 만 표시되고 사용자 이름이 없음

### 해야 할 일
- [ ] `/api/user` 엔드포인트에서 사용자 이름 반환
- [ ] 메인 화면에 "안녕하세요, {이름}님!" 형태로 표시
- [ ] 이름이 없을 경우 "사용자"로 fallback

### 완료 기준
로그인 후 화면에 실제 이름이 표시되면 완료
```

### AI에게 Issue 초안 요청하기

```
나: "사용자 이름을 화면에 표시하는 기능을 만들려고 해.
     GitHub Issue 형식으로 작성해줘. 백엔드는 FastAPI, 
     프론트엔드는 React야."

AI: [위와 같은 구조화된 Issue 초안 생성]
```

---

## 2. AI와 함께 코드 작성하기 — Issue를 코드로 변환

Issue가 준비됐으면 AI에게 코드 작성을 요청합니다. 핵심은 **맥락을 충분히 제공**하는 것입니다.

### 백엔드 코드 (FastAPI — `app/routers/user.py`)

```python
# app/routers/user.py
from fastapi import APIRouter, Depends
from app.auth import get_current_user
from app.models import User

router = APIRouter()

@router.get("/api/user")
async def get_user_info(current_user: User = Depends(get_current_user)):
    return {
        "name": current_user.name or "사용자",
        "email": current_user.email
    }
```

### 프론트엔드 코드 (React — `src/components/Welcome.jsx`)

```jsx
// src/components/Welcome.jsx
import { useEffect, useState } from "react";

export default function Welcome() {
  const [userName, setUserName] = useState("사용자");

  useEffect(() => {
    fetch("/api/user")
      .then((res) => res.json())
      .then((data) => setUserName(data.name))
      .catch(() => setUserName("사용자")); // 에러 시 기본값
  }, []);

  return <h1>안녕하세요, {userName}님!</h1>;
}
```

### AI에게 코드 요청하는 좋은 방법

```
나: "위에서 만든 Issue를 기반으로 코드를 작성해줘.
     - 파일 이름도 알려줘
     - 각 줄이 무슨 역할인지 한국어로 주석 달아줘
     - 초보자가 이해할 수 있게 단순하게 작성해줘"
```

---

## 3. Pull Request 만들기 — 코드를 팀에게 보여주기

코드를 작성했으면 **PR(Pull Request)**를 만들어 검토를 요청합니다. PR은 "내가 이런 코드를 썼는데, 맞게 했나요?" 라고 묻는 과정입니다.

### Git 명령어 흐름

```bash
# 1. 새 브랜치 만들기 (Issue 번호 포함하면 좋음)
git checkout -b feature/issue-12-show-username

# 2. 변경 파일 확인
git status

# 3. 변경 내용 스테이징
git add app/routers/user.py src/components/Welcome.jsx

# 4. 커밋 메시지 작성 (Issue 번호 참조)
git commit -m "feat: 사용자 이름 표시 기능 추가 (#12)"

# 5. 원격 저장소에 push
git push origin feature/issue-12-show-username
```

### AI에게 PR 설명 작성 요청하기

```
나: "방금 작성한 코드에 대한 PR 설명을 작성해줘.
     - 무엇을 변경했는지
     - 어떻게 테스트했는지
     - 관련 Issue 번호는 #12야"
```

### PR 템플릿 예시 (`.github/pull_request_template.md`)

```markdown
## 변경 내용
- 사용자 이름 API 엔드포인트 추가 (`/api/user`)
- 메인 화면에 이름 표시 컴포넌트 추가

## 관련 Issue
Closes #12

## 테스트 방법
1. `uvicorn app.main:app --reload` 실행
2. 로그인 후 메인 화면 확인
3. "안녕하세요, [이름]님!" 메시지 확인
```

---

## 4. GitHub Actions로 자동 배포하기 — 코드가 자동으로 서버에 올라가게 하기

PR이 승인되고 `main` 브랜치에 merge되면, 코드가 **자동으로 서버에 배포**되도록 설정합니다.

### 배포 워크플로 파일 (`.github/workflows/deploy.yml`)

```yaml
# .github/workflows/deploy.yml
name: 자동 배포

# main 브랜치에 push될 때 실행
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # 1단계: 코드 가져오기
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      # 2단계: Python 환경 설정
      - name: Python 설치
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      # 3단계: 의존성 설치
      - name: 패키지 설치
        run: pip install -r requirements.txt

      # 4단계: 테스트 실행 (실패하면 배포 중단)
      - name: 테스트 실행
        run: pytest tests/

      # 5단계: AWS EC2에 배포
      - name: 서버 배포
        env:
          DEPLOY_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SERVER_IP: ${{ secrets.SERVER_IP }}
        run: |
          echo "$DEPLOY_KEY" > deploy_key.pem
          chmod 600 deploy_key.pem
          ssh -i deploy_key.pem ubuntu@$SERVER_IP \
            "cd ~/app && git pull && sudo systemctl restart myapp"
```

### GitHub Secrets 설정하기

배포에 필요한 비밀 정보는 코드에 직접 쓰지 않고 **Secrets**에 저장합니다.

```
GitHub 저장소 → Settings → Secrets and variables → Actions
→ New repository secret

이름: SSH_PRIVATE_KEY
값: (EC2 서버의 .pem 파일 내용)

이름: SERVER_IP
값: (EC2 서버의 IP 주소, 예: 13.125.xx.xx)
```

---

## 따라 하기 실습

### 실습 1 — AI와 함께 Issue 작성하기

1. GitHub 저장소의 **Issues** 탭에서 **New issue** 클릭
2. AI에게 아래 메시지 입력:
   ```
   "상품 목록 페이지에 검색 기능을 추가하는 GitHub Issue를 작성해줘.
    백엔드는 FastAPI, 프론트엔드는 React야. 완료 기준도 포함해줘."
   ```
3. AI가 생성한 내용을 Issue에 붙여넣고 제출
4. Issue 번호를 확인해서 메모 (예: `#15`)

### 실습 2 — AI가 작성한 코드를 브랜치에 커밋하기

1. 터미널에서 새 브랜치 생성:
   ```bash
   git checkout -b feature/issue-15-search
   ```
2. AI에게 실습 1의 Issue 내용을 보여주며 코드 요청:
   ```
   "이 Issue를 구현하는 코드를 작성해줘.
    파일 경로: backend/app/routers/products.py
    프론트: frontend/src/components/SearchBar.jsx"
   ```
3. 파일을 생성하고 커밋 후 push:
   ```bash
   git add backend/app/routers/products.py frontend/src/components/SearchBar.jsx
   git commit -m "feat: 상품 검색 기능 추가 (#15)"
   git push origin feature/issue-15-search
   ```

### 실습 3 — GitHub Actions 워크플로 추가하고 자동 배포 확인하기

1. 프로젝트에 `.github/workflows/` 디렉토리 생성:
   ```bash
   mkdir -p .github/workflows
   ```
2. AI에게 요청:
   ```
   "FastAPI 앱을 EC2에 배포하는 GitHub Actions 워크플로를 작성해줘.
    테스트는 pytest로 실행하고, 테스트 실패 시 배포가 중단되어야 해."
   ```
3. 생성된 내용을 `.github/workflows/deploy.yml`에 저장
4. GitHub Secrets에 `SSH_PRIVATE_KEY`와 `SERVER_IP` 등록
5. PR을 merge하고 **Actions 탭**에서 배포 진행 상황 확인

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| Secrets 이름 오타 | `Error: SSH_PRIVAT_KEY not found` | Settings → Secrets에서 이름 정확히 확인 |
| .pem 파일 권한 문제 | `WARNING: UNPROTECTED PRIVATE KEY FILE!` | `chmod 600 deploy_key.pem` 실행 |
| 브랜치 push 안 함 | PR 생성 시 브랜치가 안 보임 | `git push origin 브랜치이름` 먼저 실행 |
| 테스트 없이 배포 | Actions가 성공해도 앱이 안 됨 | `pytest tests/` 단계를 워크플로에 추가 |
| main 직접 push | 팀원 코드 덮어씀 | 반드시 브랜치 만들고 PR로 merge |
| 커밋 메시지에 Issue 번호 누락 | Issue와 코드 연결 안 됨 | 메시지 끝에 `(#이슈번호)` 추가 |
| YAML 들여쓰기 오류 | `mapping values are not allowed here` | YAML은 스페이스 2칸, 탭 사용 금지 |

---

## 확인 체크리스트

- [ ] GitHub에서 Issue를 생성하고 완료 기준을 명시했다
- [ ] AI에게 Issue 내용을 제공하고 코드를 받았다
- [ ] `main` 브랜치가 아닌 새 브랜치에서 작업했다
- [ ] 커밋 메시지에 Issue 번호를 포함했다 (예: `#12`)
- [ ] GitHub에 push하고 PR을 생성했다
- [ ] PR 설명에 변경 내용과 테스트 방법을 작성했다
- [ ] `.github/workflows/deploy.yml` 파일을 만들었다
- [ ] GitHub Secrets에 민감한 정보를 저장했다 (코드에 직접 쓰지 않음)
- [ ] Actions 탭에서 배포 성공 (초록색 체크) 을 확인했다
- [ ] 배포 후 실제 서버 주소에서 기능이 작동하는지 확인했다

---

## 한 번 더 생각해 보기

1. **왜 코드를 `main`에 직접 push하지 않고 브랜치를 만들까요?** — 혼자 작업할 때도 이 습관이 필요할까요? 어떤 상황에서 도움이 될지 생각해 보세요.

2. **테스트가 실패했을 때 배포가 자동으로 멈추는 것**은 어떤 문제를 예방해 줄까요? 테스트 없이 배포했다가 생길 수 있는 상황을 구체적으로 상상해 보세요.

3. **AI가 작성한 코드를 그대로 사용해도 될까요?** — AI가 틀리거나 보안 문제가 있는 코드를 생성할 수도 있습니다. 여러분은 어떻게 AI 코드를 검토할 수 있을까요?

---

## 다음 장

다음 장에서는 배포된 서비스에 실제 사용자 트래픽이 들어올 때 발생하는 문제를 모니터링하고, CloudWatch와 로그를 활용해 장애를 빠르게 감지하고 대응하는 방법을 배웁니다.