# Chapter 04: GitHub Actions로 자동 배포하기

## 이 장에서 배우는 것

- GitHub Actions가 무엇인지
- 코드를 push하면 자동으로 Lambda에 배포되도록 설정하는 방법
- workflow 파일(`.yml`) 구조 이해하기
- AWS 인증 정보를 GitHub에 안전하게 저장하는 방법
- 배포 성공/실패를 GitHub에서 확인하는 방법

---

## 먼저 쉬운 설명

지금까지는 코드를 수정할 때마다 Lambda 콘솔을 열어서 직접 붙여넣고 Deploy를 클릭했다.

이 과정이 반복되면 실수가 생길 수 있고 번거롭다.

**GitHub Actions**는 "main 브랜치에 push되면 자동으로 Lambda에 배포해라"처럼 자동화 규칙을 설정하는 도구다.

```
git push origin main
    ↓
GitHub Actions 자동 실행
    ↓
Lambda에 코드 자동 업로드
    ↓
Deploy 완료
```

---

## 1. GitHub Actions 기본 구조

Actions는 저장소의 `.github/workflows/` 폴더 안에 `.yml` 파일로 정의한다.

```
memo-service/
├── lambda_function.py
├── .github/
│   └── workflows/
│       └── deploy.yml    ← Actions 설정 파일
```

`.yml` 파일의 기본 구조:

```yaml
name: 워크플로우 이름

on:                        # 언제 실행할지
  push:
    branches: [main]       # main에 push될 때

jobs:                      # 무엇을 실행할지
  deploy:
    runs-on: ubuntu-latest # 어떤 환경에서 실행할지

    steps:                 # 단계별 실행 목록
      - name: 코드 체크아웃
        uses: actions/checkout@v4

      - name: Python 설정
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'
```

---

## 2. AWS 인증 정보 GitHub에 저장하기

Lambda에 배포하려면 AWS 접근 권한이 필요하다. 이 정보를 코드에 직접 넣으면 안 된다.

### IAM 사용자 Access Key 만들기

1. AWS 콘솔 → IAM → Users → 내 사용자 선택
2. **Security credentials** 탭 → **Create access key**
3. Use case: **Other** 선택
4. Key 생성 후 **Access key ID**와 **Secret access key** 복사 (이 화면에서만 볼 수 있음)

### GitHub Secrets에 저장하기

1. GitHub 저장소 → **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret** 클릭
3. 두 가지 저장:
   - Name: `AWS_ACCESS_KEY_ID`, Value: 복사한 Access key ID
   - Name: `AWS_SECRET_ACCESS_KEY`, Value: 복사한 Secret access key

---

## 3. 배포 workflow 파일 만들기

`.github/workflows/deploy.yml` 파일을 만든다:

```yaml
name: Deploy to AWS Lambda

on:
  push:
    branches: [main]   # main 브랜치 push 시 실행

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # 1. 코드 가져오기
      - name: Checkout code
        uses: actions/checkout@v4

      # 2. Python 설정
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      # 3. AWS CLI 설정
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      # 4. Lambda에 배포
      - name: Deploy to Lambda
        run: |
          zip deployment.zip lambda_function.py
          aws lambda update-function-code \
            --function-name memo-service \
            --zip-file fileb://deployment.zip

      # 5. 배포 완료 확인
      - name: Verify deployment
        run: |
          aws lambda get-function \
            --function-name memo-service \
            --query 'Configuration.LastModified'
```

---

## 4. 배포 실행하고 확인하기

### 코드 push

```bash
git add .github/workflows/deploy.yml
git commit -m "ci: GitHub Actions Lambda 자동 배포 추가"
git push origin main
```

### Actions 진행 상황 확인

1. GitHub 저장소 → **Actions** 탭
2. 방금 push한 workflow 실행이 보임
3. 각 step 옆에 ✅ 또는 ❌ 표시
4. 실패한 step 클릭 → 로그 확인

### 성공 화면

```
✅ Checkout code
✅ Set up Python
✅ Configure AWS credentials
✅ Deploy to Lambda
✅ Verify deployment
```

---

## 5. 의존성 패키지 포함하기

Lambda에서 외부 패키지(예: `requests`)를 쓴다면 배포 패키지에 포함해야 한다.

```yaml
      # 패키지 설치 후 함께 zip
      - name: Install dependencies
        run: |
          pip install -r requirements.txt -t ./package

      - name: Deploy to Lambda
        run: |
          cp lambda_function.py ./package/
          cd package && zip -r ../deployment.zip .
          cd ..
          aws lambda update-function-code \
            --function-name memo-service \
            --zip-file fileb://deployment.zip
```

`requirements.txt` 예:
```
requests==2.31.0
```

---

## 6. 테스트 통과 후 배포하기 (선택)

배포 전에 자동으로 테스트를 실행하고, 실패하면 배포를 중단할 수 있다.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'
      - name: Run tests
        run: python -m unittest discover -s . -p "test_*.py"

  deploy:
    needs: test           # test job이 성공해야만 실행
    runs-on: ubuntu-latest
    steps:
      # ... 배포 단계
```

---

## 따라 하기 실습

### 실습 1. workflow 파일 만들고 배포 테스트

1. `.github/workflows/deploy.yml` 파일 생성
2. GitHub Secrets에 AWS 인증 정보 저장
3. `lambda_function.py`에 버전 정보 추가

```python
VERSION = "1.1.0"

def lambda_handler(event, context):
    # ... 기존 코드
    print(f"[VERSION] {VERSION}")
```

4. push → Actions 탭에서 배포 확인
5. Lambda URL 호출 → CloudWatch Logs에서 VERSION 로그 확인

### 실습 2. 실패하는 배포 확인

의도적으로 잘못된 YAML 문법을 만들어서 Actions가 어떻게 실패를 표시하는지 확인한다.

```yaml
      - name: Deploy to Lambda
        run |           # : 빠짐
          zip deployment.zip lambda_function.py
```

push 후 Actions 탭에서 오류 메시지 확인.

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| Secrets 이름 오타 | `Error: Input required and not supplied: aws-access-key-id` | Secrets 이름 정확히 확인 |
| 함수 이름 오타 | `ResourceNotFoundException` | `--function-name` 값 확인 |
| 리전 잘못 지정 | 함수를 못 찾음 | `aws-region`을 함수가 있는 리전으로 설정 |
| YAML 들여쓰기 오류 | workflow 파싱 실패 | 스페이스 2칸 또는 4칸 일관되게 사용 (탭 금지) |

---

## 확인 체크리스트

- [ ] `.github/workflows/deploy.yml` 파일을 만들 수 있는가
- [ ] AWS 인증 정보를 GitHub Secrets에 저장할 수 있는가
- [ ] main 브랜치에 push 후 Actions 탭에서 결과를 확인할 수 있는가
- [ ] 배포 성공 후 Lambda URL로 동작을 검증할 수 있는가

---

## 다음 단계

이 장까지 완료하면 전체 흐름이 자동화됩니다:

```
코드 수정 → git push → GitHub Actions → Lambda 자동 배포 → URL로 바로 사용
```

---

## 참고 자료

- GitHub Actions 공식 문서 — https://docs.github.com/en/actions
- aws-actions/configure-aws-credentials — https://github.com/aws-actions/configure-aws-credentials
- AWS Lambda update-function-code — https://docs.aws.amazon.com/cli/latest/reference/lambda/update-function-code.html
