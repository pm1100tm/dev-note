# CloudFormation DeletionPolicy

## 왜 DeletionPolicy가 필요한가요?

스택을 삭제하면 그 스택이 만든 리소스도 기본적으로 함께 삭제됩니다. 개발 환경을 정리할 때는 편리하지만, 운영
데이터베이스나 로그를 보관하는 S3 버킷까지 사라지면 복구하기 어렵습니다.

`DeletionPolicy`는 리소스마다 "스택이 지워질 때 이 리소스를 어떻게 처리할지"를 정하는 속성입니다.
삭제할지, 남겨 둘지, 스냅샷을 만든 뒤 삭제할지를 고를 수 있어 데이터 손실을 막는 안전장치 역할을 합니다.

- 공식 문서:
  [DeletionPolicy attribute](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-deletionpolicy.html)
  (확인일: 2026-10-13)

## 언제 적용되나요?

`DeletionPolicy`는 다음 두 경우에 동작합니다.

- 스택 자체를 삭제할 때
- 스택 업데이트에서 템플릿의 리소스 정의를 제거해 리소스가 스택에서 빠질 때

반면 속성을 바꿔 리소스가 **교체(Replacement)** 되는 경우에는 적용되지 않습니다. 예를 들어 RDS의
`Engine` 속성을 바꾸면 CloudFormation은 새 DB를 만들고 이전 DB를 지우는데, 이때 이전 리소스의
처리 방식은 `UpdateReplacePolicy` 속성으로 따로 정합니다.

