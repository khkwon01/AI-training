## 이 장에서 배우는 것

- AWS WAF(Web Application Firewall)가 무엇인지, 왜 API 보호에 필요한지 이해한다
- WAF의 핵심 구성 요소(Web ACL, Rule Group, Rule)를 구분할 수 있다
- AWS 콘솔과 CLI/Terraform으로 기본 WAF 규칙을 만들고 API Gateway에 연결한다
- AWS 관리형 규칙(Managed Rule Groups)을 활용해 빠르게 기본 보호를 적용한다
- WAF 로그를 CloudWatch/S3로 남기고 차단 결과를 확인한다

---

## 먼저 쉬운 설명

여러분이 만든 API는 인터넷에 공개되는 순간부터 다양한 공격에 노출됩니다.  
SQL 인젝션, 봇(Bot) 대량 요청, 알려진 악성 IP 등 — 이 모든 위협을 애플리케이션 코드 안에서만 막으려면 한계가 있습니다.

**AWS WAF**는 요청이 여러분의 서버(Lambda, EC2, API Gateway)에 도달하기 **전에** 가로채서 규칙에 맞지 않는 트래픽을 걸러내는 일종의 **문지기**입니다.

```
인터넷 사용자
    │
    ▼
[AWS WAF] ← 여기서 나쁜 요청을 차단
    │
    ▼
API Gateway / ALB / CloudFront
    │
    ▼
Lambda / EC2 (여러분의 API)
```

코드 한 줄 없이도 SQL 인젝션 패턴이나 봇 트래픽의 대부분을 막을 수 있습니다.  
이 장에서는 그 기초부터 실제 적용까지 단계별로 익힙니다.

---

## 1. WAF 핵심 개념 이해하기

AWS WAF는 세 가지 계층으로 구성됩니다.

```
Web ACL (출입문 전체)
 └─ Rule Group (규칙 묶음 — 예: "SQL 인젝션 방어 묶음")
     └─ Rule (개별 규칙 — 예: "URI에 'DROP TABLE' 포함 시 차단")
```

| 용어 | 역할 | 비유 |
|------|------|------|
| **Web ACL** | WAF의 최상위 단위. 리소스(API GW 등)에 연결 | 건물 입구 전체 보안 정책 |
| **Rule Group** | 여러 Rule을 묶은 재사용 가능한 묶음 | 보안 규정집 한 챕터 |
| **Rule** | 조건(Statement) + 행동(Action)으로 구성 | 개별 보안 규정 하나 |
| **Statement** | IP, URI, 헤더 등 매칭 조건 | "이런 사람은…" |
| **Action** | Allow / Block / Count / CAPTCHA | "…들여보낸다 / 막는다" |

### 중요: Regional vs CloudFront

```
# API Gateway, ALB, App Runner → scope = REGIONAL
# CloudFront → scope = CLOUDFRONT (반드시 us-east-1 리전에서 생성)
```

실수로 리전을 잘못 선택하면 연결이 안 됩니다. API Gateway를 사용할 때는 **REGIONAL**을 사용합니다.

---

## 2. AWS 관리형 규칙(Managed Rule Groups) 활용하기

처음부터 모든 규칙을 직접 만들 필요 없습니다.  
AWS가 미리 만들어 놓은 **관리형 규칙 그룹**을 사용하면 됩니다.

```python
# aws_waf_managed_rules.py
# boto3로 사용 가능한 AWS 관리형 규칙 목록 확인하기

import boto3

client = boto3.client('wafv2', region_name='ap-northeast-2')

response = client.list_available_managed_rule_groups(
    Scope='REGIONAL'
)

for group in response['ManagedRuleGroups']:
    print(f"이름: {group['Name']}")
    print(f"벤더: {group['VendorName']}")
    print(f"설명: {group.get('Description', '없음')}")
    print("---")
```

**자주 쓰는 AWS 관리형 규칙 그룹:**

| 규칙 그룹 이름 | 막아주는 것 |
|---|---|
| `AWSManagedRulesCommonRuleSet` | OWASP Top 10 기본 공격 |
| `AWSManagedRulesSQLiRuleSet` | SQL 인젝션 |
| `AWSManagedRulesKnownBadInputsRuleSet` | Log4Shell 등 알려진 악성 입력 |
| `AWSManagedRulesAmazonIpReputationList` | 악성 IP 목록 |
| `AWSManagedRulesBotControlRuleSet` | 봇 트래픽 (추가 요금 있음) |

