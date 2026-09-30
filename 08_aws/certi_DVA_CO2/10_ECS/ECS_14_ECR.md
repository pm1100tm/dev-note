# Amazon ECR(Elastic Container Registry)

Amazon ECR(Elastic Container Registry)은 Docker 이미지와 OCI(Open
Container Initiative) 이미지, OCI 호환 아티팩트를 저장하고 관리하는
AWS 관리형 컨테이너 레지스트리입니다.

ECS는 ECR에 저장된 이미지를 Pull하여 Task의 컨테이너로 실행합니다.

이 문서는 Private Repository의 기본 사용 방법을 설명합니다.

내용은 2026-09-30에 확인했으며, 자세한 사양은 [Amazon ECR 공식 문서](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)를 참고합니다.

## ECR과 ECS의 관계

- ECR은 이미지를 저장하고, ECS는 그 이미지를 실행합니다.
- ECR에 이미지를 Push했다고 컨테이너가 자동 실행되지는 않습니다.
- ECS Task Definition의 `image`에 ECR Repository URI와 태그를 지정해야 합니다.

```text
개발자 → Docker image build → ECR image push
  → ECS Task Definition에 image URI 지정 → ECS image pull → 컨테이너 실행
```

- ECR은 이미지 레이어를 ECR이 관리하는 Amazon S3 버킷에 저장합니다.
- 사용자는 버킷을 직접 관리하지 않습니다.
- 프라이빗 서브넷의 ECS Task가 이미지를 가져올 때는 ECR API와 이미지 레이어가 있는 S3에 연결할
  네트워크 경로가 필요합니다.

## Private Repository와 Public Repository

