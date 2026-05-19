## 이 장에서 배우는 것

- GitHub Actions에서 환경 변수(Environment Variables)가 무엇인지 이해한다
- Secrets(비밀값)를 안전하게 저장하고 워크플로우에서 사용하는 방법을 익힌다
- `env:` 키워드로 워크플로우 전체, 잡(job), 스텝(step) 단위로 변수를 설정한다
- Repository Secrets, Environment Secrets, Organization Secrets의 차이를 구분한다
- 실수로 비밀값이 로그에 노출되는 상황을 예방한다

---

## 먼저 쉬운 설명

배포 자동화를 만들다 보면 반드시 이런 고민이 생깁니다.

> "AWS 비밀 키, DB 비밀번호 같은 민감한 정보를 코드에 직접 적으면 안 되는데… 어떻게 하지?"

코드 파일에 비밀번호를 그냥 쓰면 GitHub에 공개될 수 있고, 한 번 커밋 이력에 남으면 지우기가 매우 어렵습니다. 실제로 이 실수 하나로 클라우드 요금 폭탄을 맞거나 해킹 피해를 입은 사례가 많습니다.

GitHub Actions는 이 문제를 해결하기 위해 **Secrets(비밀값)** 기능을 제공합니다. 자물쇠가 달린 금고라고 생각하면 됩니다. 값을 한 번 저장하면 다시 볼 수 없고, 워크플로우 실행 중에만 꺼내 쓸 수 있습니다. 로그에도 `***`으로 자동 마스킹됩니다.

환경 변수는 그보다 덜 민감한 설정값(예: 리전 이름, 서버 URL, 로그 레벨)을 워크플로우 YAML 안에서 한 곳에 모아 관리하는 방법입니다.

이 두 가지를 제대로 익히면 "코드에는 비밀이 없는" 안전한 파이프라인을 만들 수 있습니다.

---

## 1. 환경 변수(env) 기본 사용법

환경 변수는 YAML 파일 안에 직접 써도 괜찮은 값들, 즉 민감하지 않은 설정에 사용합니다.

**설정 범위는 세 가지입니다:**

| 범위 | 위치 | 적용 대상 |
|------|------|-----------|
| 워크플로우 전체 | `on:` 아래 최상위 `env:` | 모든 잡, 모든 스텝 |
| 잡(job) | `jobs.<job-id>.env:` | 해당 잡의 모든 스텝 |
| 스텝(step) | `jobs.<job-id>.steps[*].env:` | 해당 스텝만 |

```yaml
# .github/workflows/deploy.yml

name: 배포 파이프라인

on:
  push:
    branches: [main]

# 워크플로우 전체에서 쓸 수 있는 환경 변수
env:
  APP_NAME: my-service
  AWS_REGION: ap-northeast-2
  NODE_ENV: production

jobs:
  build:
    runs-on: ubuntu-latest

    # 이 잡에서만 쓸 환경 변수
    env:
      BUILD_DIR: ./dist

    steps:
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: 빌드 실행
        # 스텝 안에서 환경 변수 참조: ${{ env.변수이름 }}
        run: |
          echo "앱 이름: ${{ env.APP_NAME }}"
          echo "빌드 결과 위치: ${{ env.BUILD_DIR }}"
          npm run build --outDir ${{ env.BUILD_DIR }}

      - name: 특정 스텝만 쓸 환경 변수
        env:
          LOG_LEVEL: debug
        run: echo "로그 레벨: $LOG_LEVEL"
```

> **팁:** `${{ env.변수명 }}`은 YAML 표현식 문법이고, `$변수명`은 셸(shell) 문법입니다. 둘 다 동작하지만 YAML 표현식은 `run:` 블록 밖(예: `with:` 인자)에서도 쓸 수 있어서 더 범용적입니다.

---

## 2. Secrets 저장하기 — GitHub 저장소 설정

비밀번호, API 키처럼 외부에 노출되면 안 되는 값은 반드시 **Secrets**로 저장해야 합니다.

**저장 방법 (GitHub 웹 UI):**

