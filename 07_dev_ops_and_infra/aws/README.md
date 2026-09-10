# AWS

AWS 서비스의 기본 개념, 실습, 운영 이슈를 정리한다. 서비스 설정은 계정·리전·VPC·IAM 정책에
따라 달라지므로, 콘솔에서 실행하기 전 비용·권한·삭제 영향과 공식 문서를 함께 확인한다.

## 기초와 인프라 실습

- [AWS 인증서](000_aws_certificate.md)
- [AWS 소개](001. Intro.md)
- [네트워크 구성](002. 네트워크 구성하기.md)
- [웹 서버 생성](003. Webserver 생성하기.md)
- [로드 밸런서 구성](004. 로드벨런서 구성하기.md)
- [인스턴스 타입 변경](005. 인스턴스 타입 변경하기.md)
- [Elastic IP 설정](006_elastic_ip_setting.md)
- [AWS CLI 설정](007_aws_cli_setting.md)

## 분석과 생성형 AI

- [Athena 소개](athena/01_Athena란_무엇인가.md)
- [Athena 실습](athena/02_Athena실습.md)
- [Athena 문제 해결](athena/03_Athena_Trouble_Shooting.md)
- [Bedrock 소개](bedrock/01_BedRock이란_무엇인가.md)
- [Bedrock 설정](bedrock/02_BedRock설정.md)
- [Bedrock 실습](bedrock/03_BedRock실습.md)
- [Bedrock 예외 처리](bedrock/04_예외처리_1.md)

## 운영 원칙

- 루트 계정 대신 최소 권한 IAM 역할을 사용하고, 액세스 키는 저장소와 문서에 기록하지 않는다.
- 리소스 생성 전 리전·태그·비용 알림을 확인하고, 실습 종료 후 리소스를 삭제한다.
- 고가용성과 보안 설정은 서비스별 기본값에 의존하지 말고 요구사항과 장애 시나리오로 검증한다.
