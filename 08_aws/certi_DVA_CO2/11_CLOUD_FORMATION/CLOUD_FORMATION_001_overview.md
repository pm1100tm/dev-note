# AWS CloudFormation 개요

## CloudFormation이란?

AWS CloudFormation은 AWS 인프라 구성을 템플릿 파일로 작성하고,
그 파일대로 리소스를 생성·변경·삭제하는 서비스입니다.

여기서 인프라는 EC2 인스턴스, 보안 그룹, S3 버킷, 로드 밸런서처럼 애플리케이션을 실행하기 위해
필요한 AWS 자원입니다.

템플릿은 YAML 또는 JSON으로 작성하는 설계도이며, 실제로 생성된 리소스 묶음은
**스택(Stack)** 이라고 부릅니다.

- [AWS 공식 문서](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-overview.html)

## 선언형 방식으로 구성하기

CloudFormation은 **무엇을 만들지** 선언하는 방식입니다.
리소스를 만드는 API 호출 순서를 직접 작성하지 않아도,
CloudFormation이 의존 관계를 파악해 필요한 순서로 처리합니다.

예를 들어 템플릿에 다음 구성을 정의할 수 있습니다.

- 보안 그룹 1개
- 해당 보안 그룹을 사용하는 EC2 인스턴스 2개
- 각 EC2에 연결할 Elastic IP 2개
- S3 버킷 1개
- EC2 앞에서 요청을 분산할 로드 밸런서 1개

스택을 생성하면 CloudFormation이 정의한 설정을 바탕으로 리소스를 만들고 연결합니다.

다만 모든 의존 관계를 자동으로 알 수 있는 것은 아니므로, 필요한 경우 `DependsOn`으로 생성 순서를
명시할 수 있습니다.

## 왜 사용하나요?

| 장점                   | 의미                                                                                                                                                                       |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Infrastructure as Code | 콘솔에서 수동으로 만든 구성을 템플릿으로 관리합니다. Git으로 버전을 남기고 코드 리뷰로 변경을 검토할 수 있습니다.                                                          |
| 반복 가능한 배포       | 같은 템플릿으로 개발·테스트·운영 환경을 비슷한 형태로 다시 만들 수 있습니다.                                                                                               |
| 변경 관리              | 스택 단위로 리소스를 생성·변경·삭제합니다. 변경 전에는 Change Set으로 예상 변경 사항을 검토할 수 있습니다.                                                                 |
| 비용 파악              | 스택 태그를 지원하는 리소스에는 공통 태그를 전달할 수 있어, 비용 분석에서 스택 또는 서비스별 비용을 구분하는 데 도움이 됩니다. 템플릿의 월 예상 비용도 조회할 수 있습니다. |
| 관심사 분리            | VPC, 네트워크, 애플리케이션처럼 책임이 다른 구성을 별도 스택으로 나눌 수 있습니다.                                                                                         |

- 개발 환경은 사용하지 않는 시간에 스택을 삭제하고 업무 시작 시 다시 생성하도록 자동화할 수 있습니다.
- 단, 데이터가 필요한 RDS나 S3는 삭제 정책과 백업을 먼저 확인해야 합니다.

- CloudFormation 서비스 자체는 무료입니다.
- 그러나, 템플릿으로 생성한 EC2, S3 등의 AWS 리소스에는 각각의 요금이 부과됩니다.

[템플릿 비용 추정 API](https://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_EstimateTemplateCost.html)

## Infrastructure Composer

- AWS Infrastructure Composer는 템플릿을 화면에서 시각적으로 구성하는 도구입니다.
- 예전 이름은 Application Composer입니다.

리소스 카드를 끌어다 놓고 연결하면 아키텍처를 확인하면서 CloudFormation 템플릿을 작성하거나 수정할 수 있습니다.
처음에는 구조를 이해하는 데 유용하지만, 생성된 템플릿도 Git으로 관리하고 배포 전 변경 내용을 검토해야 합니다.

[Infrastructure Composer 공식 문서](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/infrastructure-composer-for-cloudformation.html)

## 템플릿과 스택의 관계

```text
템플릿(YAML 또는 JSON) 작성
        ↓
스택 생성
        ↓
CloudFormation이 AWS 리소스를 생성·연결
        ↓
스택 단위로 변경 또는 삭제
```

- 하나의 템플릿으로 개발용 스택과 운영용 스택을 각각 만들 수 있습니다.
- 환경마다 달라지는 값은 파라미터로 받아 템플릿을 재사용합니다.

## 함께 기억할 점

- 템플릿의 `Resources` 섹션은 필수이며, 생성할 AWS 리소스와 설정을 정의합니다.
- 스택을 삭제하면 기본적으로 스택이 관리하는 리소스도 함께 삭제됩니다.
- 운영 데이터가 있는 리소스는 `DeletionPolicy`, 백업, 변경 영향도를 확인한 뒤 배포합니다.
- 공개 예제 템플릿은 그대로 운영에 사용하지 말고 보안·비용·권한 설정을 검토합니다.
