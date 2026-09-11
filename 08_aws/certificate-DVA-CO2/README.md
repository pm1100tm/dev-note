# AWS Developer Associate 학습 노트

AWS Certified Developer – Associate 범위의 핵심 서비스를 주제별로 정리한다. 이 문서는
학습용 노트이며, 시험 범위와 서비스 동작은 AWS 공식 문서의 최신 버전을 기준으로 확인한다.

## 학습 범위

- [기본 개념: 리전, AZ, IAM, AMI](00_BASIC/001_지역_및_AZ.md)
- [EC2](01_EC2/003_EC2_01.md)
- [EBS](02_EBS/004_EBS_A.md)
- [EFS](03_EFS/004_EFS_A.md)
- [RDS와 캐시](04_RDS/001_RDS_Overview.md)
- [Route 53](05_ROUTE53/002_Route53_Overview.md)
- [VPC](06_VPC/001_VPC_overview.md)
- [S3](07_S3/001_S3_overview.md)
- [CloudFront](08_CLOUD_FRONT/01_cf_overview.md)
- [Elastic Beanstalk](10_ELASTIC_BEANSTARK/000_Elastic_Beanstalk_overview.md)

## 제외 범위

이 정리 작업에서는 `09_ECS`, `lambda`, `ELB_ASG`, `sqs` 하위 문서를 변경하지 않는다.

## 학습 원칙

- 서비스의 책임 범위, 보안 경계, 비용 모델, 장애 복구 전략을 함께 비교한다.
- 콘솔 실습은 최소 권한 IAM 사용자 또는 역할로 수행하고, 종료 시 리소스 삭제를 확인한다.
