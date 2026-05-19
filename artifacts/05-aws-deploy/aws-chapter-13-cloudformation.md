## 이 장에서 배우는 것

- AWS CloudFormation이 무엇인지, 왜 사용하는지 이해한다
- YAML 형식으로 CloudFormation 템플릿을 작성하는 방법을 배운다
- 스택(Stack)을 생성하고 업데이트하고 삭제하는 방법을 익힌다
- S3 버킷과 EC2 인스턴스를 코드로 정의하는 기본 예제를 따라 할 수 있다
- 파라미터(Parameters)와 출력(Outputs)을 활용해 재사용 가능한 템플릿을 만든다
- 흔히 발생하는 배포 오류를 읽고 스스로 고칠 수 있다

---

## 먼저 쉬운 설명

서버를 만들고, 데이터베이스를 연결하고, 네트워크를 설정하는 일을 AWS 콘솔에서 마우스로 클릭해서 했다고 상상해 보세요. 이 작업을 열 번, 백 번 반복해야 한다면 어떨까요? 매번 실수가 생기고, 다른 환경(개발/운영)에서 설정이 달라지는 문제가 생깁니다.

**CloudFormation은 인프라를 코드로 적는 도구입니다.**

마치 요리 레시피처럼, YAML 파일 하나에 "S3 버킷 하나 만들고, EC2 서버 하나 만들고, 이렇게 연결해"라고 써 두면 AWS가 그대로 만들어 줍니다. 이 파일을 Git에 저장해서 버전 관리도 되고, 같은 환경을 여러 개 복사하는 것도 쉬워집니다.

**핵심 개념 세 가지만 기억하세요:**
- **템플릿(Template)**: 인프라를 정의하는 YAML(또는 JSON) 파일
- **스택(Stack)**: 템플릿으로 실제로 생성된 AWS 리소스 묶음
- **리소스(Resource)**: S3, EC2, RDS 등 실제로 만들어지는 AWS 서비스 하나하나

---

## 1. CloudFormation 템플릿의 기본 구조

CloudFormation 템플릿은 정해진 섹션으로 구성됩니다. 가장 중요한 섹션은 `Resources`이며, 이것은 필수입니다.

```yaml
# my-first-template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: '나의 첫 번째 CloudFormation 템플릿'

Parameters:
  ProjectName:
    Type: String
    Default: my-project
    Description: '프로젝트 이름 (영문 소문자, 숫자, 하이픈만 사용)'

Resources:
  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub '${ProjectName}-storage-${AWS::AccountId}'
      VersioningConfiguration:
        Status: Enabled

Outputs:
  BucketName:
    Description: '생성된 S3 버킷 이름'
    Value: !Ref MyS3Bucket
    Export:
      Name: !Sub '${ProjectName}-bucket-name'
```

**각 섹션 설명:**

| 섹션 | 필수 여부 | 역할 |
|------|----------|------|
| `AWSTemplateFormatVersion` | 선택 | 템플릿 버전 (항상 `'2010-09-09'` 고정) |
| `Description` | 선택 | 템플릿 설명 |
| `Parameters` | 선택 | 스택 생성 시 입력받는 값 |
| `Resources` | **필수** | 실제로 만들 AWS 리소스 |
| `Outputs` | 선택 | 스택 생성 후 확인할 값 |

> **팁:** `!Sub`는 문자열 안에 변수를 삽입하는 함수입니다. `${ProjectName}`처럼 쓰면 파라미터 값으로 치환됩니다. `${AWS::AccountId}`는 현재 AWS 계정 번호를 자동으로 가져옵니다.

---

## 2. 리소스(Resource) 정의 방법

모든 리소스는 동일한 패턴으로 정의됩니다.

```yaml
# 리소스 정의 기본 패턴
Resources:
  리소스논리ID:          # 템플릿 안에서 이 리소스를 부르는 이름 (영문)
    Type: AWS::서비스::리소스타입
    Properties:
      속성이름: 속성값
```

**실제 예시 — EC2 인스턴스와 보안 그룹 함께 만들기:**