- 공식 문서:
  [UpdateReplacePolicy attribute](https://docs.aws.amazon.com/AWSCloudFormation/latest/TemplateReference/aws-attribute-updatereplacepolicy.html)

## 옵션 비교

| 옵션                   | 동작                                                     | 적용 대상                    |
| ---------------------- | -------------------------------------------------------- | ---------------------------- |
| `Delete`               | 리소스와 내용을 삭제합니다. 대부분 리소스의 기본값입니다. | 모든 리소스                  |
| `Retain`               | 스택이 삭제되어도 리소스를 그대로 남깁니다.              | 모든 리소스                  |
| `Snapshot`             | 마지막 스냅샷을 만든 뒤 리소스를 삭제합니다.             | 스냅샷을 지원하는 리소스     |
| `RetainExceptOnCreate` | 처음 생성하던 작업이 롤백되면 삭제하고, 그 외에는 보존합니다. | 모든 리소스             |

`RetainExceptOnCreate`는 슬라이드에는 없는 옵션으로, 공식 문서 기준으로 추가 확인한 내용입니다.

### Delete: 기본 동작

`DeletionPolicy`를 지정하지 않으면 CloudFormation은 리소스를 삭제합니다. 다만 공식 문서에는
다음 예외와 주의 사항이 있습니다. 슬라이드는 "기본값은 Delete"라고만 설명하므로 함께 기억해 둡니다.

- `AWS::RDS::DBCluster`의 기본값은 `Snapshot`입니다.
- `DBClusterIdentifier` 속성을 지정하지 않은 `AWS::RDS::DBInstance`의 기본값도
  `Snapshot`입니다.
- S3 버킷은 객체가 하나라도 남아 있으면 삭제에 실패합니다. 버킷을 비우는 방법은
  [Custom Resources](CLOUD_FORMATION_015_custom_resources.md)에서
  다룹니다.

### Retain: 리소스 보존

`Retain`은 모든 리소스 타입에 사용할 수 있습니다. 스택 삭제가 끝나도 리소스는 계속 남아 있으며,
CloudFormation의 관리 대상에서만 빠집니다.

```yaml
Resources:
  LogBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
```

- 보존된 리소스는 계속 요금이 부과됩니다. 필요 없어지면 직접 삭제해야 합니다.
- S3 버킷을 `Retain`으로 남기면 같은 이름으로 새 버킷을 만들 수 없으므로, 버킷 이름을 고정했다면 재배포
  전에 확인합니다.

### Snapshot: 삭제 전 마지막 백업

`Snapshot`은 리소스를 삭제하기 직전에 스냅샷을 만듭니다. 리소스는 사라지지만 스냅샷으로 나중에 복원할 수
있습니다.

```yaml
Resources:
  AppDatabase:
    Type: AWS::RDS::DBInstance
    DeletionPolicy: Snapshot
    UpdateReplacePolicy: Snapshot
    Properties:
      Engine: mysql
      DBInstanceClass: db.t3.micro
      AllocatedStorage: '20'
      MasterUsername: admin
      ManageMasterUserPassword: true
```

위 예시는 스냅샷 설정을 보여 주기 위한 조각이며, 실제 배포에는 서브넷 그룹과 보안 그룹 같은 네트워크 설정이 추가로
필요합니다. `ManageMasterUserPassword: true`는 비밀번호를 템플릿에 쓰지 않고 Secrets
Manager가 관리하도록 하는 설정입니다.

공식 문서 기준으로 `Snapshot`을 지원하는 리소스는 다음과 같습니다.

- `AWS::EC2::Volume`
- `AWS::ElastiCache::CacheCluster`,
  `AWS::ElastiCache::ReplicationGroup`
- `AWS::RDS::DBInstance`, `AWS::RDS::DBCluster`
- `AWS::Redshift::Cluster`
- `AWS::Neptune::DBCluster`
- `AWS::DocDB::DBCluster`

스냅샷도 삭제할 때까지 스토리지 요금이 부과됩니다.

### RetainExceptOnCreate: 새로 만들다 실패한 리소스만 삭제

`RetainExceptOnCreate`는 평소에는 `Retain`처럼 동작하지만, 리소스를 처음 만든 작업이 롤백될
때는 삭제합니다. 생성 실패로 남은 빈 리소스는 정리하고, 이미 사용 중인 리소스와 데이터는 보존하고 싶을 때
사용합니다.

## 실무 활용과 주의 사항

- 운영 데이터가 있는 RDS, DynamoDB 테이블, S3 버킷에는 `DeletionPolicy`를 명시합니다.
  기본값에 기대지 않고 의도를 템플릿에 남기는 것이 좋습니다.
- 교체가 일어날 수 있는 데이터 리소스에는 `UpdateReplacePolicy`도 같은 값으로 함께 지정합니다.
- `Retain`과 `Snapshot`은 삭제를 막는 대신 남은 리소스와 스냅샷 비용이 생기므로, 주기적으로 정리
  대상을 확인합니다.
- 스택 삭제는 되돌릴 수 없습니다. 삭제 전에 `DeletionPolicy` 설정과 백업 여부를 먼저 확인합니다.

## 정리

- `DeletionPolicy`는 스택 삭제 또는 템플릿에서 리소스를 제거할 때 리소스 처리 방식을 정합니다.
- 기본값은 `Delete`이지만, RDS DB 클러스터와 클러스터에 속하지 않은 RDS DB 인스턴스는
  `Snapshot`이 기본값입니다.
- `Retain`은 모든 리소스에, `Snapshot`은 스냅샷을 지원하는 리소스에만 사용할 수 있습니다.
- 객체가 남아 있는 S3 버킷은 `Delete` 정책이어도 삭제에 실패합니다.
- 업데이트 중 리소스 교체에 대한 처리는 `UpdateReplacePolicy`로 정합니다.

## 시험 대비 문제

### 문제 1

CloudFormation으로 관리하는 Amazon RDS DB 인스턴스가 있습니다. 스택을 삭제하더라도 나중에
데이터를 복원할 수 있도록 마지막 백업을 남기고 싶습니다. 가장 적절한 설정은 무엇입니까?

- A. `DeletionPolicy: Delete`
- B. `DeletionPolicy: Snapshot`
- C. Stack Policy에서 `Update:Delete`를 거부한다.
- D. 스택에 Termination Protection을 활성화한다.

#### 정답

B

#### 해설

`DeletionPolicy: Snapshot`은 리소스를 삭제하기 전에 마지막 스냅샷을 만들어 데이터 복원이 가능하게
합니다. Stack Policy는 스택 업데이트 중의 변경만 제어하고, Termination Protection은 스택
삭제 자체를 막을 뿐 백업을 만들지 않습니다.

### 문제 2

개발자가 CloudFormation 스택을 삭제했지만 스택 상태가 `DELETE_FAILED`가 되었습니다. 이벤트를
확인하니 `AWS::S3::Bucket` 리소스 삭제가 실패했습니다. 템플릿에는 `DeletionPolicy`가
지정되어 있지 않습니다. 가장 가능성이 높은 원인은 무엇입니까?

- A. S3 버킷의 기본 `DeletionPolicy`가 `Retain`이기 때문이다.
- B. S3 버킷 안에 객체가 남아 있기 때문이다.
- C. S3 버킷은 CloudFormation으로 삭제할 수 없기 때문이다.
- D. 스택에 Stack Policy가 설정되어 있기 때문이다.

#### 정답

B

#### 해설

`DeletionPolicy`를 지정하지 않으면 기본값은 `Delete`이지만, S3 버킷은 객체가 남아 있으면 삭제할
수 없습니다. 삭제 전에 버킷을 비우거나, Lambda 기반 Custom Resource로 스택 삭제 시 버킷을
자동으로 비우도록 구성합니다. Stack Policy는 스택 업데이트에만 적용되므로 스택 삭제와는 관계가 없습니다.