1. 저장소 → **Settings** 탭 클릭
2. 왼쪽 메뉴 **Secrets and variables** → **Actions** 클릭
3. **New repository secret** 버튼 클릭
4. `Name`: `AWS_ACCESS_KEY_ID`, `Secret`: 실제 키 값 입력 후 저장

저장한 뒤에는 **값을 다시 확인할 수 없습니다.** 이름만 보입니다. 값을 잃어버렸다면 새로 발급해서 덮어써야 합니다.

**워크플로우에서 Secrets 참조하기:**

```yaml
# .github/workflows/deploy-aws.yml

name: AWS 배포

on:
  push:
    branches: [main]

env:
  AWS_REGION: ap-northeast-2

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: AWS 자격증명 설정
        uses: aws-actions/configure-aws-credentials@v4
        with:
          # secrets 참조 문법: ${{ secrets.SECRET_이름 }}
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: S3에 파일 업로드
        run: |
          aws s3 sync ./dist s3://${{ secrets.S3_BUCKET_NAME }} \
            --delete \
            --region ${{ env.AWS_REGION }}
```

> **중요:** `secrets.`는 Secrets 전용 네임스페이스입니다. `env.`와 혼동하지 마세요.

---

## 3. Environments — 운영/스테이징 분리하기

같은 저장소에서 `staging`과 `production`을 따로 관리해야 할 때 **Environments**를 사용합니다. 각 환경마다 다른 Secrets와 변수를 설정할 수 있습니다.

**환경 생성 (GitHub 웹 UI):**

1. 저장소 → **Settings** → **Environments**
2. **New environment** → 이름 입력 (`staging`, `production`)
3. `production`에는 **Required reviewers** 설정(배포 전 승인 필요)을 추가할 수 있음

```yaml
# .github/workflows/deploy-env.yml

name: 환경별 배포

on:
  push:
    branches:
      - main        # main → production
      - develop     # develop → staging

jobs:
  deploy-staging:
    # develop 브랜치에서만 실행
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    # environment 키로 해당 환경의 Secrets 사용
    environment: staging

    steps:
      - uses: actions/checkout@v4

      - name: 스테이징 서버에 배포
        run: |
          echo "배포 대상: ${{ vars.SERVER_URL }}"
          # staging 환경의 Secrets 자동 적용
          curl -X POST "${{ vars.SERVER_URL }}/deploy" \
            -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}"

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production   # 승인 필요 환경

    steps:
      - uses: actions/checkout@v4

      - name: 운영 서버에 배포
        run: |
          echo "운영 배포 시작: ${{ vars.SERVER_URL }}"
          curl -X POST "${{ vars.SERVER_URL }}/deploy" \
            -H "Authorization: Bearer ${{ secrets.DEPLOY_TOKEN }}"
```

> **`vars.` vs `secrets.` 차이:** `vars.`는 Environment Variables(비민감 설정값, 로그에 그대로 표시), `secrets.`는 Secrets(민감값, 로그에 `***`으로 마스킹).

---

## 4. 비밀값이 로그에 노출되지 않도록 주의하기

GitHub Actions는 등록된 Secrets 값을 로그에서 자동으로 `***`으로 바꿔줍니다. 하지만 **우회되는 상황**을 알아야 합니다.

```yaml
# 위험한 예시 — 절대 이렇게 하지 마세요

steps:
  - name: 절대 하지 말 것 1: 비밀값을 직접 echo
    run: echo "${{ secrets.DB_PASSWORD }}"
    # 로그: *** (마스킹됨, 괜찮아 보이지만...)

  - name: 절대 하지 말 것 2: base64 인코딩 후 출력
    run: echo "${{ secrets.DB_PASSWORD }}" | base64
    # 로그: dGVzdHBhc3N3b3Jk — 마스킹 우회! 값이 노출됨

  - name: 절대 하지 말 것 3: JSON에 포함해서 출력
    run: |
      echo '{"password": "${{ secrets.DB_PASSWORD }}"}'
    # 일부 경우 마스킹이 되지 않을 수 있음
```

