# CloudFormation 동작 방식과 배포 방법

## 왜 동작 방식을 알아야 하나요?

CloudFormation은 템플릿 파일 하나만 넘기면 리소스를 만들어 주지만, 그 사이에 템플릿이 어디에 저장되고
스택이 어떻게 식별되는지 알아야 변경과 삭제를 안전하게 할 수 있습니다. 특히 "템플릿을 고치면 바로 반영되는가?",
"스택을 지우면 무엇이 사라지는가?"는 시험과 실무 모두에서 자주 나오는 질문입니다.

## 템플릿에서 리소스까지의 흐름

```shell
템플릿 작성(YAML/JSON)
        ↓ 업로드
S3 버킷에 템플릿 저장
        ↓ 참조
CloudFormation이 템플릿을 읽고 스택 생성
        ↓
AWS 리소스(EC2, S3, 보안 그룹 등) 생성
```

- **템플릿**: 만들 리소스를 선언한 설계도입니다.
- **스택(Stack)**: 템플릿으로 만들어진 리소스 묶음이며, CloudFormation은 스택 단위로
  생성·변경·삭제합니다.
- **리소스**: 스택이 실제로 만든 EC2 인스턴스, 보안 그룹 같은 AWS 자원입니다.

### 템플릿은 S3를 거쳐 전달됩니다

CloudFormation은 S3에 올라간 템플릿을 참조해 스택을 만듭니다. 콘솔에서 로컬 파일을 업로드하면
CloudFormation이 S3 버킷에 템플릿을 저장한 뒤 그 위치를 사용합니다.

다만 슬라이드의 "반드시 S3에 업로드한다"는 설명은 단순화된 표현입니다. 공식 문서 기준으로 AWS CLI나 API
요청 본문에 템플릿을 직접 넣을 수 있는 크기는 51,200바이트까지이고, 그보다 큰 템플릿은 S3에 올린 뒤 URL로
전달해야 합니다. S3 객체로 전달하는 템플릿은 최대 1MB입니다.

| 전달 방식                         | 최대 크기     | 사용하는 옵션                     |
| --------------------------------- | ------------- | --------------------------------- |
| 요청 본문에 직접 포함             | 51,200바이트  | `--template-body file://...`      |
| S3에 업로드한 뒤 URL로 전달       | 1MB           | `--template-url https://...`      |

- 근거:
  [CloudFormation 할당량](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html)
  (확인일: 2026-10-13)

### 템플릿은 수정하지 않고 새 버전을 올립니다

이미 사용한 템플릿을 그 자리에서 고쳐 스택에 반영하는 방식은 없습니다. 템플릿 파일을 수정한 뒤 **새 버전을 다시
업로드하고 스택 업데이트를 실행**해야 합니다. CloudFormation은 이전 템플릿과 새 템플릿을 비교해 바뀐
리소스만 변경합니다.

그래서 템플릿은 Git으로 관리하는 것이 좋습니다. 어떤 버전이 언제 배포됐는지 추적할 수 있고, 변경 내용을 코드
리뷰로 검토할 수 있기 때문입니다.

### 스택은 이름으로 구분합니다

스택은 이름으로 식별되며, 같은 계정·같은 리전 안에서 실행 중인 스택 이름은 겹칠 수 없습니다. 스택을 만들면
CloudFormation이 고유한 스택 ID(ARN)도 함께 부여합니다.

```shell
arn:aws:cloudformation:ap-northeast-2:123456789012:stack/my-web-stack/<고유 ID>
```

위 ARN의 계정 ID `123456789012`와 스택 이름은 예시 값입니다.

### 스택을 삭제하면 만든 리소스도 삭제됩니다

스택을 삭제하면 그 스택이 만든 리소스가 기본적으로 모두 삭제됩니다. 리소스를 하나씩 찾아 지울 필요가 없어 정리가
편하지만, 운영 데이터가 있는 리소스도 함께 사라질 수 있습니다.

- 지우면 안 되는 리소스는 `DeletionPolicy`로 보존하거나 스냅샷을 남길 수 있습니다.
  [DeletionPolicy](CLOUD_FORMATION_013_deletion_policy.md)에서 다룹니다.
- 스택 자체를 실수로 지우지 않도록 막는 기능은
  [Termination Protection](CLOUD_FORMATION_014_stack_policy_and_termination_protection.md)에서
  다룹니다.

## 배포 방법: 수동 방식과 자동 방식

| 구분      | 템플릿 작성                                       | 배포 방법                                      | 적합한 상황                       |
| --------- | ------------------------------------------------- | ---------------------------------------------- | --------------------------------- |
| 수동 방식 | Infrastructure Composer 또는 코드 편집기          | CloudFormation 콘솔에서 파라미터 입력 후 생성  | 학습, 일회성 테스트               |
| 자동 방식 | 코드 편집기에서 YAML 파일 작성 후 Git으로 관리     | AWS CLI 또는 CD(Continuous Delivery) 도구      | 반복 배포, 운영 환경 자동화       |

