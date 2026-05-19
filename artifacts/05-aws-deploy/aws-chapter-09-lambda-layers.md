## 이 장에서 배우는 것

- AWS Lambda Layer가 무엇인지, 왜 필요한지 이해한다
- Python 패키지를 올바른 폴더 구조로 패키징하는 방법을 배운다
- ZIP 파일로 Layer를 만들고 AWS에 업로드하는 방법을 익힌다
- Lambda 함수에 Layer를 연결하는 방법을 실습한다
- Layer 버전 관리와 여러 함수에서 공유하는 방법을 이해한다

---

## 먼저 쉬운 설명

Lambda 함수를 처음 만들 때는 코드가 단순해서 괜찮습니다. 그런데 `requests`, `pandas`, `boto3` 같은 외부 패키지가 필요해지면 문제가 생깁니다. Lambda는 여러분의 노트북이 아니기 때문에, 직접 패키지를 설치해 줘야 합니다.

가장 흔한 방법은 코드와 패키지를 함께 ZIP으로 묶는 것인데, 이렇게 하면 함수 10개를 만들 때마다 같은 패키지를 10번 복사해야 합니다. 파일 크기도 크고, 한 패키지 버전을 바꾸려면 10개를 다 수정해야 합니다.

**Lambda Layer**는 이 문제를 해결합니다. 패키지를 한 번만 업로드해 놓고, 여러 함수에서 "저는 저 Layer 쓸게요"라고 연결만 하면 됩니다. 마치 회사 공용 프린터처럼 — 각자 프린터를 살 필요 없이 연결만 하면 됩니다.

---

## 1. Lambda Layer의 구조 이해하기

Lambda Layer는 ZIP 파일 안에 **정해진 폴더 구조**가 있어야 합니다. Python 패키지는 반드시 `python/` 폴더 아래에 있어야 합니다. 이 경로를 틀리면 Lambda가 패키지를 찾지 못합니다.

```
my-layer.zip
└── python/
    ├── requests/
    ├── requests-2.31.0.dist-info/
    ├── certifi/
    └── urllib3/
```

Python 런타임별로 경로를 더 구체적으로 지정할 수도 있습니다:

```
my-layer.zip
└── python/
    └── lib/
        └── python3.11/
            └── site-packages/
                └── requests/
```

> 초보자에게는 `python/` 바로 아래에 패키지를 넣는 첫 번째 방식이 더 간단합니다. Lambda가 두 경로를 모두 인식합니다.

---

## 2. 로컬에서 패키지 설치하고 패키징하기

`pip install` 명령어에 `-t` 옵션을 쓰면 원하는 폴더에 패키지를 설치할 수 있습니다.

```bash
# 1. 작업 폴더 만들기
mkdir my-python-layer
cd my-python-layer

# 2. python/ 폴더 안에 패키지 설치
pip install requests -t python/

# 3. 설치된 내용 확인
ls python/
# 출력 예시:
# requests/  certifi/  urllib3/  charset_normalizer/

# 4. ZIP으로 압축 (python/ 폴더째로 묶어야 함)
zip -r my-layer.zip python/

# 파일 크기 확인
ls -lh my-layer.zip
# -rw-r--r--  1 user  staff  1.2M  my-layer.zip
```

**중요:** `python/` 폴더 *안*에서 압축하면 안 됩니다. `python/` 폴더를 *포함해서* 압축해야 합니다.

```bash
# 잘못된 방법 — python/ 안으로 들어가서 압축
cd python
zip -r ../my-layer.zip .   # ❌ 이렇게 하면 경로가 틀립니다

# 올바른 방법 — python/ 폴더가 있는 위치에서 압축
cd ..
zip -r my-layer.zip python/  # ✅
```

---

## 3. AWS CLI로 Layer 업로드하기

ZIP 파일이 준비됐으면 AWS에 업로드합니다.

```bash
# Layer 업로드 명령어
aws lambda publish-layer-version \
  --layer-name "my-requests-layer" \
  --description "requests 라이브러리 Layer" \
  --zip-file fileb://my-layer.zip \
  --compatible-runtimes python3.11 python3.12

# 업로드 성공 시 출력 예시:
# {
#     "LayerVersionArn": "arn:aws:lambda:ap-northeast-2:123456789012:layer:my-requests-layer:1",
#     "Version": 1,
#     "Description": "requests 라이브러리 Layer",
#     "CreatedDate": "2026-05-18T10:00:00.000+0000",
#     "CompatibleRuntimes": ["python3.11", "python3.12"]
# }
```

출력에서 `LayerVersionArn` 값을 메모해 두세요. Lambda 함수에 연결할 때 필요합니다.

---

## 4. Lambda 함수에 Layer 연결하기

Layer를 Lambda 함수에 연결하는 방법은 두 가지입니다.

**방법 1: AWS CLI 사용**