```yaml
# ec2-with-sg.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'EC2 인스턴스와 보안 그룹 생성 예제'

Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
    Description: 'EC2 인스턴스 타입'
  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: '접속에 사용할 키 페어 이름'

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: '웹 서버용 보안 그룹'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0  # 실제 운영에서는 내 IP만 허용할 것

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType          # !Ref 로 파라미터 값 참조
      ImageId: ami-0c9c942bd7bf113a2           # ap-northeast-2 (서울) Amazon Linux 2023
      KeyName: !Ref KeyPairName
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup          # !Ref 로 같은 템플릿 안 리소스 참조
      Tags:
        - Key: Name
          Value: my-web-server

Outputs:
  InstancePublicIP:
    Description: 'EC2 인스턴스 퍼블릭 IP'
    Value: !GetAtt WebServerInstance.PublicIp  # !GetAtt 로 리소스 속성 가져오기
```

**자주 쓰는 내장 함수:**

| 함수 | 사용법 | 역할 |
|------|--------|------|
| `!Ref` | `!Ref 리소스ID` | 리소스의 기본 식별자 또는 파라미터 값 참조 |
| `!GetAtt` | `!GetAtt 리소스ID.속성` | 리소스의 특정 속성 값 가져오기 |
| `!Sub` | `!Sub '문자열 ${변수}'` | 문자열 치환 |
| `!Join` | `!Join [구분자, [값1, 값2]]` | 문자열 합치기 |

---

## 3. 스택 생성·업데이트·삭제 (AWS CLI)

템플릿 파일을 작성했으면 실제로 AWS에 배포해야 합니다. AWS CLI를 사용합니다.

```bash
# 스택 생성
aws cloudformation create-stack \
  --stack-name my-first-stack \
  --template-body file://my-first-template.yaml \
  --parameters ParameterKey=ProjectName,ParameterValue=demo-app \
  --region ap-northeast-2

# 스택 생성 상태 확인 (완료될 때까지 기다림)
aws cloudformation wait stack-create-complete \
  --stack-name my-first-stack \
  --region ap-northeast-2

# 스택 현재 상태 확인
aws cloudformation describe-stacks \
  --stack-name my-first-stack \
  --region ap-northeast-2 \
  --query 'Stacks[0].StackStatus'

# 템플릿을 수정한 후 스택 업데이트
aws cloudformation update-stack \
  --stack-name my-first-stack \
  --template-body file://my-first-template.yaml \
  --parameters ParameterKey=ProjectName,ParameterValue=demo-app \
  --region ap-northeast-2

# 스택 삭제 (모든 리소스가 함께 삭제됨!)
aws cloudformation delete-stack \
  --stack-name my-first-stack \
  --region ap-northeast-2
```

**스택 상태 코드 읽는 법:**

| 상태 | 의미 |
|------|------|
| `CREATE_IN_PROGRESS` | 리소스 생성 중 |
| `CREATE_COMPLETE` | 생성 완료 |
| `CREATE_FAILED` | 생성 실패 |
| `UPDATE_COMPLETE` | 업데이트 완료 |
| `ROLLBACK_COMPLETE` | 오류로 인해 이전 상태로 롤백됨 |
| `DELETE_COMPLETE` | 삭제 완료 |

---

## 4. 변경 세트(Change Set)로 안전하게 업데이트하기

운영 환경에서 스택을 바로 업데이트하면 예상치 못한 리소스가 삭제될 수 있습니다. **변경 세트**를 사용하면 실제 적용 전에 무엇이 바뀌는지 미리 확인할 수 있습니다.

```bash
# 1단계: 변경 세트 생성 (실제로 변경하지 않음)
aws cloudformation create-change-set \
  --stack-name my-first-stack \
  --change-set-name preview-update-v2 \
  --template-body file://my-first-template.yaml \
  --parameters ParameterKey=ProjectName,ParameterValue=demo-app \
  --region ap-northeast-2

# 2단계: 변경 내용 확인
aws cloudformation describe-change-set \
  --stack-name my-first-stack \
  --change-set-name preview-update-v2 \
  --region ap-northeast-2 \
  --query 'Changes[*].ResourceChange.{Action:Action,Resource:ResourceType,ID:LogicalResourceId,Replace:Replacement}'

# 출력 예시:
# [
#   {
#     "Action": "Modify",
#     "Resource": "AWS::EC2::Instance",
#     "ID": "WebServerInstance",
#     "Replace": "false"       <- false면 교체 없이 수정, true면 리소스 삭제 후 재생성!
#   }
# ]

# 3단계: 확인 후 실제 적용
aws cloudformation execute-change-set \
  --stack-name my-first-stack \
  --change-set-name preview-update-v2 \
  --region ap-northeast-2
```

