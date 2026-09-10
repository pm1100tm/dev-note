# CI/CD

CI/CD는 코드 변경을 빌드·테스트·배포로 연결하는 자동화 흐름이다. 파이프라인에는 비밀값을 직접 넣지 않고, 배포 대상·롤백 절차·승인 조건을 명확히 정의해야 한다.

## 문서

* [GitHub Actions 기본](cicd_github_action.md)
* [GitHub Actions로 FastAPI 배포](cicd_github_action_fast_api.md)
* [Jenkins, Docker, AWS EC2 배포](../../08_cicd/cicd_jenkins_docker_aws_ec2.md)
* [EC2의 Jenkins 성능 이슈](cicd_jenkins_on_ec2_performance_issue.md)

## 운영 전 확인

* 배포용 자격 증명은 GitHub Secrets, Jenkins Credentials, 클라우드 IAM으로 관리한다.
* 이미지 태그는 재현 가능한 버전 또는 commit SHA를 사용한다.
* health check, 로그 확인, 롤백 기준을 파이프라인 실행 전 준비한다.