```bash
# 기존 함수에 Layer 추가
aws lambda update-function-configuration \
  --function-name my-api-function \
  --layers "arn:aws:lambda:ap-northeast-2:123456789012:layer:my-requests-layer:1"

# 여러 Layer를 동시에 연결할 때 (공백으로 구분)
aws lambda update-function-configuration \
  --function-name my-api-function \
  --layers \
    "arn:aws:lambda:ap-northeast-2:123456789012:layer:my-requests-layer:1" \
    "arn:aws:lambda:ap-northeast-2:123456789012:layer:my-pandas-layer:2"
```

**방법 2: Lambda 함수 코드에서 import 테스트**

```python
# lambda_function.py
import json
import requests  # Layer에서 가져옴

def lambda_handler(event, context):
    # Layer가 제대로 연결됐으면 이 코드가 동작합니다
    response = requests.get("https://httpbin.org/get")
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "status": response.status_code,
            "message": "Layer 연결 성공!"
        })
    }
```

---

## 5. requirements.txt로 여러 패키지 한 번에 관리하기

실제 프로젝트에서는 패키지가 여러 개입니다. `requirements.txt`를 사용하면 목록을 관리하기 쉽습니다.

```text
# requirements.txt
requests==2.31.0
boto3==1.34.0
python-dateutil==2.8.2
```

```bash
# requirements.txt의 패키지를 모두 python/ 폴더에 설치
pip install -r requirements.txt -t python/

# 불필요한 파일 제거 (ZIP 크기 줄이기)
find python/ -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null
find python/ -name "*.pyc" -delete
find python/ -name "*.dist-info" -type d -exec rm -rf {} + 2>/dev/null

# ZIP 압축
zip -r my-layer.zip python/

echo "완료! 파일 크기:"
ls -lh my-layer.zip
```

Lambda Layer의 최대 크기는 **압축 전 250MB**, **ZIP 파일 50MB** (직접 업로드 시)입니다. 50MB를 초과하면 S3를 통해 업로드해야 합니다.

```bash
# 50MB 초과 시 — S3를 거쳐 업로드
aws s3 cp my-layer.zip s3://my-bucket/layers/my-layer.zip

aws lambda publish-layer-version \
  --layer-name "my-large-layer" \
  --description "대용량 패키지 Layer" \
  --content S3Bucket=my-bucket,S3Key=layers/my-layer.zip \
  --compatible-runtimes python3.11
```

---

## 따라 하기 실습

### 실습 1: requests Layer 만들기

아래 명령어를 순서대로 실행해서 첫 번째 Layer를 만들어 봅니다.

```bash
# 1. 작업 디렉토리 생성
mkdir ~/lambda-layer-practice
cd ~/lambda-layer-practice

# 2. python 폴더에 requests 설치
pip install requests -t python/

# 3. 설치 확인
ls python/

# 4. ZIP 생성
zip -r requests-layer-v1.zip python/

# 5. AWS에 업로드 (리전을 본인 리전으로 바꾸세요)
aws lambda publish-layer-version \
  --layer-name "practice-requests-layer" \
  --description "실습용 requests Layer" \
  --zip-file fileb://requests-layer-v1.zip \
  --compatible-runtimes python3.11 \
  --region ap-northeast-2
```

성공하면 터미널에 JSON 응답이 출력됩니다. `"Version": 1`이 보이면 성공입니다.

---

### 실습 2: Layer를 사용하는 Lambda 함수 배포하기

실습 1에서 만든 Layer ARN을 사용합니다.

```bash
# 1. Lambda 함수 코드 작성
cat > weather_function.py << 'EOF'
import json
import requests

def lambda_handler(event, context):
    city = event.get("city", "Seoul")
    
    # 실제 API 대신 테스트용 엔드포인트 사용
    response = requests.get(f"https://httpbin.org/anything?city={city}")
    data = response.json()
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "city": city,
            "requests_version": requests.__version__,
            "layer_working": True
        }, ensure_ascii=False)
    }
EOF

# 2. 함수 코드만 ZIP (Layer는 별도)
zip weather-function.zip weather_function.py

# 3. Lambda 함수 생성 (IAM Role ARN은 본인 것으로 교체)
aws lambda create-function \
  --function-name weather-practice-function \
  --runtime python3.11 \
  --role arn:aws:iam::123456789012:role/lambda-basic-role \
  --handler weather_function.lambda_handler \
  --zip-file fileb://weather-function.zip \
  --layers "arn:aws:lambda:ap-northeast-2:123456789012:layer:practice-requests-layer:1"
```

---

### 실습 3: Layer 버전 업데이트하기

패키지 버전을 올리거나 패키지를 추가할 때 Layer를 새 버전으로 업데이트합니다.

