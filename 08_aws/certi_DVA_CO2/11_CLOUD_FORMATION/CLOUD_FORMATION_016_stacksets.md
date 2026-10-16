# CloudFormation StackSets

## 왜 StackSets가 필요한가요?

회사가 커지면 개발, 스테이징, 운영처럼 AWS 계정을 여러 개로 나누고, 서비스 지역에 따라 여러 리전을 사용합니다.
이때 "모든 계정의 모든 리전에 같은 보안 설정을 적용하라"는 요구가 생기면, 계정과 리전마다 스택을 하나씩 만드는
방식은 번거롭고 실수하기 쉽습니다.

StackSets는 **하나의 템플릿과 한 번의 작업으로 여러 계정·여러 리전에 스택을 생성·업데이트·삭제**하는
기능입니다.

- 공식 문서:
  [StackSets concepts](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-concepts.html)
  (확인일: 2026-10-13)

## 주요 용어

| 용어                   | 의미                                                                             |
| ---------------------- | -------------------------------------------------------------------------------- |
| 관리자 계정            | StackSet을 만들고 관리하는 계정입니다.                                           |
| 대상 계정              | StackSet이 실제 스택을 만드는 계정입니다.                                        |
| StackSet               | 여러 계정·리전에 배포할 스택들을 묶는 컨테이너입니다. 같은 템플릿을 사용합니다.  |
| 스택 인스턴스          | 특정 대상 계정의 특정 리전에 있는 스택에 대한 참조입니다.                        |

```shell
관리자 계정
  └─ StackSet (템플릿 1개)
       ├─ 리전 1: 계정 A의 스택 인스턴스, 계정 B의 스택 인스턴스
       └─ 리전 2: 계정 A의 스택 인스턴스, 계정 C의 스택 인스턴스
```

- StackSet을 업데이트하면 연결된 **모든** 스택 인스턴스가 모든 계정과 리전에서 업데이트됩니다. 일부
  스택에만 다른 템플릿을 적용할 수는 없습니다.
- 스택마다 다른 값이 필요하면 파라미터로 조정합니다.
- StackSet은 리전 단위 리소스입니다. 만든 리전에서만 조회하고 변경할 수 있습니다.

## 권한 모델

| 구분           | Self-managed 권한                                     | Service-managed 권한                                     |
| -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| IAM 역할 준비  | 관리자 계정과 대상 계정에 필요한 역할을 직접 만듭니다. | StackSets가 필요한 역할을 대신 만듭니다.                 |
| 배포 대상      | 역할을 만들 수 있는 모든 AWS 계정                     | AWS Organizations에 속한 계정(조직 단위 OU로 지정)       |
| 관리자 계정    | StackSet을 만든 계정                                  | 조직의 관리 계정 또는 위임된 관리자(Delegated Administrator) |
| 자동 배포      | 지원하지 않습니다.                                    | OU에 새 계정이 추가되면 자동으로 스택을 배포할 수 있습니다. |

- Self-managed 방식은 보통 관리자 계정에
  `AWSCloudFormationStackSetAdministrationRole`, 대상 계정에
  `AWSCloudFormationStackSetExecutionRole` 역할을 만들어 신뢰 관계를 맺습니다.
- Service-managed 방식은 먼저 Organizations에서 StackSets의 신뢰할 수 있는
  액세스(trusted access)를 활성화해야 합니다.
- 공식 문서 기준으로 Service-managed 권한의 StackSet은 Macro와 Transform을 포함한
  템플릿을 지원하지 않습니다.

## 작업 옵션

여러 계정에 한 번에 배포하므로, 동시에 몇 곳을 처리할지와 실패를 얼마나 허용할지를 정할 수 있습니다.

| 옵션                      | 의미                                                                     |
| ------------------------- | ------------------------------------------------------------------------ |
| Maximum concurrent accounts | 한 번에 작업할 대상 계정 수 또는 비율입니다.                           |
| Failure tolerance         | 리전마다 허용할 실패 스택 수 또는 비율입니다. 넘으면 작업을 멈춥니다.    |
| Region concurrency        | 리전에 순차(Sequential, 기본값) 또는 병렬(Parallel)로 배포합니다.        |
| Retain stacks             | StackSet에서 스택 인스턴스를 제거해도 대상 계정의 스택과 리소스를 남깁니다. |

처음에는 동시 계정 수와 실패 허용치를 낮게 잡아 문제를 빨리 발견하고, 안정성이 확인되면 높이는 방식이 안전합니다.

## CLI 예시

아래는 Organizations의 특정 OU에 속한 모든 계정의 두 리전에 기본 설정 템플릿을 배포하는 예시입니다.
관리 계정 또는 위임된 관리자 계정에서, `baseline.yaml` 템플릿이 있는 디렉터리에서 실행한다고 가정합니다.

