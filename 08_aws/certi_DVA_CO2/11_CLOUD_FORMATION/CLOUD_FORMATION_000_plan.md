# CloudFormation 학습 계획

## 학습 목적

AWS CloudFormation의 개념을 이해하고, 직접 템플릿을 작성해 스택을 만들고 지우는 데까지 익히는 것이
목표입니다. AWS Certified Developer – Associate(DVA-C02) 시험에서 자주 묻는 함수,
옵션, 보호 장치도 함께 정리합니다.

- 참고 자료: AWS Certified Developer Slides v46, 380~427쪽 (AWS
  CloudFormation 단원)
- 확인 기준: 버전에 따라 달라질 수 있는 내용은
  [AWS CloudFormation 공식 문서](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)를
  기준으로 확인합니다.
- 확인일: 2026-10-13

## 챕터 구성

| 번호 | 문서                                                                                                    | 다루는 내용                                                  | 슬라이드 |
| ---- | ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ | -------- |
| 001  | [CloudFormation 개요](CLOUD_FORMATION_001_overview.md)                                                  | 선언형 IaC, 장점, Infrastructure Composer (기존 문서 유지)   | 380~384  |
| 002  | [동작 방식과 배포 방법](CLOUD_FORMATION_002_how_it_works_and_deploy.md)                                 | 템플릿 업로드, 스택, 수동 배포와 자동 배포                   | 385~386  |
| 003  | [템플릿 구성 요소와 YAML 기초](CLOUD_FORMATION_003_template_structure_and_yaml.md)                      | 템플릿 섹션, YAML 문법                                       | 387, 389 |
| 004  | [첫 실습: EC2와 보안 그룹 스택](CLOUD_FORMATION_004_hands_on_first_stack.md)                            | AWS CLI로 스택 생성·변경·삭제                                | 388, 392 |
| 005  | [Resources](CLOUD_FORMATION_005_resources.md)                                                           | 리소스 타입, 공식 문서 읽는 법, Macro와 Custom Resource 개념 | 390~393  |
| 006  | [Parameters](CLOUD_FORMATION_006_parameters.md)                                                         | 파라미터 타입과 제약 조건, `Ref`, Pseudo Parameters          | 394~399  |
| 007  | [Mappings](CLOUD_FORMATION_007_mappings.md)                                                             | `Fn::FindInMap`, Mappings와 Parameters 선택 기준             | 400~402  |
| 008  | [Outputs와 Cross-Stack Reference](CLOUD_FORMATION_008_outputs_and_cross_stack.md)                       | `Export`, `Fn::ImportValue`, 스택 간 참조 제약               | 403~405  |
| 009  | [Conditions](CLOUD_FORMATION_009_conditions.md)                                                         | 조건 정의와 적용, 조건 함수                                  | 406~408  |
| 010  | [Intrinsic Functions](CLOUD_FORMATION_010_intrinsic_functions.md)                                       | `Ref`, `Fn::GetAtt`, `Fn::Sub`, `Fn::Base64` 등 내장 함수    | 409~415  |
| 011  | [Rollbacks](CLOUD_FORMATION_011_rollbacks.md)                                                           | 생성·업데이트 실패 시 롤백, `ContinueUpdateRollback`         | 416      |
| 012  | [Service Role과 Capabilities](CLOUD_FORMATION_012_service_role_and_capabilities.md)                     | `iam:PassRole`, `CAPABILITY_IAM`, `CAPABILITY_AUTO_EXPAND`   | 417~418  |
| 013  | [DeletionPolicy](CLOUD_FORMATION_013_deletion_policy.md)                                                | `Delete`, `Retain`, `Snapshot`                               | 419~421  |
| 014  | [Stack Policy와 Termination Protection](CLOUD_FORMATION_014_stack_policy_and_termination_protection.md) | 업데이트 보호와 삭제 보호                                    | 422~423  |
| 015  | [Custom Resources](CLOUD_FORMATION_015_custom_resources.md)                                             | `ServiceToken`, Lambda 기반 사용자 정의 리소스               | 424~426  |
| 016  | [StackSets](CLOUD_FORMATION_016_stacksets.md)                                                           | 여러 계정·리전에 같은 스택 배포                              | 427      |

## 학습 순서

### 1단계: 기본 개념과 첫 실습 (001~004)

CloudFormation이 무엇이고 템플릿이 어떻게 스택이 되는지 이해합니다. 마지막에는 AWS CLI로 작은 스택을
직접 만들고 삭제해 봅니다.

### 2단계: 템플릿 섹션 (005~010)

`Resources`, `Parameters`, `Mappings`, `Outputs`, `Conditions` 섹션과
내장 함수를 차례로 익힙니다. 시험 문제의 상당수가 이 범위에서 나오므로, 각 섹션의 역할과 함수 이름을 정확히
구분합니다.

### 3단계: 운영과 보호 장치 (011~014)

배포 실패 시 롤백, 권한 위임, 삭제·업데이트 보호처럼 운영 환경에서 사고를 막는 기능을 정리합니다.

### 4단계: 확장 기능 (015~016)

CloudFormation이 직접 지원하지 않는 작업을 Lambda로 처리하는 Custom Resource와, 여러
계정·리전에 한 번에 배포하는 StackSets를 다룹니다.

## 각 챕터 작성 형식

- 배경과 필요한 이유: 이 기능이 없으면 어떤 불편이 있는지 먼저 설명합니다.
- 동작 원리: 용어의 뜻과 동작 흐름을 짧게 정리합니다.
- 작은 예시: YAML 템플릿 조각이나 AWS CLI 명령을 함께 둡니다.
- 실무 활용과 주의 사항: 언제 쓰는지, 무엇을 조심해야 하는지 적습니다.
- 시험 대비 문제: 챕터마다 2문제와 정답, 해설을 둡니다.

## 참고 사항

- 슬라이드 원문을 옮기지 않고, 내용을 이해한 뒤 직접 설명하는 방식으로 작성합니다.
- 시험 대비 문제는 슬라이드 내용을 바탕으로 새로 만든 연습 문제이며, 실제 시험 문제가 아닙니다.
- 슬라이드 작성 시점 이후 바뀐 내용은 공식 문서 기준으로 보완하고, 차이가 있으면 본문에 표시합니다.
- 실습은 비용이 발생할 수 있으므로, 실습이 끝나면 스택 삭제와 리소스 정리를 확인합니다.