```yaml
# 안전한 예시

steps:
  - name: 올바른 방법: 환경 변수로 전달
    env:
      # secrets 값을 환경 변수에 담아 셸에 전달
      DB_PASS: ${{ secrets.DB_PASSWORD }}
    run: |
      # 셸 변수로 사용 (YAML 표현식으로 직접 조작하지 않음)
      ./scripts/db-migrate.sh
      # 스크립트 안에서 $DB_PASS 로 참조
```

**규칙:** Secrets를 `run:` 블록에서 직접 문자열 조작(자르기, 인코딩, JSON 직렬화)하지 마세요. 항상 `env:`로 셸 변수에 넘기고 셸 스크립트 안에서 처리하세요.

---

## 따라 하기 실습

### 실습 1 — 저장소에 Secrets 등록하고 워크플로우에서 사용하기

1. GitHub 저장소 → **Settings → Secrets and variables → Actions → New repository secret**
   - `Name`: `NOTIFY_WEBHOOK_URL`
   - `Secret`: (임시로 `https://example.com/hook` 입력)

2. 아래 파일을 생성합니다.

```yaml
# .github/workflows/practice-secrets.yml

name: Secrets 실습

on:
  workflow_dispatch:   # 수동 실행 버튼 활성화

jobs:
  notify:
    runs-on: ubuntu-latest

    steps:
      - name: 웹훅 URL 확인 (마스킹 테스트)
        run: |
          echo "웹훅 호출 시작"
          # 값 자체를 출력하면 *** 로 마스킹됨
          echo "URL: ${{ secrets.NOTIFY_WEBHOOK_URL }}"

      - name: 실제 웹훅 호출
        env:
          WEBHOOK: ${{ secrets.NOTIFY_WEBHOOK_URL }}
        run: |
          curl -s -o /dev/null -w "HTTP 상태코드: %{http_code}\n" \
            -X POST "$WEBHOOK" \
            -H "Content-Type: application/json" \
            -d '{"message": "배포 완료"}'
```

3. **Actions 탭 → practice-secrets 워크플로우 → Run workflow** 로 수동 실행하고 로그를 확인합니다. `URL: ***`으로 마스킹된 것을 확인하세요.

---

### 실습 2 — 환경 변수로 빌드 설정 통합 관리하기

실습 1에서 만든 저장소에 다음 파일을 추가합니다.

```yaml
# .github/workflows/practice-env.yml

name: 환경 변수 실습

on:
  push:
    branches: [main]

env:
  APP_VERSION: "1.0.0"
  NODE_VERSION: "20"
  DEPLOY_REGION: ap-northeast-2

jobs:
  build-and-info:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Node.js 설치
        uses: actions/setup-node@v4
        with:
          # env 변수를 with: 안에서도 사용 가능
          node-version: ${{ env.NODE_VERSION }}

      - name: 빌드 정보 출력
        run: |
          echo "==============================="
          echo "앱 버전  : ${{ env.APP_VERSION }}"
          echo "Node 버전: ${{ env.NODE_VERSION }}"
          echo "배포 리전: ${{ env.DEPLOY_REGION }}"
          echo "커밋 SHA : ${{ github.sha }}"
          echo "==============================="

      - name: package.json 버전과 일치 확인
        run: |
          PKG_VER=$(node -p "require('./package.json').version" 2>/dev/null || echo "파일 없음")
          echo "package.json 버전: $PKG_VER"
          echo "워크플로우 버전  : ${{ env.APP_VERSION }}"
```

`main` 브랜치에 푸시한 뒤 Actions 탭에서 빌드 정보가 올바르게 출력되는지 확인합니다.

---

### 실습 3 — staging / production 환경 분리하기

실습 1~2를 완성한 뒤, 환경별 배포를 설정합니다.

1. **Settings → Environments** 에서 `staging` 환경 생성 후 Environment Variable 추가:
   - `Name`: `SERVER_URL`, `Value`: `https://staging.example.com`

2. 아래 워크플로우를 작성합니다.