| 구분   | Private Repository                                               | Public Repository                                                  |
| ------ | ---------------------------------------------------------------- | ------------------------------------------------------------------ |
| 사용처 | 조직 내부 애플리케이션 이미지                                    | 공개 배포할 오픈 소스 또는 샘플 이미지                             |
| Pull   | IAM 또는 Repository Policy로 허용한 주체만 가능합니다.           | 공개 이미지는 익명으로도 Pull할 수 있습니다.                       |
| 주소   | `<account-id>.dkr.ecr.<region>.amazonaws.com/<repository>:<tag>` | `public.ecr.aws/<registry-alias>/<repository>:<tag>`               |
| 탐색   | 해당하지 않습니다.                                               | [Amazon ECR Public Gallery](https://gallery.ecr.aws)에서 찾습니다. |

Private Repository의 다른 AWS 계정 접근은 IAM Identity Policy와
Repository Policy를 함께 검토합니다.

Public Repository에는 비밀번호, 인증서, 내부 URL, 상용 라이선스 파일을 포함하면 안 됩니다.

Public Gallery의 동작은 [ECR Public 공식 문서](https://docs.aws.amazon.com/AmazonECR/latest/public/public-registries.html)
에서 확인할 수 있습니다.

## Repository, Tag, Digest

| 식별자     | 예시                          | 용도                                                            |
| ---------- | ----------------------------- | --------------------------------------------------------------- |
| Repository | `demo`                        | 같은 애플리케이션 이미지의 저장 공간입니다.                     |
| Tag        | `1.2.0`, `20260930`, `latest` | 배포 버전이나 용도를 사람이 구분합니다.                         |
| Digest     | `sha256:...`                  | 정확히 같은 이미지 내용을 고정해 재현 가능한 배포에 사용합니다. |

- `latest`는 특별한 버전 번호가 아니라 일반 태그입니다.
- 같은 태그에 새 이미지를 Push할 수 있으면 `latest`가 가리키는 이미지가 바뀝니다.
- 운영 배포에서는 빌드 번호, Git commit SHA, 릴리스 버전 같은 태그나 Digest를 사용하면 추적하기 쉽습니다.

- Repository의 Tag immutability를 켜면 기존 태그를 다른 이미지로 덮어쓸 수 없습니다.
- 같은 태그를 다시 Push하면 `ImageTagAlreadyExistsException`이 발생합니다.

자세한 설정은 [태그 변경 방지 공식 문서](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html)를 참고합니다.

## 이미지 Push 준비

다음 전제 조건이 필요합니다.

- Docker가 실행 중이어야 합니다.
- AWS CLI v2와 AWS 자격 증명(`aws configure` 또는 IAM Role)이 필요합니다.
- 대상 리전에 Private Repository가 있어야 합니다.
- 현재 IAM 주체에 ECR 로그인과 Push 권한이 있어야 합니다.

다음 명령으로 예제 Repository를 만듭니다.
이미 존재하는 Repository에 실행하면 오류가 발생하므로 최초 한 번만 실행합니다.

```shell
aws ecr create-repository \
  --repository-name demo \
  --image-tag-mutability IMMUTABLE \
  --region ap-northeast-2
```

출력의 `repositoryUri`가 이미지에 붙일 주소입니다.
이 문서의 `123456789012`는 예시 AWS 계정 ID이므로 실제 값으로 바꿉니다.

## AWS CLI v2로 Docker 로그인

ECR Private Repository에 Push하거나 Pull하려면 Docker 클라이언트가
대상 Registry에 인증해야 합니다. `get-login-password`는 IAM 권한으로
발급받은 인증 토큰을 출력하고, 이를 `docker login`의 표준 입력으로
전달합니다.

```shell
aws ecr get-login-password --region ap-northeast-2 \
  | docker login --username AWS --password-stdin \
    123456789012.dkr.ecr.ap-northeast-2.amazonaws.com
```

인증 토큰은 발급한 IAM 주체의 권한 범위를 따르며 12시간 동안
유효합니다. 토큰이 만료되거나 다른 리전의 Registry에 접근하면 다시
로그인해야 합니다. 인증 토큰의 동작은 [Private Registry 인증 문서](https://docs.aws.amazon.com/AmazonECR/latest/userguide/registry_auth.html)에서 확인할 수 있습니다.

비밀번호나 `get-login-password`의 출력을 파일, 소스 코드, 빌드 로그에
저장하지 않습니다. 위 명령처럼 표준 입력으로 바로 전달합니다.

## Docker 이미지 Tag, Push, Pull

로컬에 `demo:1.0.0` 이미지가 있다고 가정합니다. 먼저 ECR
Repository URI를 붙인 새 Tag를 만든 뒤 Push합니다.

```shell
docker tag demo:1.0.0 \
  123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/demo:1.0.0

docker push \
  123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/demo:1.0.0
```

다른 개발 환경이나 EC2 인스턴스에서 Pull하려면 먼저 같은 Registry에
로그인한 뒤 이미지 URI를 사용합니다.

```shell
docker pull \
  123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/demo:1.0.0
```

다음 명령으로 저장된 이미지의 태그와 Digest를 확인할 수 있습니다.

```shell
aws ecr describe-images \
  --repository-name demo \
  --region ap-northeast-2 \
  --query 'imageDetails[].{Tags:imageTags,Digest:imageDigest,PushedAt:imagePushedAt}' \
  --output table
```

AWS CLI와 Docker CLI로 Repository 생성, 인증, Push, Pull을 수행하는
전체 절차는 [ECR 시작 안내](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html)
를 참고합니다.

## IAM 권한과 ECS 이미지 Pull

Private ECR 접근은 IAM으로 제어합니다. `docker login`을 실행하는
사용자 또는 Role에는 `ecr:GetAuthorizationToken` 권한이 필요합니다.

이미지를 Pull하거나 Push하려면 이미지 조회, 레이어 다운로드 또는 업로드 권한도 필요합니다.

| 실행 환경                 | 먼저 확인할 권한 주체                                   |
| ------------------------- | ------------------------------------------------------- |
| ECS Fargate               | Task Execution Role                                     |
| ECS EC2 Launch Type       | ECS Container Instance의 IAM Role 또는 Instance Profile |
| 로컬 Docker 또는 일반 EC2 | 명령을 실행하는 IAM User 또는 IAM Role                  |

Fargate에서 Private ECR 이미지를 Pull하는 권한은 Task Role이 아니라 Task Execution Role에
부여합니다.

AWS 관리형 정책 `AmazonECSTaskExecutionRolePolicy`에는 일반적인 Private ECR 이미지
Pull 권한이 포함되어 있습니다.

ECS용 ECR 권한과 시작 유형별 Role은 [ECR 이미지를 ECS에서 사용하는 방법](https://docs.aws.amazon.com/AmazonECR/latest/userguide/ECR_on_ECS.html)
을 참고합니다.

로컬 또는 EC2에 읽기 전용 권한이 필요하면 AWS 관리형 정책 `AmazonEC2ContainerRegistryReadOnly`도 검토할 수 있습니다.

## 이미지 보안과 수명 주기 관리

| 기능               | 목적                                                   | 적용 시 유의점                                       |
| ------------------ | ------------------------------------------------------ | ---------------------------------------------------- |
| 이미지 취약점 스캔 | 운영체제 패키지와 의존성의 알려진 취약점을 확인합니다. | 결과의 심각도와 적용 가능성을 검토합니다.            |
| Tag immutability   | 동일 태그를 다른 이미지로 덮어쓰는 일을 막습니다.      | 새 버전 태그가 필요합니다.                           |
| Lifecycle Policy   | 오래된 이미지와 태그 없는 이미지를 자동 만료합니다.    | 적용 전에 Preview로 삭제 대상을 확인합니다.          |
| 암호화             | 저장된 이미지 레이어를 암호화합니다.                   | Repository 생성 후 암호화 설정은 변경할 수 없습니다. |

- Lifecycle Policy는 이미지의 태그, 생성 시점, 개수를 기준으로 만료 규칙을 정의합니다.
- 이미지를 삭제하면 되돌리기 어려우므로 먼저 Lifecycle Policy Preview로 대상을 확인해야 합니다.
- 정책에 일치한 이미지는 보통 24시간 이내에 만료 처리됩니다.

자세한 동작은 [ECR Lifecycle Policy 공식 문서](https://docs.aws.amazon.com/AmazonECR/latest/userguide/LifecyclePolicies.html)
를 참고합니다.

## Pull 또는 Push 실패 시 확인 순서

| 증상                             | 확인할 원인                                     | 조치                                                      |
| -------------------------------- | ----------------------------------------------- | --------------------------------------------------------- |
| `no basic auth credentials`      | Docker 로그인 누락 또는 12시간 토큰 만료        | 대상 리전의 Registry URI로 다시 로그인합니다.             |
| `AccessDeniedException`          | IAM Identity Policy 또는 Repository Policy 부족 | 로그인 권한과 Pull 또는 Push 권한을 부여합니다.           |
| `repository does not exist`      | Repository 이름, 계정 ID, 리전이 다름           | ECR 콘솔 또는 `describe-repositories`로 URI를 확인합니다. |
| `ImageTagAlreadyExistsException` | Immutable Repository에서 기존 태그를 재사용함   | 새 버전 태그를 사용하거나 태그 정책을 검토합니다.         |
| ECS의 `CannotPullContainerError` | Task Execution Role 또는 네트워크 경로 문제     | Role 권한과 ECR API, ECR DKR, S3 연결을 확인합니다.       |

프라이빗 서브넷의 Fargate Task는 NAT Gateway를 통해 인터넷에
나가거나 ECR Interface VPC Endpoint와 S3 Gateway Endpoint를 통해
ECR에 접근할 수 있습니다.

이미지 레이어는 S3에 저장되므로 ECR Endpoint만 구성하면 Pull이 완성되지 않을 수 있습니다.

Endpoint 구성은 [ECR VPC Endpoint 공식 문서](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html)
를 참고합니다.

## 핵심 정리

- ECR은 컨테이너 이미지를 저장하고 관리하며, ECS는 그 이미지를
  Pull하여 실행합니다.
- Private Repository는 IAM과 Repository Policy로 접근을 제어하고,
  Public Repository는 Amazon ECR Public Gallery에서 공개합니다.
- AWS CLI v2의 `get-login-password`와 `docker login`으로 대상
  Registry에 인증한 뒤 Tag, Push, Pull 순서로 작업합니다.
- 이미지 Pull 실패는 계정 ID·리전·태그, IAM 권한, 네트워크 경로를
  함께 확인합니다.
- 운영 Repository에서는 취약점 스캔, 불변 태그, Lifecycle Policy를
  조합해 이미지 품질과 저장 비용을 관리합니다.
