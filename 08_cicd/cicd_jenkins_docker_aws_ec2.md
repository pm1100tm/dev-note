# 🚀 Jenkins 를 사용하여 AWS EC2 에 docker container 실행까지의 pipeline 구축

## 1️⃣ EC2 서버 준비 & RDS 준비

- 두 개의 EC2 인스턴스를 t3.medium 타입 이상으로 생성합니다.

  - Jenkins 서버 (EC2 #1): t3.medium
  - 개발 서버 (EC2 #2): t3.medium
  - RDS 생성

```plain
📚 t3.medium 타입 이상이 필요한 이유
- 지금 여기서는 Jenkins 와 Java 를 Docker Image 를 사용하여 배포하는 것이 목적합니다.
- 일단 Docker 를 설치하고 실행하게 되면, 1GB 의 메모리는 부족합니다.
- t3.micro 또는 t3.small 타입은 1GB 의 메모리 밖에 주어지지 않습니다.
- 실제로 Jenkins UI 가 매우 느렸으며, 수시로 서버가 죽는 현상을 경험하였습니다.
```

### 보안 그룹 설정

- Jenkins 와 개발서버에 대해서 동일한 보안 그룹을 사용하였습니다.
- 필요에 따라서 보안 그룹을 나누어서 설정할 수 있습니다.
- 사용자 지정 TCP 8080 port \*Jenkins 는 일반적으로 8080 포트 샤용
- SSH 22 port \* SSH 접속을 위해서
- RDS 에 대한 보안 그룹 생성 후, Inbound 규칙에서 개발 서버의 보안 그룹 허용

![개발 서버 보안그룹](./assets/jenkins/sg1.png)
![RDS 보안그룹](./assets/jenkins/sg2.png)

### 개발 서버와 Jenkins 서버에 대해서 탄력적 IP(EIP) 적용하기

서버 접속 및 코드 상에서 IP 입력 부분에서 매번 변경하지 않도록, 고정 IPv4 를 생성하여,
각 인스턴스에 연결해줍니다.

> 📚 EIP 는 인스턴스와 연결 중일 때는 한 개 까지 무료이며, 연결되지 않은 상태로 두면 요금 부과

## 2️⃣ Jenkins 서버에 기본 환경 설치 (EC2 #1)

SSH 접속 후 Docker & Compose 설치:

```shell
# 1. 패키지 리스트 업데이트 & 보안 업데이트 적용
sudo apt update && sudo apt upgrade -y
# → 최신 보안 패치와 패키지 리스트를 갱신합니다.

# 2. 필수 패키지 설치 (HTTPS 저장소 접근, GPG 키 등록 등에 필요)
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release

# 3. Docker 공식 GPG 키를 받아서 시스템에 등록
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# → Docker 패키지가 위조되지 않았음을 검증하기 위한 키 등록
# → /usr/share/keyrings/docker-archive-keyring.gpg 파일 생성됨

# 4. Docker 공식 저장소 추가
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# → apt가 Docker 공식 저장소에서 패키지를 가져올 수 있도록 등록

# 5. 패키지 리스트 다시 업데이트 (Docker 저장소 반영됨)
sudo apt update

# 6. Docker 엔진 및 CLI 설치
sudo apt install -y docker-ce docker-ce-cli containerd.io
# → docker-ce: Docker Community Edition 엔진
# → docker-ce-cli: Docker CLI (docker 명령어)
# → containerd.io: 컨테이너 런타임

# 7. 현재 사용자에게 docker 그룹 권한 부여 - 중요함
sudo usermod -aG docker $USER
# → sudo 없이 `docker ps` 같은 명령 실행 가능
#   (적용하려면 로그아웃/로그인 필요 or `newgrp docker` 실행)

# 8. Docker 버전 확인
docker --version
# → Docker 엔진이 정상 설치되었는지 확인

# 9. Docker Compose v2 플러그인 설치 (공식 권장 방식)
sudo apt install -y docker-compose-plugin
# → 설치 후 `docker compose` 명령으로 사용 가능 (공백 있는 방식)

# 10. Docker Compose 버전 확인
docker compose version
# → Compose v2 버전 출력 확인
```

---

## 3️⃣ Jenkins Dockerfile / Docker Compose 세팅

### Dockerfile

```Dockerfile
# Jenkins LTS + JDK 21 기반 이미지
FROM jenkins/jenkins:lts-jdk21

# root 권한 유지 (패키지 설치 및 플러그인 설치용)
USER root

# 기본 유틸 설치
RUN apt-get update && apt-get install -y \
    git \
    curl \
    unzip \
    && rm -rf /var/lib/apt/lists/*

# 플러그인 목록 복사 및 설치
COPY docker-dev/jenkins-plugin.txt /usr/share/jenkins/ref/jenkins-plugin.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/jenkins-plugin.txt

# jenkins 사용자로 다시 전환
USER jenkins
```

jenkins-plugin.txt:

```plain
# 📂 프로젝트 구조 관리
cloudbees-folder

# 🔐 보안
antisamy-markup-formatter
credentials-binding

# 🔄 파이프라인 & GitHub 통합
workflow-aggregator
github-branch-source
git

# 🛠️ 빌드 도구
gradle

# 📡 원격 배포/에이전트
ssh-slaves

# 📧 알림
email-ext

# ⚙️ 운영 편의
timestamper
ws-cleanup
build-timeout
pipeline-graph-view
matrix-auth
```

jenkins-plugin.txt 를 통해서, Java Springboot 앱의 CI/CD 에 필요한 플러그인을, 미리
설치해줍니다.

### Docker compose 파일

네임드 볼륨 jenkins_home 를 생성하여 관리합니다.

```shell
version: '3.8'
services:
  jenkins:
    build:
      context: ..
      dockerfile: docker-dev/Dockerfile-jenkins
    container_name: myblog-jenkins
    user: root
    ports:
      - "8080:8080"
      - "50000:50000"
    environment:
      - JAVA_OPTS=-Xms512m -Xmx2048m  # JVM 메모리 제한
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock

volumes:
  jenkins_home:
```

#### 📦 네임드 볼륨(named volume)을 사용하는 이유

##### Docker 가 직접 관리 → 성능 최적화

- 네임드 볼륨은 Docker 엔진이 /var/lib/docker/volumes/ 아래에서 직접 관리합니다.
- 로컬 디스크와 달리 OverlayFS 최적화가 적용되어 입출력(I/O) 속도가 더 빠릅니다.
- Jenkins처럼 수많은 작은 파일(플러그인 캐시, 빌드 로그 등)을 다루는 경우 성능 차이가 큽니다.
  - 로컬 바인드 마운트는 호스트 파일시스템과 Docker OverlayFS 사이에서 I/O overhead가 발생합니다.
  - Jenkins는 수천 개의 작은 파일을 다루므로 속도 저하가 심각해집니다.
  - macOS/Windows에서는 Hypervisor를 거치기 때문에 더 느려집니다.

##### 데이터 영속성 보장

- 컨테이너가 삭제되어도 네임드 볼륨은 남아있습니다.
- docker compose down -v로 명시적으로 삭제하기 전까지는 안전하게 보관됩니다.
- Jenkins 서버를 재배포해도 설정, 플러그인, 빌드 히스토리가 그대로 유지됩니다.

##### 호스트 경로 독립성

- 특정 경로(./jenkins_home)에 의존하지 않고, 어떤 서버/환경에서도 동일하게 동작합니다.
- 여러 개발자/운영자가 같은 compose 파일을 써도 로컬 파일 경로 충돌이 없습니다.

### 실행

```shell
docker compose up -d
```

처음 실행하면 Jenkins 관리자 비밀번호를 로그에서 확인:

```shell
docker logs jenkins
```

- 브라우저에서 http://<JENKINS_EC2_PUBLIC_IP>:8080 접속 →
- Unlock Jenkins (로그에서 비밀번호 확인)
- 관리자 계정 생성
- 플러그인 설치
  - 위의 jenkins_plugin.txt 를 사용하지 않고, 직접 설치하는 경우
  - Github, SSH Agent, Pipeline 등 설치

## 4️⃣ 배포 서버 (EC2 #2) 설정

배포 서버에도 Docker & Docker Compose 설치 (Spring Boot 배포용):

**_위의, 2️⃣ Jenkins 서버에 기본 환경 설치 (EC2 #1) 참조_**

## 5️⃣ Jenkins → 배포 서버 SSH 연결

Jenkinsfile 을 프로젝트 루트에 생성하여, Jenkins 에서 해당 파일의 스크립트를 실행할 예정입니다.
그렇게 하기 위해서는, Jenkins 서버에서 개발 서버로의 ssh 접속이 가능해야 합니다.

- Jenkins 서버에서 ssh-keygen 을 사용하여, 키 파일을 만들어줍니다.
- 생성된 rsa 퍼블릭키를, 개발 서버에 등록해줍니다.
- 개발 서버의 ~/.ssh/authrized_keys 에 추가합니다.

```shell
# Jenkins 서버에서 키 생성
ssh-keygen -t rsa -b 4096 -C "jenkins-deploy" -f ~/.ssh/jenkins_id_rsa
```

젠킨스 서버에서 cat 으로 .pub 파일 출력 후 복사 -> 개발서버에 붙여넣기

```shell
ubuntu@ip-172-31-44-221:~/.ssh$ cat jenkins_id_rsa.pub
```

젠킨스 서버:

![Jenkins 서버](./assets/jenkins/ssh1.png)

개발 서버:

![개발 서버](./assets/jenkins/ssh2.png)

젠킨스 서버 -> 개발 서버 ssh 접속 테스트:

```shell
ssh -i ~/.ssh/jenkins_id_rsa ubuntu@<DEPLOY_SERVER_IP>
```

## 6️⃣ Jenkins Job (Pipeline) 구성

Jenkins UI에서 새로운 Item → Pipeline 생성.
GitHub 연동 후 아래 예시 파이프라인 스크립트 (Jenkinsfile) 작성:

![Jenkinsfile-dev](./assets/jenkins/jenkinsfile-dev.png)

```Jenkinsfile
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'develop',
                    url: '<git repository url>',
                    credentialsId: 'my-jenkins-credentials'
            }
        }

        stage('Build') {
            steps {
                sh './gradlew clean build -x test'
            }
        }

        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }

        stage('Deploy') {
            steps {
                sshagent (credentials: ['deploy-server-ssh']) {
                    sh """
                      ssh -o StrictHostKeyChecking=no ubuntu@<dev server public ip> '
                        cd ~/myblog-backend &&
                        git pull origin develop &&
                        docker compose -f docker-dev/compose-container.yml down &&
                        docker compose -f docker-dev/compose-container.yml up --build -d
                      '
                    """
                }
            }
        }
    }
}
```

위의 credential 설정에 대해서는, 아래의 [7️⃣ Jenkins Credential 설정] 에서 설명한다.

- credentialsId: 'my-jenkins-credentials'
- sshagent (credentials: ['deploy-server-ssh'])

📚 TIPs - pipeline 아래의 tools java 등을 설정하지 않은 이유

- 지금 설치된 Jenkins dockerfile 은 이미 Java 21 버전이 설치되어 있기 때문에, 별도의
  tools 설정이 필요 없습니다.

## 7️⃣ Jenkins Credential 설정

여기서는 아래의 credentialsId 와 sshagent 의 credentials 에 들어가는 값에 대해서 설명합니다.

```Jenkinsfile
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'develop',
                    url: '<git repository url>',
                    credentialsId: 'my-jenkins-credentials'
            }
        }

        ...

        stage('Deploy') {
            steps {
                sshagent (credentials: ['deploy-server-ssh']) {
                    sh """
                      ...
                    """
                }
            }
        }
    }
}
```

### 🔑 credentialsId 가 필요한 이유

GitHub 인증 문제 해결

- Jenkins에서 git 플러그인을 통해 private repo를 clone/pull 하려면 인증이 필요합니다.
- 예전에는 GitHub username/password를 썼지만, 지금은 Personal Access Token (PAT) 또는
  SSH key만 지원합니다.
- credentialsId는 Jenkins 내부 Credentials Store에 저장된 인증 정보를 참조하기 위한 키입니다.

보안 유지

- Jenkinsfile에 직접 토큰이나 비밀번호를 적으면 보안상 위험합니다. (GitHub에 노출될 수 있음)
- 대신 Jenkins Credentials Store에 안전하게 저장하고, Jenkinsfile에서는 credentialsId 만
  명시합니다.
- 실제 비밀값은 빌드 실행 시점에만 주입됩니다.

유연한 관리

- 같은 Jenkinsfile을 쓰더라도 Jenkins 서버마다 다른 인증정보를 연결할 수 있습니다.
- 예: 로컬 Jenkins는 개인 GitHub 토큰, 회사 Jenkins는 조직 계정 토큰 → 코드 수정 없이 가능.

#### credentialsId 설정 방법

Github 접속

- 오른쪽 상단 profile -> settings
- 왼쪽 하단 <> Developer settings
- Personal access tokens -> Tokens (classic)
- Generate new token
- Token name, Expiration, Repository access 설정
- 최소한 repo, workflow, write:packages 선택
- 생성되는 키 복사 (다시 볼 수 없음)

Jenkins 에서 Credential 설정

- 상단 프로필 -> Credentials 클릭
  ![jenkins-credentials](./assets/jenkins/jenkins-credential1.png)
- Stores from partent 의 (global) 클릭
  ![jenkins-credentials](./assets/jenkins/jenkins-credential2.png)
- Add Credentials 클릭
  ![jenkins-credentials](./assets/jenkins/jenkins-credential3.png)
- Username with password 로 설정
  - Username 은 Github Id 로 설정
  - Password 는 Github 에서 생성한 키 붙여넣기
  - ID 는 커스텀하게 설정하되, 해당 ID 를 Jenkinsfile 에서 사용함
  - ![jenkins-credentials](./assets/jenkins/jenkins-credential4.png)

#### deploy-server-ssh 설정 방법

- 위와 동일한 방법으로 Add Credentials 클릭
- SSH Username with private key 선택
- ID: deploy-server-ssh \*Jenkinsfile 에서 sshagent 에 들어가는 값
- Username: ubuntu
- Private Key 에 AWS .pem 키의 내용 붙여넣기
  - 아래와 같이 했을 때, terminal 에 출력되는 값의 마지막에 % 는 제외하고 복붙 하도록 한다

```shell
─    ~/.ssh ─────────────────────────────────────────────────────────── ✔
╰─ cat swd-aws-keypair.pem
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAmFw7llVRobtMJBtIctCzxffGrWNNc7KdePt2laL2pQ0lF7Zr
...
```

![jenkins credential](./assets/jenkins/jenkins-credential5.png)

## 8️⃣ GitHub → Jenkins Webhook 연결

- GitHub repo → Settings → Webhooks
  - Payload URL: http://<JENKINS_EC2_PUBLIC_IP>:8080/github-webhook/
  - Content type: application/json

![github webhook](./assets/jenkins/github-webhook.png)

## 9️⃣ Jenkins pipeline 설정

![pipeline](./assets/jenkins/jenkins-pipeline1.png)
![pipeline](./assets/jenkins/jenkins-pipeline2.png)
![pipeline](./assets/jenkins/jenkins-pipeline3.png)
![pipeline](./assets/jenkins/jenkins-pipeline4.png)
![pipeline](./assets/jenkins/jenkins-pipeline5.png)
![pipeline](./assets/jenkins/jenkins-pipeline6.png)
![pipeline](./assets/jenkins/jenkins-pipeline7.png)
![pipeline](./assets/jenkins/jenkins-pipeline8.png)
![pipeline](./assets/jenkins/jenkins-pipeline9.png)

---

✅ 완료

---

## Trouble Shooting

젠킨스 CI/CD 연동 작업에서 발생한 모든 트러블 슈팅을 기록합니다.

### ssh 접속 불가

✅ pem 키 파일 권한 확인

EC2 접속할 때 pem 키 권한이 잘못되면 바로 거절됩니다.

```shell
ls -l ~/.ssh/swd-aws-keypair.pem
```

정상 권한은 아래 중 하나여야 합니다:

- -r-------- (chmod 400)
- -rw------- (chmod 600)

```shell
chmod 400 ~/.ssh/swd-aws-keypair.pem
```

✅ 2. EC2 보안 그룹 (Security Group) 설정

AWS 콘솔 → EC2 → 인스턴스 선택 → 보안 그룹 확인

- Inbound 규칙에 아래가 있어야 함:
  - SSH (TCP) / 포트 22 / 소스: 내 IP (또는 0.0.0.0/0 테스트용)

✅ 3. 올바른 유저명 확인

AMI 종류에 따라 접속 계정이 달라집니다.

- Amazon Linux 2 / Amazon Linux 2023 → ec2-user
- Ubuntu → ubuntu
- CentOS → centos
- RHEL → ec2-user 또는 root

즉, AMI가 Ubuntu라면 아래처럼 시도해야 합니다:

```shell
ssh -i ~/.ssh/swd-aws-keypair.pem ubuntu@3.35.208.126
```

✅ 4. 퍼블릭 IP / Elastic IP 확인

- EC2에 퍼블릭 IPv4 주소가 할당돼 있는지 (또는 Elastic IP 연결했는지) 확인
- 프라이빗 서브넷에만 있으면 외부에서 접근 불가

✅ 5. 네트워크 ACL / VPC 라우팅 확인

- 퍼블릭 서브넷에 있어야 외부 접근 가능
- 인터넷 게이트웨이(IGW)가 VPC에 연결돼 있어야 함

✅ 6. SSH 로그 확인 (EC2 내부)

혹시 이미 내부 콘솔로 들어갈 수 있다면 (AWS Systems Manager Session Manager 같은 것)

- /var/log/secure 또는 /var/log/auth.log 확인해서 SSH 거절 이유 확인 가능

### docker compose up 명령어를 수행할 때 permission denided

```shell
ubuntu@ip-172-31-44-221:~/jenkins$ docker compose up -d
WARN[0000] /home/ubuntu/jenkins/docker-compose.yml: the attribute version is obsolete, it will be ignored, please remove it to avoid potential confusion
unable to get image 'jenkins/jenkins:lts-jdk21': permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.51/images/jenkins/jenkins:lts-jdk21/json": dial unix /var/run/docker.sock: connect: permission denied
```

에러 메세지의 핵심은 아래의 메세지이다.

```shell
permission denied while trying to connect to the Docker daemon socket
unix:///var/run/docker.sock: connect: permission denied
```

위에서 현재 유저를 docker group 에 추가하는 명령어를 실행하였는데, 그룹 변경은 즉시 반영이 아니기
때문에, 세션을 새로 열어야 한다. 따라서, 로그아웃 후 다시 ec2 에 접속한다.

```shell
exit
ssh -i ~/.ssh/swd-aws-keypair.pem ubuntu@<EC2_PUBLIC_IP>
```

그룹 확인하여, docker 가 포함되어 있어야 한다.

```shell
ubuntu@ip-172-31-44-221:~/jenkins$ groups
ubuntu adm cdrom sudo dip lxd docker
```