> **중요:** `Replacement: true`가 뜨면 해당 리소스가 **삭제되고 새로 만들어집니다.** EC2 인스턴스라면 IP가 바뀌고, RDS라면 데이터가 사라질 수 있으니 반드시 확인하세요.

---

## 따라 하기 실습

### 실습 1: S3 버킷 스택 만들고 확인하기

아래 파일을 `lab01-s3-bucket.yaml`로 저장하세요.

```yaml
# lab01-s3-bucket.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: '실습 1 - S3 버킷 생성'

Parameters:
  YourName:
    Type: String
    Description: '영문 소문자 이름 (예: gildong)'
    MinLength: 2
    MaxLength: 20

Resources:
  LabS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'cfn-lab-${YourName}-${AWS::AccountId}'
      Tags:
        - Key: CreatedBy
          Value: CloudFormation
        - Key: Lab
          Value: '1'

Outputs:
  BucketName:
    Value: !Ref LabS3Bucket
    Description: '생성된 버킷 이름'
  BucketArn:
    Value: !GetAtt LabS3Bucket.Arn
    Description: '버킷 ARN'
```

```bash
# 스택 생성 (YourName에 본인 이름 입력)
aws cloudformation create-stack \
  --stack-name cfn-lab01 \
  --template-body file://lab01-s3-bucket.yaml \
  --parameters ParameterKey=YourName,ParameterValue=gildong \
  --region ap-northeast-2

# 완료 대기
aws cloudformation wait stack-create-complete \
  --stack-name cfn-lab01 --region ap-northeast-2

# Outputs 확인
aws cloudformation describe-stacks \
  --stack-name cfn-lab01 --region ap-northeast-2 \
  --query 'Stacks[0].Outputs'
```

**확인:** AWS 콘솔 → S3 에서 `cfn-lab-gildong-123456789012` 버킷이 생성됐는지 확인합니다.

---

### 실습 2: 태그 추가 후 변경 세트로 업데이트하기

`lab01-s3-bucket.yaml` 파일에서 `Tags` 섹션에 태그를 하나 추가합니다.

```yaml
      Tags:
        - Key: CreatedBy
          Value: CloudFormation
        - Key: Lab
          Value: '1'
        - Key: Environment      # ← 이 줄 추가
          Value: Learning       # ← 이 줄 추가
```

```bash
# 변경 세트 생성
aws cloudformation create-change-set \
  --stack-name cfn-lab01 \
  --change-set-name add-env-tag \
  --template-body file://lab01-s3-bucket.yaml \
  --parameters ParameterKey=YourName,ParameterValue=gildong \
  --region ap-northeast-2

# 변경 내용 미리 보기
aws cloudformation describe-change-set \
  --stack-name cfn-lab01 \
  --change-set-name add-env-tag \
  --region ap-northeast-2 \
  --query 'Changes[*].ResourceChange.{Action:Action,Resource:ResourceType,Replace:Replacement}'

# 적용
aws cloudformation execute-change-set \
  --stack-name cfn-lab01 \
  --change-set-name add-env-tag \
  --region ap-northeast-2
```

---

### 실습 3: 스택 삭제하기

```bash
# 스택 삭제 (S3 버킷 안에 파일이 있으면 삭제 실패함 — 버킷을 먼저 비워야 함)
aws cloudformation delete-stack \
  --stack-name cfn-lab01 \
  --region ap-northeast-2

# 삭제 완료 대기
aws cloudformation wait stack-delete-complete \
  --stack-name cfn-lab01 --region ap-northeast-2

echo "스택 삭제 완료!"
```

**확인:** AWS 콘솔 → S3 에서 버킷이 사라졌는지 확인합니다.

---

## 자주 하는 실수

