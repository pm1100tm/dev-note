# 🚀 Connect to AWS RDS on Local Datagrip

자신의 로컬 PC(여기서는 MAC 기준)에서 AWS RDS 에 연결이 되지 않는 이슈가 발생하였습니다.
원인은 간단합니다. RDS 의 보안그룹 설정이 올바르게 되어있지 않았기 때문입니다.

## 분명 전에 로컬 PC의 Datagrip 에서 RDS 연결이 되었는데?

실습을 진행하는 동안 외부 장소의 IP만 연결해둔적이 있습니다. 즉, 집의 Wifi IP 가 RDS 의 보안그룹에
정상적으로 등록되지 않았던 것입니다.

그러면 RDS의 보안그룹을 살펴보겠습니다.

### RDS 의 보안그룹 설정

![rds-sg](./assets/connection_to_rds_with_ip.png)

- 첫번째 Inbound Rule

  - 먼저 RDS가 PostgreSQL 이기 때문에, Type 을 PostgreSQL 로 잡습니다.
  - 그리고 PostgreSQL 의 기본 포트 번호 5432 를 사용하기 때문에, Port Range 에
    5432 를 설정합니다.
  - 그리고 만약을 위한 보안 방책으로 자신의 IP를 설정합니다.

- 두번째 Inbound Rule
  - 두번째 Inbound Rule 은 개발서버(EC2)의 보안그룹을 잡습니다.
  - 이렇게 해서 EC2로 부터 들어오는 트래픽을 연결합니다.
  - RDS 의 Outbound Rule 은 EC2와 동일하게 All traffic 으로 설정하여,
    어떤 곳에서든 허용하게 해주었습니다.