```shell
# 1. StackSet 생성 (Service-managed 권한, 새 계정 자동 배포)
aws cloudformation create-stack-set \
  --stack-set-name org-baseline \
  --template-body file://baseline.yaml \
  --permission-model SERVICE_MANAGED \
  --auto-deployment Enabled=true,RetainStacksOnAccountRemoval=false

# 2. 대상 OU와 리전에 스택 인스턴스 생성
aws cloudformation create-stack-instances \
  --stack-set-name org-baseline \
  --deployment-targets OrganizationalUnitIds=ou-xxxx-xxxxxxxx \
  --regions ap-northeast-2 us-east-1 \
  --operation-preferences MaxConcurrentPercentage=25,FailureTolerancePercentage=10

# 3. 스택 인스턴스 상태 확인
aws cloudformation list-stack-instances --stack-set-name org-baseline
```

- `org-baseline`, `baseline.yaml`, `ou-xxxx-xxxxxxxx`는 예시 값입니다. OU
  ID는 Organizations 콘솔에서 확인한 실제 값으로 바꿉니다.
- 위임된 관리자 계정에서 실행할 때는 각 명령에 `--call-as DELEGATED_ADMIN`을 추가합니다.
- 템플릿이 IAM 리소스를 만들면 `create-stack-set`에
  `--capabilities CAPABILITY_NAMED_IAM` 같은 옵션이 필요합니다.
- 스택 인스턴스가 모두 `CURRENT` 상태가 되면 배포가 완료된 것입니다. `OUTDATED`라면 작업이
  실패했거나 중간에 멈춘 것이므로 원인을 확인합니다.

스택 인스턴스를 삭제하면 `--retain-stacks`를 지정하지 않은 경우 대상 계정의 스택과 리소스가 함께
삭제됩니다. 여러 계정에 한 번에 적용되므로 삭제 대상 계정·리전과 보존 여부를 먼저 확인합니다. StackSet
자체는 스택 인스턴스가 하나도 없을 때만 삭제할 수 있습니다.

## 실무 활용

- 모든 계정에 CloudTrail, AWS Config 같은 감사·보안 설정을 일괄 적용
- 모든 계정에 공통 IAM 역할(예: 보안팀 점검용 역할) 배포
- 여러 리전에 같은 네트워크·모니터링 기본 구성을 배포

## 정리

- StackSets는 하나의 템플릿으로 여러 계정·리전에 스택을 한 번에 생성·업데이트·삭제합니다.
- 관리자 계정에서 StackSet을 만들고, 대상 계정·리전마다 스택 인스턴스가 생깁니다.
- StackSet을 업데이트하면 연결된 모든 스택 인스턴스가 업데이트됩니다.
- Service-managed 권한은 AWS Organizations와 연동되며, 관리 계정 또는 위임된 관리자만
  만들 수 있고 새 계정 자동 배포를 지원합니다.

## 시험 대비 문제

### 문제 1

보안팀이 회사의 모든 AWS 계정과 여러 리전에 동일한 AWS Config 규칙을 배포하려고 합니다. 한 번의 작업으로
배포하고, 이후 변경도 한 번에 반영하고 싶습니다. 가장 적절한 방법은 무엇입니까?

- A. 각 계정에서 같은 템플릿으로 스택을 하나씩 만든다.
- B. 중첩 스택(Nested Stack)으로 계정별 스택을 묶는다.
- C. CloudFormation StackSets로 대상 계정과 리전에 스택 인스턴스를 배포한다.
- D. Cross-Stack Reference로 다른 계정의 스택 출력값을 가져온다.

#### 정답

C

#### 해설

StackSets는 하나의 템플릿으로 여러 계정·리전에 스택을 생성하고, StackSet을 업데이트하면 모든 스택
인스턴스에 변경을 반영합니다. 중첩 스택과 Cross-Stack Reference는 같은 계정·리전 안에서 스택을
나누고 연결하는 기능입니다.

### 문제 2

AWS Organizations를 사용하는 회사가 특정 OU에 새 계정이 추가될 때마다 공통 IAM 역할이 자동으로
만들어지기를 원합니다. 관리 부담을 가장 줄이는 StackSets 구성은 무엇입니까?

- A. Self-managed 권한으로 StackSet을 만들고, 새 계정이 생길 때마다 실행 역할을 수동으로
  만든다.
- B. Service-managed 권한으로 StackSet을 만들고 자동 배포(automatic
  deployment)를 활성화한다.
- C. 새 계정에서 Termination Protection을 활성화한다.
- D. 각 계정에 Custom Resource를 배포해 IAM 역할을 만든다.

#### 정답

B

#### 해설

Service-managed 권한은 AWS Organizations와 연동되어 필요한 IAM 역할을 StackSets가
대신 만들고, 자동 배포를 켜면 OU에 새로 추가된 계정에도 스택 인스턴스를 자동으로 생성합니다.
Self-managed 방식은 대상 계정마다 실행 역할을 직접 준비해야 합니다.