---

## 3. Terraform으로 Web ACL 만들고 API Gateway에 연결하기

실무에서는 콘솔보다 코드(IaC)로 관리하는 것이 좋습니다.

```hcl
# waf.tf — API Gateway 보호용 기본 Web ACL

resource "aws_wafv2_web_acl" "api_waf" {
  name        = "api-protection-waf"
  description = "API Gateway를 보호하는 기본 WAF"
  scope       = "REGIONAL"  # API Gateway는 반드시 REGIONAL

  default_action {
    allow {}  # 기본은 허용, 규칙에 걸리면 차단
  }

  # 규칙 1: AWS 공통 규칙 (OWASP Top 10)
  rule {
    name     = "AWS-CommonRules"
    priority = 1  # 숫자가 낮을수록 먼저 평가됨

    override_action {
      none {}  # 관리형 규칙의 기본 action을 그대로 사용
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesCommonRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "CommonRules"
      sampled_requests_enabled   = true
    }
  }

  # 규칙 2: SQL 인젝션 방어
  rule {
    name     = "AWS-SQLiRules"
    priority = 2

    override_action {
      none {}
    }

    statement {
      managed_rule_group_statement {
        name        = "AWSManagedRulesSQLiRuleSet"
        vendor_name = "AWS"
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "SQLiRules"
      sampled_requests_enabled   = true
    }
  }

  # 규칙 3: 특정 IP 직접 차단 (예: 공격자 IP)
  rule {
    name     = "BlockBadIPs"
    priority = 0  # 가장 먼저 평가

    action {
      block {}
    }

    statement {
      ip_set_reference_statement {
        arn = aws_wafv2_ip_set.blocked_ips.arn
      }
    }

    visibility_config {
      cloudwatch_metrics_enabled = true
      metric_name                = "BlockedIPs"
      sampled_requests_enabled   = true
    }
  }

  visibility_config {
    cloudwatch_metrics_enabled = true
    metric_name                = "ApiWAF"
    sampled_requests_enabled   = true
  }

  tags = {
    Environment = "production"
    Purpose     = "api-protection"
  }
}

# 차단할 IP 목록
resource "aws_wafv2_ip_set" "blocked_ips" {
  name               = "blocked-ip-list"
  scope              = "REGIONAL"
  ip_address_version = "IPV4"

  addresses = [
    "192.0.2.1/32",   # 예시: 특정 공격자 IP
    "198.51.100.0/24" # 예시: 악성 IP 대역
  ]
}

# Web ACL을 API Gateway Stage에 연결
resource "aws_wafv2_web_acl_association" "api_gateway" {
  resource_arn = aws_api_gateway_stage.main.arn
  web_acl_arn  = aws_wafv2_web_acl.api_waf.arn
}
```

---

## 4. WAF 로그 활성화하기

WAF가 어떤 요청을 차단했는지 확인하려면 로그를 켜야 합니다.

```hcl
# waf_logging.tf — WAF 로그를 S3로 전송

resource "aws_s3_bucket" "waf_logs" {
  # 주의: WAF 로그 버킷 이름은 반드시 "aws-waf-logs-"로 시작해야 함
  bucket = "aws-waf-logs-my-api-production"
}

resource "aws_wafv2_web_acl_logging_configuration" "api_waf_logs" {
  log_destination_configs = [aws_s3_bucket.waf_logs.arn]
  resource_arn            = aws_wafv2_web_acl.api_waf.arn

  # 민감한 헤더(Authorization 등)는 로그에서 제외
  redacted_fields {
    single_header {
      name = "authorization"
    }
  }
}
```

```python
# check_waf_logs.py — CloudWatch에서 WAF 차단 로그 조회

import boto3
from datetime import datetime, timedelta

client = boto3.client('logs', region_name='ap-northeast-2')

# WAF 로그 그룹 이름 형식: aws-waf-logs-{이름}
LOG_GROUP = 'aws-waf-logs-my-api-production'

response = client.filter_log_events(
    logGroupName=LOG_GROUP,
    startTime=int((datetime.now() - timedelta(hours=1)).timestamp() * 1000),
    filterPattern='{ $.action = "BLOCK" }',  # 차단된 요청만
    limit=20
)

print(f"지난 1시간 차단된 요청 수: {len(response['events'])}건")
for event in response['events']:
    print(event['message'][:200])  # 처음 200자만 출력
```

