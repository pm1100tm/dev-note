# ECS 배포 전략 - Overview

ECS 서비스의 배포는 새 태스크 정의를 적용하여 기존 버전의 태스크를 새 버전으로 교체하는 과정입니다.
이 문서에서는 ECS에서 선택할 수 있는 배포 방식과 서비스 특성에 따른 선택 기준을 설명합니다.

## 먼저 구분할 개념

- 현재 `ECS` 배포 컨트롤러는 Rolling Update, Blue/Green, Canary, Linear 전략을 지원합니다.
- `CODE_DEPLOY` 컨트롤러를 사용하는 기존 구성에서는 CodeDeploy가 Blue/Green 배포와 트래픽 전환을 관리합니다.
- Canary와 Linear는 별도 배포 컨트롤러가 아닙니다. 새 버전에 트래픽을 전달하는 속도를 정하는 전략입니다.

| 배포 전략      | 배포 컨트롤러            | 기존·새 버전 공존   | 적합한 상황                |
| -------------- | ------------------------ | ------------------- | -------------------------- |
| Rolling Update | `ECS`                    | 교체 중 일부 공존   | 일반적인 무중단 배포       |
| Blue/Green     | `ECS` 또는 `CODE_DEPLOY` | 트래픽 전환 전 공존 | 빠른 전환과 즉시 되돌리기  |
| Canary         | `ECS` 또는 `CODE_DEPLOY` | 점진 전환 중 공존   | 새 버전의 실제 트래픽 검증 |
| Linear         | `ECS` 또는 `CODE_DEPLOY` | 점진 전환 중 공존   | 일정한 비율로 위험을 제한  |

모든 방식이 무조건 다운타임을 없애지는 않습니다. 태스크의 정상 시작 시간, 로드 밸런서 헬스 체크,
충분한 클러스터 용량, 세션 처리 방식이 함께 갖춰져야 합니다.

## 데이터베이스 변경 시 원칙

배포 전략이 안전해도 데이터베이스 변경이 비호환이면 장애가 날 수
있습니다. v1과 v2가 공존하는 기간을 고려해 다음 순서로 변경합니다.

- 먼저 v1과 v2가 모두 읽고 쓸 수 있는 컬럼·인덱스를 추가합니다.
- 애플리케이션을 배포해 새 구조를 사용하도록 전환합니다.
- v1이 더 이상 실행되지 않고 데이터 이전이 끝난 것을 확인한 뒤
  기존 컬럼이나 제약 조건을 제거합니다.

컬럼 삭제, 타입 변경, 필수 제약 조건 추가처럼 즉시 호환성을 깨는
변경은 애플리케이션 배포와 한 번에 수행하지 않는 편이 안전합니다.

## 선택 기준

| 상황                                  | 우선 검토할 방식       | 이유                                                 |
| ------------------------------------- | ---------------------- | ---------------------------------------------------- |
| 일반적인 웹 서비스, 단순한 운영 구성  | Rolling Update         | ECS 기본 기능으로 비용과 설정을 줄일 수 있습니다.    |
| 빠른 트래픽 복구가 중요함             | Blue/Green All-at-once | 이전 대상 그룹으로 전환하는 롤백 경로가 분명합니다.  |
| 신규 기능의 실제 사용자 검증이 필요함 | Blue/Green Canary      | 작은 트래픽으로 오류 지표를 먼저 관찰할 수 있습니다. |
| 대규모 변경의 위험을 천천히 줄여야 함 | Blue/Green Linear      | 비율과 간격을 정해 점진적으로 전환할 수 있습니다.    |

## 배포 전 점검 목록

- 새 태스크 정의의 이미지 태그, 환경 변수, IAM 권한을 확인합니다.
- 컨테이너와 로드 밸런서 헬스 체크가 실제 준비 완료를 판단하는지
  확인합니다.
- 배포 중 추가 태스크를 실행할 CPU·메모리와 IP 주소 여유를
  확인합니다.
- 실패 알림을 위해 ECS service deployment 이벤트 또는 CodeDeploy
  이벤트와 CloudWatch alarm을 연결합니다.
- 롤백할 대상 태스크 정의와 데이터베이스 호환 조건을 미리
  확인합니다.

## 참고 자료

- [Amazon ECS service deployment controllers and strategies](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_service-options.html)
- [Deployment configurations for an Amazon ECS compute platform](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations-ecs.html)
- [How the Amazon ECS deployment circuit breaker detects failures](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-circuit-breaker.html)

확인일: 2026-09-29