수동 방식은 화면을 보면서 진행하므로 처음 배울 때 이해하기 쉽습니다. 반면 운영 환경에서는 사람이 콘솔에서 값을
입력하다가 실수할 수 있으므로, CLI나 CI/CD 파이프라인으로 배포 과정을 자동화하는 방식을 권장합니다.

### AWS CLI로 배포하는 예시

아래 명령은 자동 방식의 기본 형태입니다. 실제 실행 순서와 결과 확인은
[첫 실습: EC2와 보안 그룹 스택](CLOUD_FORMATION_004_hands_on_first_stack.md)에서
다룹니다.

```shell
# 스택 새로 생성
aws cloudformation create-stack \
  --stack-name my-web-stack \
  --template-body file://template.yaml

# 수정한 템플릿으로 기존 스택 업데이트
aws cloudformation update-stack \
  --stack-name my-web-stack \
  --template-body file://template.yaml

# 스택이 없으면 생성, 있으면 변경 세트를 만들어 업데이트
aws cloudformation deploy \
  --stack-name my-web-stack \
  --template-file template.yaml
```

- `my-web-stack`과 `template.yaml`은 예시 이름이므로 실제 스택 이름과 템플릿 파일 경로로
  바꿉니다.
- `deploy`는 생성과 업데이트를 하나의 명령으로 처리하므로 CI/CD 스크립트에서 자주 사용합니다.
- 공식 문서:
  [AWS CLI `cloudformation deploy`](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/deploy/index.html)

### 스택 상태 읽기

배포 결과는 스택 상태로 확인합니다. 자주 보는 상태는 다음과 같습니다.

| 상태                       | 의미                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `CREATE_IN_PROGRESS`       | 스택 생성 중입니다.                                          |
| `CREATE_COMPLETE`          | 스택 생성이 끝났습니다.                                      |
| `ROLLBACK_COMPLETE`        | 생성에 실패해 만든 리소스를 되돌렸습니다. 이 스택은 삭제 후 다시 만들어야 합니다. |
| `UPDATE_COMPLETE`          | 스택 업데이트가 끝났습니다.                                  |
| `UPDATE_ROLLBACK_COMPLETE` | 업데이트에 실패해 이전 상태로 되돌렸습니다.                  |
| `DELETE_COMPLETE`          | 스택 삭제가 끝났습니다.                                      |

실패 원인은 콘솔의 **Events** 탭이나
`aws cloudformation describe-stack-events` 명령으로 확인합니다. 롤백 동작은
[Rollbacks](CLOUD_FORMATION_011_rollbacks.md)에서 자세히 다룹니다.

## 정리

- 템플릿은 S3를 거쳐 CloudFormation에 전달되며, 요청 본문에 직접 넣을 수 있는 크기는
  51,200바이트까지입니다.
- 템플릿을 바꾸려면 새 버전을 업로드하고 스택을 업데이트합니다.
- 스택은 이름으로 구분하고, 스택을 삭제하면 스택이 만든 리소스도 기본적으로 삭제됩니다.
- 학습은 콘솔 중심의 수동 방식, 운영은 CLI나 CD 도구를 쓰는 자동 방식이 적합합니다.

## 시험 대비 문제

### 문제 1

개발자가 운영 중인 CloudFormation 스택의 보안 그룹 규칙을 바꾸려고 합니다. 가장 적절한 방법은
무엇입니까?

- A. 기존에 업로드한 템플릿 파일을 S3에서 직접 열어 수정하면 스택에 자동 반영된다.
- B. 템플릿을 수정해 새 버전을 업로드하고 스택 업데이트를 실행한다.
- C. 스택을 삭제하지 않고 EC2 콘솔에서 보안 그룹 규칙만 수정한다.
- D. 같은 이름으로 새 스택을 하나 더 생성한다.

#### 정답

B

#### 해설

CloudFormation은 이미 사용한 템플릿을 고쳐 반영하는 방식이 아니라, 새 버전의 템플릿으로 스택을
업데이트하는 방식입니다. C처럼 콘솔에서 직접 수정하면 템플릿과 실제 리소스 상태가 달라지는 드리프트(drift)가
생깁니다. D는 같은 리전에서 실행 중인 스택 이름이 겹칠 수 없으므로 불가능합니다.

### 문제 2

팀이 만든 CloudFormation 템플릿 크기가 120KB입니다. AWS CLI의 `--template-body`
옵션으로 스택을 만들려고 하자 크기 제한 오류가 발생했습니다. 가장 적절한 해결 방법은 무엇입니까?

- A. 템플릿을 JSON으로 바꿔 다시 시도한다.
- B. 템플릿을 S3 버킷에 업로드한 뒤 `--template-url`로 전달한다.
- C. AWS Support에 요청해 요청 본문 크기 제한을 늘린다.
- D. 템플릿을 압축 파일로 만들어 `--template-body`로 전달한다.

#### 정답

B

#### 해설

요청 본문에 직접 넣을 수 있는 템플릿은 51,200바이트까지입니다. 그보다 큰 템플릿은 S3에 업로드한 뒤 URL로
전달해야 하며, 이 경우 최대 1MB까지 사용할 수 있습니다. 더 큰 구성은 중첩 스택(Nested Stack)으로
템플릿을 나누는 방법을 검토합니다.