| 실수 | 오류 메시지 | 원인 | 해결 방법 |
|------|------------|------|-----------|
| 버킷 이름 중복 | `BucketAlreadyExists` 또는 `BucketAlreadyOwnedByYou` | S3 버킷 이름은 전 세계에서 고유해야 함 | `${AWS::AccountId}` 또는 날짜를 이름에 포함 |
| YAML 들여쓰기 오류 | `Template format error: YAML not well-formed` | 탭(Tab) 사용 또는 들여쓰기 칸 수 불일치 | 스페이스만 사용, 에디터에서 탭을 스페이스로 변환 설정 |
| 리소스 이름에 특수문자 | `Resource name must be alphanumeric` | 논리 ID에 하이픈(`-`) 등 사용 | 논리 ID는 영문+숫자만 사용 (예: `WebServer`, `MyS3Bucket`) |
| 스택 삭제 시 버킷 내 파일 존재 | `The bucket you tried to delete is not empty` | S3 버킷에 객체가 남아 있음 | 버킷 먼저 비우거나 `DeletionPolicy: Retain` 설정 |
| 잘못된 AMI ID | `InvalidAMIID.NotFound` | AMI ID가 해당 리전에 존재하지 않음 | 리전별 AMI ID 확인 (콘솔 → EC2 → AMI 카탈로그) |
| 키 페어 없음 | `InvalidKeyPair.NotFound` | 지정한 키 페어가 해당 리전에 없음 | 리전에서 키 페어 생성 후 정확한 이름 사용 |
| `!Ref` 대상 오타 | `Template contains errors: Unresolved resource dependencies` | 참조하는 리소스 논리 ID 철자 오류 | 논리 ID 대소문자까지 정확히 일치하는지 확인 |
| `UPDATE_ROLLBACK_FAILED` | 콘솔에서 빨간색 상태 표시 | 업데이트 중 오류 발생 후 롤백도 실패 | 콘솔 → 스택 → 이벤트 탭에서 구체적 오류 메시지 확인 |

---

## 확인 체크리스트

- [ ] CloudFormation 템플릿에서 `Resources` 섹션이 왜 필수인지 설명할 수 있다
- [ ] `!Ref`와 `!GetAtt`의 차이를 알고 상황에 맞게 사용할 수 있다
- [ ] `Parameters`를 사용해서 같은 템플릿으로 개발/운영 환경을 다르게 배포할 수 있다
- [ ] `aws cloudformation create-stack` 명령으로 스택을 생성할 수 있다
- [ ] 변경 세트(Change Set)를 만들고 `Replacement: true`가 무엇을 의미하는지 안다
- [ ] 스택 이벤트 탭에서 오류 메시지를 찾아 원인을 파악할 수 있다
- [ ] 스택을 삭제하면 그 안의 리소스도 함께 삭제된다는 것을 이해한다
- [ ] S3 버킷 이름이 전 세계에서 유일해야 하는 이유를 알고, `${AWS::AccountId}`로 중복을 피하는 방법을 쓸 수 있다

---

## 한 번 더 생각해 보기

1. **변경 세트 없이 바로 `update-stack`을 사용하면 어떤 위험이 있을까요?** 특히 `Replacement: true`인 리소스가 포함된 경우, 운영 데이터베이스에 어떤 일이 벌어질 수 있을지 생각해 보세요.

2. **템플릿을 Git으로 관리하면 어떤 이점이 있을까요?** "어제 배포한 것과 오늘 배포한 것이 다르다"는 사실을 어떻게 빠르게 확인할 수 있을지 떠올려 보세요.

3. **`Parameters`를 사용하면 템플릿 하나로 개발(`dev`)과 운영(`prod`) 환경을 모두 관리할 수 있습니다.** 환경마다 인스턴스 타입이나 버킷 이름이 달라야 한다면 파라미터를 어떻게 구성하면 좋을까요?

---

## 다음 장

다음 장에서는 CloudFormation **중첩 스택(Nested Stack)** 과 **크로스 스택 참조(Cross-Stack Reference)** 를 사용해서 대규모 인프라를 여러 파일로 나누어 체계적으로 관리하는 방법을 배웁니다.