# Softbank 2025 Hackathon - Infrastructure Template

AWS 인프라를 Terraform으로 구성하는 해커톤 템플릿입니다.

## 현재 배포 상태

배포 후 `terraform output` 명령어로 실제 값을 확인하세요.

```bash
# 전체 출력값 확인
terraform output

# 개별 확인
terraform output <OUTPUT_NAME>
```

### 주요 Output 항목

| Output | 설명 |
|--------|------|
| `cloudfront_domain_name` | CloudFront 배포 도메인 |
| `cloudfront_id` | CloudFront Distribution ID |
| `alb_dns_name` | ALB DNS 주소 |
| `alb_arn` | ALB ARN |
| `bastion_public_ip` | Bastion Host Public IP |
| `bastion_ssh_command` | Bastion SSH 접속 명령어 |
| `ecr_repositories` | ECR 레포지토리 URL (backend/frontend) |
| `vpc_id` | VPC ID |
| `public_subnet_ids` | Public Subnet ID 목록 |
| `private_app_subnet_ids` | Private App Subnet ID 목록 |
| `private_db_subnet_ids` | Private DB Subnet ID 목록 |
| `alb_security_group_id` | ALB 보안 그룹 ID |
| `ecs_tasks_security_group_id` | ECS Tasks 보안 그룹 ID |
| `bastion_security_group_id` | Bastion 보안 그룹 ID |
| `waf_web_acl_arn` | WAF Web ACL ARN |

## 사전 준비 (필수)

**Terraform 실행 전에 [PRE-REQUIREMENTS.md](./PRE-REQUIREMENTS.md)를 먼저 확인하세요.**

필수 사전 작업:
- S3 버킷 생성 (Terraform State 저장용)
- DynamoDB 테이블 생성 (State Locking용)
- EC2 Key Pair 생성 (Bastion 사용시)
- ACM 인증서 생성 (CloudFront 사용시 - us-east-1 리전)

## 아키텍처

```
                         Internet
                            │
                   ┌────────▼────────┐
                   │   CloudFront    │ *.eunha.icu
                   │      + WAF      │
                   └────────┬────────┘
                            │
                   ┌────────▼────────┐
                   │       ALB       │
                   └────────┬────────┘
                            │
         ┌──────────────────┼──────────────────┐
         │                  │                  │
    ┌────▼────┐       ┌─────▼─────┐      ┌─────▼─────┐
    │ Backend │       │ Frontend  │      │  Bastion  │
    │  (ECS)  │       │   (ECS)   │      │   (EC2)   │
    └─────────┘       └───────────┘      └───────────┘

VPC: 10.180.0.0/20
├── Public Subnets (2 AZ: ap-northeast-2a, 2c)
├── Private App Subnets (2 AZ)
└── Private DB Subnets (2 AZ)
```

## 모듈 구성

| 모듈 | 설명 | 상태 |
|------|------|------|
| vpc | VPC, 서브넷, NAT Gateway, 라우팅 | 배포됨 |
| security-groups | ALB, ECS, Bastion 보안그룹 | 배포됨 |
| ecr | Docker 이미지 레지스트리 | 배포됨 |
| alb | Application Load Balancer | 배포됨 |
| cloudfront | CDN 배포 (*.eunha.icu) | 배포됨 |
| waf | Web Application Firewall | 배포됨 |
| bastion | Bastion Host (점프 서버) | 배포됨 |

## 빠른 시작

### 1. 사전 준비 완료 확인

[PRE-REQUIREMENTS.md](./PRE-REQUIREMENTS.md) 참조

### 2. 배포

```bash
# 초기화
terraform init

# 계획 확인
terraform plan

# 배포
terraform apply
```

## 주요 변수

### 프로젝트 설정
| 변수 | 설명 | 기본값 |
|------|------|--------|
| `project_name` | 프로젝트 이름 (리소스 접두사) | `blue-final` |
| `environment` | 환경 (dev/staging/prod) | `dev` |

### VPC 설정
| 변수 | 설명 | 기본값 |
|------|------|--------|
| `vpc_cidr` | VPC CIDR | `10.180.0.0/20` |
| `availability_zones` | 가용 영역 | `["ap-northeast-2a", "ap-northeast-2c"]` |

### 선택적 모듈
| 변수 | 설명 | 기본값 |
|------|------|--------|
| `create_cloudfront` | CloudFront 생성 여부 | `true` |
| `create_waf` | WAF 생성 여부 | `true` |
| `create_bastion` | Bastion Host 생성 여부 | `true` |

### Bastion 설정
| 변수 | 설명 | 기본값 |
|------|------|--------|
| `bastion_key_name` | EC2 Key Pair 이름 | `blue-key` |
| `bastion_instance_type` | 인스턴스 타입 | `t3.micro` |

## 출력값

```bash
# 주요 출력값 확인
terraform output

# VPC
terraform output vpc_id
terraform output public_subnet_ids

# ALB
terraform output alb_arn
terraform output alb_dns_name

# ECR
terraform output ecr_repositories

# CloudFront
terraform output cloudfront_domain_name

# Bastion
terraform output bastion_public_ip
terraform output bastion_ssh_command
```

## 디렉토리 구조

```
.
├── README.md            # 이 파일
├── PRE-REQUIREMENTS.md  # 사전 준비 사항
├── main.tf              # 메인 모듈 구성
├── variables.tf         # 변수 정의
├── outputs.tf           # 출력값 정의
├── providers.tf         # AWS Provider 설정
├── backend.tf           # Terraform 백엔드 설정
└── modules/
    ├── vpc/             # VPC, 서브넷, NAT
    ├── security-groups/ # 보안그룹
    ├── ecr/             # ECR 레지스트리
    ├── alb/             # ALB
    ├── cloudfront/      # CDN
    ├── waf/             # WAF
    └── bastion/         # Bastion Host
```

## 정리

```bash
terraform destroy
```

## 라이선스

MIT