```bash
# 1. 기존 python/ 폴더 삭제 후 재설치 (버전 명시)
rm -rf python/
pip install "requests==2.32.0" -t python/

# 2. 새 ZIP 생성
zip -r requests-layer-v2.zip python/

# 3. 같은 Layer 이름으로 재업로드 — 자동으로 Version 2가 됩니다
aws lambda publish-layer-version \
  --layer-name "practice-requests-layer" \
  --description "requests 2.32.0 업그레이드" \
  --zip-file fileb://requests-layer-v2.zip \
  --compatible-runtimes python3.11 \
  --region ap-northeast-2

# 4. 함수를 새 Layer 버전으로 업데이트
aws lambda update-function-configuration \
  --function-name weather-practice-function \
  --layers "arn:aws:lambda:ap-northeast-2:123456789012:layer:practice-requests-layer:2"

# 5. 테스트 실행
aws lambda invoke \
  --function-name weather-practice-function \
  --payload '{"city": "부산"}' \
  --cli-binary-format raw-in-base64-out \
  output.json

cat output.json
```

---

## 자주 하는 실수

| 실수 | 에러 메시지 | 원인 | 해결 방법 |
|------|------------|------|-----------|
| ZIP 폴더 구조 오류 | `Unable to import module 'lambda_function': No module named 'requests'` | `python/` 폴더 없이 패키지를 바로 압축함 | `python/` 폴더를 포함해서 ZIP 생성: `zip -r layer.zip python/` |
| 잘못된 위치에서 압축 | `Unable to import module` (같은 에러) | `cd python/` 안에서 `zip` 실행 | `python/` 폴더 *밖*에서 `zip -r layer.zip python/` 실행 |
| 런타임 불일치 | `Runtime.ImportModuleError` | Layer는 python3.9용인데 함수는 python3.11 사용 | `--compatible-runtimes`에 함수 런타임 추가 후 재업로드 |
| ZIP 파일 50MB 초과 | `Request must be smaller than 52428800 bytes` | pandas, numpy 같은 대용량 패키지 직접 업로드 시도 | S3에 먼저 업로드 후 `--content S3Bucket=...,S3Key=...` 사용 |
| Layer ARN 버전 누락 | `InvalidParameterValueException: Layer version ARN ... is invalid` | ARN 끝에 `:1` 같은 버전 번호 빠짐 | ARN 형식 확인: `arn:aws:lambda:REGION:ACCOUNT:layer:NAME:VERSION` |
| 오래된 Layer 참조 | 예전 패키지 버전이 계속 실행됨 | 새 Layer 버전 업로드 후 함수 업데이트 안 함 | `update-function-configuration --layers` 로 새 버전 ARN으로 교체 |
| 함수 크기 250MB 초과 | `Unzipped size must be smaller than 262144000 bytes` | Layer + 함수 코드 합산이 250MB 초과 | 불필요한 `.dist-info`, `__pycache__` 삭제 후 재패키징 |

---

## 확인 체크리스트

- [ ] `pip install -t python/` 명령어로 `python/` 폴더 안에 패키지가 설치됐다
- [ ] `zip -r layer.zip python/` 으로 `python/` 폴더를 포함해서 압축했다
- [ ] `aws lambda publish-layer-version` 명령어 결과에서 `LayerVersionArn`을 메모했다
- [ ] ARN 끝에 버전 번호(`:1`, `:2` 등)가 포함되어 있다
- [ ] Lambda 함수의 런타임(python3.11)과 Layer의 `--compatible-runtimes`가 일치한다
- [ ] `update-function-configuration --layers` 로 함수에 Layer를 연결했다
- [ ] 테스트 실행 시 `import` 에러 없이 함수가 정상 응답했다
- [ ] `requirements.txt`로 패키지 버전을 관리하고 있다

---

## 한 번 더 생각해 보기

1. Lambda 함수 10개가 모두 `requests` 패키지를 사용한다면, Layer를 쓸 때와 각 함수에 패키지를 포함시킬 때의 차이는 무엇일까요? 패키지 버전을 2.31.0에서 2.32.0으로 올려야 할 때 각각 몇 번 작업해야 할까요?

2. Layer의 최대 크기는 압축 해제 후 250MB입니다. `pandas`와 `numpy`를 함께 설치하면 약 150MB가 됩니다. 여기에 `scikit-learn`까지 추가하면 어떤 문제가 생길 수 있고, 어떻게 해결할 수 있을까요?

3. 개발(`dev`), 스테이징(`staging`), 운영(`prod`) 환경이 따로 있다면 Layer를 어떻게 관리하는 게 좋을까요? 환경마다 Layer 이름을 다르게 해야 할까요, 아니면 같은 Layer에서 버전으로 구분해야 할까요?

---

## 다음 장

다음 장에서는 Lambda 함수와 API Gateway를 연결해서 실제 HTTP 엔드포인트를 만들고, 외부에서 호출 가능한 REST API를 배포하는 방법을 배웁니다.