```yaml
# .github/workflows/practice-environment.yml

name: 환경 분리 실습

on:
  push:
    branches: [main, develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    # 브랜치에 따라 다른 environment 선택
    environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}

    steps:
      - uses: actions/checkout@v4

      - name: 배포 환경 확인
        run: |
          echo "브랜치  : ${{ github.ref_name }}"
          echo "서버 URL: ${{ vars.SERVER_URL }}"
          echo "배포 시작!"
```

3. `develop` 브랜치와 `main` 브랜치 각각에 푸시한 후 출력되는 `SERVER_URL`이 다른지 비교합니다.

---

## 자주 하는 실수

| 실수 | 에러 메시지 / 증상 | 해결 방법 |
|------|-------------------|-----------|
| Secret 이름 오타 | `${{ secrets.MY_KYE }}` → 빈 문자열, 오류 없이 진행 | Secrets 탭에서 이름 철자 재확인. 빈 값 체크 스텝 추가 |
| `env.` 대신 `secrets.` 사용 | 값이 로그에 표시되지 않아 디버깅 어려움 | 일반 설정값은 `env:`, 민감값만 `secrets:` 구분 |
| `if:` 조건에 secrets 사용 시도 | `Error: Context access might be invalid` | Secrets는 `if:` 조건에 직접 사용 불가. 스텝 내부에서 처리 |
| Fork PR에서 Secrets가 빈 값 | 워크플로우 실행은 되지만 API 호출 실패 | 외부 Fork PR에는 Secrets가 전달되지 않는 것이 의도된 보안 동작 |
| Environment를 `environment:` 키 없이 참조 | `vars.SERVER_URL` → 빈 문자열 | 잡(job)에 `environment: 환경이름` 명시 필요 |
| Secrets 값에 개행문자 포함 | `curl: option --header: blank or empty header` | 여러 줄 Secrets 저장 시 trailing newline 없이 저장 |
| `run:` 에서 표현식 직접 조작 | 마스킹 우회로 값 노출 가능 | `env:`로 셸 변수에 할당 후 스크립트에서 처리 |

---

## 확인 체크리스트

- [ ] `secrets.SECRET_NAME` 문법으로 Secrets를 워크플로우에서 참조할 수 있다
- [ ] `env:` 키워드를 워크플로우 전체, 잡, 스텝 세 단계에 각각 설정할 수 있다
- [ ] GitHub Secrets 탭에서 새 Secret을 등록하고 이름을 확인할 수 있다
- [ ] `vars.`(Environment Variables)와 `secrets.`(Secrets)의 차이를 설명할 수 있다
- [ ] `staging`과 `production` 환경을 분리하고 각각 다른 값을 쓸 수 있다
- [ ] Secrets 값을 `echo`로 직접 출력하면 `***`으로 마스킹됨을 확인했다
- [ ] 민감한 값을 `run:` 블록에서 직접 문자열 조작하지 않아야 함을 이해했다
- [ ] Fork PR에서 Secrets가 전달되지 않는 이유를 설명할 수 있다

---

## 한 번 더 생각해 보기

1. 팀원이 퇴사했습니다. 그 팀원만 알고 있던 AWS 비밀 키가 저장소 Secrets에 등록되어 있었다면, 어떤 순서로 키를 교체해야 보안 공백이 생기지 않을까요?

2. 모노레포(monorepo)처럼 하나의 저장소에서 서로 다른 팀이 여러 서비스를 관리한다면, Repository Secrets보다 Environment Secrets를 쓰는 것이 왜 더 나을지 생각해 보세요.

3. `secrets.TOKEN`을 `run:` 블록에서 base64로 인코딩해서 출력하면 GitHub의 자동 마스킹이 왜 동작하지 않을까요? 마스킹은 정확히 어떤 원리로 작동하는 걸까요?

---

## 다음 장

다음 장에서는 지금까지 배운 Secrets와 환경 변수를 활용하여 실제 애플리케이션을 AWS Lambda에 자동 배포하는 완전한 CI/CD 파이프라인을 처음부터 끝까지 구축해 봅니다.