---

## 따라 하기 실습

### 실습 1: boto3로 Web ACL 만들기

파일명: `create_basic_waf.py`

```python
# create_basic_waf.py
# boto3로 API Gateway 보호용 최소한의 Web ACL을 생성합니다.

import boto3
import json

client = boto3.client('wafv2', region_name='ap-northeast-2')

def create_web_acl():
    response = client.create_web_acl(
        Name='my-first-api-waf',
        Scope='REGIONAL',
        DefaultAction={'Allow': {}},
        Rules=[
            {
                'Name': 'CommonRuleSet',
                'Priority': 1,
                'OverrideAction': {'None': {}},
                'Statement': {
                    'ManagedRuleGroupStatement': {
                        'VendorName': 'AWS',
                        'Name': 'AWSManagedRulesCommonRuleSet'
                    }
                },
                'VisibilityConfig': {
                    'SampledRequestsEnabled': True,
                    'CloudWatchMetricsEnabled': True,
                    'MetricName': 'CommonRuleSet'
                }
            }
        ],
        VisibilityConfig={
            'SampledRequestsEnabled': True,
            'CloudWatchMetricsEnabled': True,
            'MetricName': 'MyFirstApiWaf'
        }
    )

    arn = response['Summary']['ARN']
    print(f"Web ACL 생성 완료!")
    print(f"ARN: {arn}")
    return arn

if __name__ == '__main__':
    arn = create_web_acl()
    # ARN을 저장해 두세요. 실습 2에서 사용합니다.
    with open('waf_arn.txt', 'w') as f:
        f.write(arn)
    print("ARN을 waf_arn.txt에 저장했습니다.")
```

실행:
```bash
python create_basic_waf.py
```

---

### 실습 2: Web ACL을 API Gateway에 연결하기

파일명: `attach_waf_to_api.py`

```python
# attach_waf_to_api.py
# 실습 1에서 만든 WAF를 기존 API Gateway Stage에 연결합니다.

import boto3

# 실습 1에서 저장한 WAF ARN 읽기
with open('waf_arn.txt') as f:
    waf_arn = f.read().strip()

# API Gateway Stage의 ARN 구성
# 형식: arn:aws:apigateway:{region}::/restapis/{api-id}/stages/{stage-name}
REGION = 'ap-northeast-2'
API_ID = 'your-api-id-here'    # AWS 콘솔 > API Gateway에서 확인
STAGE_NAME = 'prod'

api_stage_arn = (
    f"arn:aws:apigateway:{REGION}::/restapis/{API_ID}/stages/{STAGE_NAME}"
)

client = boto3.client('wafv2', region_name=REGION)

try:
    client.associate_web_acl(
        WebACLArn=waf_arn,
        ResourceArn=api_stage_arn
    )
    print(f"연결 완료!")
    print(f"WAF: {waf_arn}")
    print(f"API Gateway Stage: {api_stage_arn}")
except client.exceptions.WAFNonexistentItemException:
    print("오류: API Gateway Stage를 찾을 수 없습니다. API_ID와 STAGE_NAME을 확인하세요.")
```

---

### 실습 3: WAF 규칙을 Count 모드로 테스트하기

바로 차단하기 전에 **Count(카운트) 모드**로 먼저 테스트하는 것이 안전합니다.

파일명: `waf_count_mode_test.py`

```python
# waf_count_mode_test.py
# 규칙을 Block 대신 Count 모드로 설정해 오탐(false positive)을 먼저 확인합니다.

import boto3

client = boto3.client('wafv2', region_name='ap-northeast-2')

# 기존 Web ACL 정보 가져오기
with open('waf_arn.txt') as f:
    waf_arn = f.read().strip()

waf_name = 'my-first-api-waf'

# 현재 Web ACL 조회 (업데이트에 필요한 LockToken 확인)
acl = client.get_web_acl(
    Name=waf_name,
    Scope='REGIONAL',
    Id=waf_arn.split('/')[-2]  # ARN에서 ID 추출
)

lock_token = acl['LockToken']
current_rules = acl['WebACL']['Rules']

# 모든 규칙의 OverrideAction을 Count로 변경
count_mode_rules = []
for rule in current_rules:
    new_rule = dict(rule)
    # OverrideAction이 있는 규칙(관리형 규칙)만 Count로 변경
    if 'OverrideAction' in new_rule:
        new_rule['OverrideAction'] = {'Count': {}}
    count_mode_rules.append(new_rule)

client.update_web_acl(
    Name=waf_name,
    Scope='REGIONAL',
    Id=waf_arn.split('/')[-2],
    DefaultAction={'Allow': {}},
    Rules=count_mode_rules,
    VisibilityConfig=acl['WebACL']['VisibilityConfig'],
    LockToken=lock_token
)

print("Count 모드로 변경 완료!")
print("이제 CloudWatch 메트릭에서 카운트를 확인하며 오탐 여부를 검토하세요.")
print("문제 없으면 OverrideAction을 다시 None으로 바꿔 실제 차단을 활성화하세요.")
```

