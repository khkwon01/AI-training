## 이 장에서 배우는 것

- AWS S3 버킷(Bucket)이 무엇인지, 왜 정적 파일 호스팅에 적합한지 이해한다
- Python(boto3)으로 S3에 파일을 업로드하는 방법을 익힌다
- 버킷의 퍼블릭 접근 설정과 정적 웹사이트 호스팅을 활성화하는 방법을 배운다
- Flask / FastAPI 앱에서 S3 URL을 이용해 정적 파일을 제공하는 패턴을 익힌다
- 업로드 자동화 스크립트를 직접 작성해 본다

---

## 먼저 쉬운 설명

웹 앱을 만들면 이미지, CSS, JavaScript 파일 같은 **정적 파일**이 꼭 필요합니다.
이 파일들을 Python 서버에서 직접 보내주면 서버가 느려질 수 있어요.

AWS S3는 쉽게 말해 **인터넷에 있는 거대한 USB 드라이브**입니다.
파일을 한 번 올려두면 전 세계 어디서나 빠르게 내려받을 수 있고,
서버는 "그 URL 보세요!"라고만 알려주면 되니까 훨씬 가볍게 동작합니다.

실제로 대부분의 실무 웹 서비스(쿠팡, 배달의민족 등)가 이 방식으로 이미지를 제공합니다.
이 장을 마치면 여러분의 앱도 같은 방식으로 정적 파일을 관리할 수 있습니다.

---

## 1. S3 핵심 개념 — 버킷과 객체

S3에서 파일은 **객체(Object)**, 파일이 담기는 폴더 같은 공간은 **버킷(Bucket)**이라고 부릅니다.

| 일반 개념 | S3 개념 |
|-----------|---------|
| 하드 드라이브 | 버킷(Bucket) |
| 파일 | 객체(Object) |
| 파일 경로 | 키(Key) |
| 파일 주소 | URL |

버킷 이름은 **전 세계에서 유일**해야 합니다. `my-app`처럼 흔한 이름은 이미 누군가 사용 중일 수 있어요.

```python
# 버킷 이름 규칙 예시
# 좋음: my-webapp-profile-images-2024
# 나쁨: My App (대문자, 공백 사용 불가)
# 나쁨: my_app (언더스코어 사용 불가)

BUCKET_NAME = "my-webapp-profile-images-2024"
REGION = "ap-northeast-2"  # 서울 리전
```

---

## 2. 개발 환경 준비 — boto3 설치와 자격증명 설정

boto3는 Python에서 AWS를 다루는 공식 라이브러리입니다.

```bash
pip install boto3
```

자격증명(Access Key)은 코드에 직접 쓰면 절대 안 됩니다. 환경변수로 관리하세요.

```bash
# .env 파일 (절대 Git에 올리지 마세요!)
AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
AWS_DEFAULT_REGION=ap-northeast-2
```

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

AWS_ACCESS_KEY_ID = os.environ["AWS_ACCESS_KEY_ID"]
AWS_SECRET_ACCESS_KEY = os.environ["AWS_SECRET_ACCESS_KEY"]
AWS_REGION = os.environ.get("AWS_DEFAULT_REGION", "ap-northeast-2")
BUCKET_NAME = os.environ["S3_BUCKET_NAME"]
```

```python
# s3_client.py
import boto3
from config import AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION

s3 = boto3.client(
    "s3",
    aws_access_key_id=AWS_ACCESS_KEY_ID,
    aws_secret_access_key=AWS_SECRET_ACCESS_KEY,
    region_name=AWS_REGION,
)
```

---

## 3. 버킷 생성과 정적 웹사이트 호스팅 활성화

```python
# setup_bucket.py
import boto3
import json
from config import AWS_REGION, BUCKET_NAME

s3 = boto3.client("s3", region_name=AWS_REGION)


