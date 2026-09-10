# Google Cloud Platform

GCP CLI, Compute Engine, Pub/Sub, Cloud Logging, 디스크 운영 실습을 정리한다. 명령 실행 전에는
활성 프로젝트·리전·계정·IAM 권한을 확인하고, 실습용 리소스는 비용이 발생하지 않도록 정리한다.

## 문서

- [GCE에 Docker 설정](gce_docker_settting.md)
- [gcloud CLI 연결](gcp_cli_connect.md)
- [Pub/Sub CLI](gcp_cli_pubsub.md)
- [Cloud Logging Log Writer 오류](gcp_log_writer_오류_100%.md)
- [GCE에 SSD 디스크 마운트](gcp_mount_ssd_disk_to_gce.md)
- [GCE 부팅 디스크 용량 늘리기](인스턴스_용량_늘리기.md)

## 운영 원칙

- 서비스 계정에는 필요한 역할만 부여하고, 키 파일을 저장소에 넣지 않는다.
- 디스크 크기 증설은 일반적으로 되돌릴 수 없으므로 스냅샷·백업·파일 시스템 확장 절차를 확인한다.
- 컨테이너 데이터는 디스크 마운트와 백업 정책을 함께 설계한다.