---

## 자주 하는 실수

| 실수 | 실제 오류 메시지 | 원인 | 해결 방법 |
|------|-----------------|------|-----------|
| Scope를 잘못 지정 | `WAFInvalidParameterException: Error reason: INVALID_SCOPE` | API Gateway에 CLOUDFRONT scope Web ACL 연결 시도 | API Gateway는 반드시 `REGIONAL` 사용 |
| S3 로그 버킷 이름 규칙 위반 | `WAFLogDestinationPermissionIssueException` | 버킷 이름이 `aws-waf-logs-`로 시작하지 않음 | 버킷 이름을 `aws-waf-logs-{이름}` 형식으로 변경 |
| LockToken 없이 업데이트 | `WAFOptimisticLockException: AWS WAF couldn't save your changes` | Web ACL 업데이트 시 LockToken 미포함 | `get_web_acl()`로 최신 LockToken 먼저 조회 후 사용 |
| Priority 중복 | `WAFDuplicateItemException` | 두 Rule의 Priority 값이 같음 | 각 Rule마다 고유한 Priority 값 부여 |
| CloudFront WAF를 다른 리전에서 생성 | `WAFNonexistentItemException` 또는 연결 실패 | CloudFront 전용 WAF는 `us-east-1`에서만 생성 가능 | CloudFront를 쓸 때는 리전을 `us-east-1`로 설정 |
| 바로 Block 모드로 배포 | 정상 트래픽 차단 (오류 없음, 하지만 서비스 장애) | 오탐(false positive) 미확인 상태에서 배포 | Count 모드로 먼저 배포 후 로그 확인 → Block으로 전환 |

---

## 확인 체크리스트

- [ ] Web ACL, Rule Group, Rule의 계층 구조를 말로 설명할 수 있다
- [ ] API Gateway에 연결할 때 Scope를 `REGIONAL`로 설정해야 함을 안다
- [ ] `AWSManagedRulesCommonRuleSet`이 무엇을 막아주는지 설명할 수 있다
- [ ] WAF 로그 S3 버킷 이름이 `aws-waf-logs-`로 시작해야 함을 기억한다
- [ ] LockToken이 왜 필요한지 이해했다
- [ ] 새 규칙을 배포할 때 Count 모드 → 검증 → Block 모드 순서를 지킬 수 있다
- [ ] `create_basic_waf.py`를 실행해 실제로 Web ACL을 만들어 봤다
- [ ] WAF를 API Gateway Stage에 연결하는 코드를 작성했다

---

## 한 번 더 생각해 보기

1. **우선순위(Priority)가 왜 중요할까요?**  
   IP 차단 규칙(Priority 0)을 SQL 인젝션 규칙(Priority 2)보다 먼저 평가하면 어떤 이점이 있을까요? 반대로 순서가 바뀌면 어떤 문제가 생길 수 있을까요?

2. **Count 모드와 Block 모드를 왜 나눠서 사용할까요?**  
   운영 중인 API에 새 WAF 규칙을 바로 Block 모드로 배포했다가 정상 사용자가 차단된다면 어떤 일이 벌어질까요? Count 모드 → 검증 → Block 전환 순서가 실무에서 어떤 안전장치가 되는지 생각해 보세요.

3. **WAF만 있으면 API 보안은 완벽할까요?**  
   WAF가 막을 수 없는 공격 유형(예: 인증 우회, 비즈니스 로직 공격)을 생각해 보고, WAF 외에 어떤 보안 레이어가 추가로 필요한지 적어 보세요.

---

## 다음 장

다음 장에서는 AWS WAF Rate Limiting(속도 제한) 규칙으로 API 남용과 DDoS 공격을 방어하는 방법을 배웁니다.