def create_bucket(bucket_name: str, region: str) -> None:
    """버킷을 만들고 정적 호스팅을 설정합니다."""
    # 1. 버킷 생성 (서울 리전은 LocationConstraint 필수)
    s3.create_bucket(
        Bucket=bucket_name,
        CreateBucketConfiguration={"LocationConstraint": region},
    )
    print(f"버킷 생성 완료: {bucket_name}")

    # 2. 퍼블릭 액세스 차단 해제 (정적 호스팅용)
    s3.delete_public_access_block(Bucket=bucket_name)

    # 3. 버킷 정책으로 읽기 공개 허용
    public_policy = {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "PublicReadGetObject",
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": f"arn:aws:s3:::{bucket_name}/*",
            }
        ],
    }
    s3.put_bucket_policy(
        Bucket=bucket_name,
        Policy=json.dumps(public_policy),
    )

    # 4. 정적 웹사이트 호스팅 활성화
    s3.put_bucket_website(
        Bucket=bucket_name,
        WebsiteConfiguration={
            "IndexDocument": {"Suffix": "index.html"},
            "ErrorDocument": {"Key": "error.html"},
        },
    )
    print("정적 웹사이트 호스팅 활성화 완료!")


if __name__ == "__main__":
    create_bucket(BUCKET_NAME, AWS_REGION)
```

---

## 4. 파일 업로드 — 단일 파일과 폴더 전체

```python
# uploader.py
import os
import mimetypes
import boto3
from config import BUCKET_NAME, AWS_REGION

s3 = boto3.client("s3", region_name=AWS_REGION)


def upload_file(local_path: str, s3_key: str) -> str:
    """파일 하나를 업로드하고 공개 URL을 반환합니다."""
    content_type, _ = mimetypes.guess_type(local_path)
    content_type = content_type or "application/octet-stream"

    s3.upload_file(
        Filename=local_path,
        Bucket=BUCKET_NAME,
        Key=s3_key,
        ExtraArgs={"ContentType": content_type},
    )

    url = f"https://{BUCKET_NAME}.s3.{AWS_REGION}.amazonaws.com/{s3_key}"
    print(f"업로드 완료: {url}")
    return url


def upload_directory(local_dir: str, s3_prefix: str = "") -> list[str]:
    """디렉터리 전체를 S3에 업로드합니다."""
    uploaded_urls = []

    for root, _dirs, files in os.walk(local_dir):
        for filename in files:
            local_path = os.path.join(root, filename)
            # 로컬 경로에서 S3 키 생성
            relative_path = os.path.relpath(local_path, local_dir)
            s3_key = os.path.join(s3_prefix, relative_path).replace("\\", "/")

            url = upload_file(local_path, s3_key)
            uploaded_urls.append(url)

    return uploaded_urls


if __name__ == "__main__":
    # 사용 예시
    upload_file("static/logo.png", "images/logo.png")
    upload_directory("static/", "static/")
```

---

## 5. Flask / FastAPI 앱과 연동하기

### Flask 예시

```python
# app_flask.py
from flask import Flask, render_template, request, redirect, url_for
from uploader import upload_file
import tempfile
import os

app = Flask(__name__)

S3_BASE_URL = f"https://{os.environ['S3_BUCKET_NAME']}.s3.ap-northeast-2.amazonaws.com"


@app.route("/upload", methods=["GET", "POST"])
def upload_profile_image():
    if request.method == "POST":
        file = request.files.get("image")
        if not file:
            return "파일을 선택해 주세요.", 400

        # 임시 파일로 저장 후 S3 업로드
        with tempfile.NamedTemporaryFile(delete=False, suffix=os.path.splitext(file.filename)[1]) as tmp:
            file.save(tmp.name)
            s3_key = f"profiles/{file.filename}"
            image_url = upload_file(tmp.name, s3_key)
        os.unlink(tmp.name)  # 임시 파일 삭제

        return redirect(url_for("show_image", key=s3_key))

    return render_template("upload.html")


@app.route("/image/<path:key>")
def show_image(key):
    image_url = f"{S3_BASE_URL}/{key}"
    return render_template("image.html", image_url=image_url)
```

### FastAPI 예시

```python
# app_fastapi.py
from fastapi import FastAPI, UploadFile, File
from fastapi.responses import JSONResponse
from uploader import upload_file
import tempfile
import os

app = FastAPI()

S3_BASE_URL = f"https://{os.environ['S3_BUCKET_NAME']}.s3.ap-northeast-2.amazonaws.com"


@app.post("/upload/image")
async def upload_image(file: UploadFile = File(...)):
    suffix = os.path.splitext(file.filename)[1]

    with tempfile.NamedTemporaryFile(delete=False, suffix=suffix) as tmp:
        content = await file.read()
        tmp.write(content)
        tmp_path = tmp.name

    s3_key = f"uploads/{file.filename}"
    image_url = upload_file(tmp_path, s3_key)
    os.unlink(tmp_path)

    return JSONResponse({"url": image_url, "key": s3_key})
```

---

## 따라 하기 실습

### 실습 1 — 버킷 만들고 HTML 파일 올리기

1. `.env` 파일을 만들고 AWS 자격증명을 입력합니다.
2. `setup_bucket.py`를 실행해 버킷을 생성합니다.
3. 아래 내용으로 `static/index.html`을 만든 뒤 업로드합니다.

```html
<!-- static/index.html -->
<!DOCTYPE html>
<html lang="ko">
<head><meta charset="UTF-8"><title>내 첫 S3 페이지</title></head>
<body>
  <h1>S3에서 서비스되는 페이지입니다!</h1>
  <img src="images/logo.png" alt="로고">
</body>
</html>
```

```python
# 실습 1 실행
from uploader import upload_file
upload_file("static/index.html", "index.html")
# 브라우저에서 확인:
# https://<버킷이름>.s3.ap-northeast-2.amazonaws.com/index.html
```

---

### 실습 2 — 이미지 폴더 전체 업로드 자동화

프로젝트에 `static/images/` 폴더를 만들고 PNG 파일 2~3개를 넣은 뒤,
`upload_directory`로 한 번에 올려봅니다.

```python
# 실습 2 실행
from uploader import upload_directory

urls = upload_directory("static/images/", s3_prefix="images/")
for url in urls:
    print(url)
# 출력 예시:
# 업로드 완료: https://my-webapp-....amazonaws.com/images/logo.png
# 업로드 완료: https://my-webapp-....amazonaws.com/images/banner.png
```

---

### 실습 3 — Flask 앱에서 업로드 폼 연동

실습 1~2에서 만든 업로더를 Flask 앱에 붙여봅니다.

```bash
# 패키지 설치
pip install flask python-dotenv boto3

# 서버 실행
flask --app app_flask run --debug
```

브라우저에서 `http://localhost:5000/upload` 에 접속해 이미지를 올리고,
S3 URL로 리디렉션되는지 확인합니다.

---

## 자주 하는 실수

| 실수 | 실제 에러 메시지 | 해결 방법 |
|------|-----------------|-----------|
| 자격증명을 코드에 직접 작성 | `botocore.exceptions.NoCredentialsError` | `.env` 파일 + `python-dotenv` 사용 |
| 서울 리전에서 `LocationConstraint` 누락 | `IllegalLocationConstraintException` | `CreateBucketConfiguration={"LocationConstraint": "ap-northeast-2"}` 추가 |
| 버킷 정책 없이 파일 접근 | `403 Forbidden` | 버킷 퍼블릭 정책 또는 Presigned URL 사용 |
| Content-Type 미설정으로 이미지가 다운로드됨 | 브라우저가 이미지 표시 대신 파일 다운로드 | `ExtraArgs={"ContentType": "image/png"}` 지정 |
| 버킷 이름에 대문자 또는 공백 사용 | `InvalidBucketName` | 소문자, 숫자, 하이픈만 사용 |
| `.env` 파일을 Git에 커밋 | 자격증명 유출 | `.gitignore`에 `.env` 추가 |
| 리전을 `us-east-1`로 잘못 지정 | 업로드는 되지만 URL이 다른 리전을 가리킴 | `AWS_DEFAULT_REGION=ap-northeast-2` 확인 |

---

## 확인 체크리스트

- [ ] `.env` 파일이 `.gitignore`에 추가되어 있다
- [ ] boto3가 설치되어 있고 `import boto3`가 에러 없이 실행된다
- [ ] S3 버킷이 생성되었고 이름이 전 세계에서 유일하다
- [ ] 버킷의 퍼블릭 액세스 차단이 해제되어 있다
- [ ] 버킷 정책에 `s3:GetObject` 허용 규칙이 있다
- [ ] 파일 업로드 시 `ContentType`을 명시적으로 지정한다
- [ ] 업로드된 파일을 브라우저 URL로 직접 접근해서 확인했다
- [ ] Flask 또는 FastAPI 앱에서 S3 URL을 HTML 템플릿에 전달한다
- [ ] 임시 파일 업로드 후 `os.unlink()`로 로컬에서 삭제한다

---

## 한 번 더 생각해 보기

1. 사용자가 올린 파일을 바로 퍼블릭으로 공개하면 어떤 보안 문제가 생길 수 있을까요? 퍼블릭 정책 대신 어떤 방법을 쓸 수 있을까요?

2. 이미지 100개를 매번 수동으로 업로드하면 비효율적입니다. 이 장에서 만든 `upload_directory` 함수를 GitHub Actions CI/CD 파이프라인에 연결하려면 어떻게 해야 할까요?

3. S3에 올린 파일은 삭제하기 전까지 요금이 청구됩니다. 사용자가 프로필 사진을 교체할 때 기존 사진을 자동으로 삭제하려면 코드를 어떻게 수정해야 할까요?

---

## 다음 장

다음 장에서는 S3와 CloudFront를 연결해 전 세계 어디서나 빠르게 파일을 제공하는 CDN 설정 방법을 배웁니다.