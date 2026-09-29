# ECS 배포 전략 - 블루/그린

## 🚀 Blue/Green

Blue/Green 배포는 현재 운영 중인 서비스 리비전을 Blue, 새 버전의 서비스 리비전을 Green으로 분리해
실행하는 배포 방식입니다. Green을 검증한 뒤 운영 트래픽을 Blue에서 Green으로 전환합니다.

AWS에서 가장 추천하는 방식입니다.

- 운영 중인 서비스 리비전: Blue
- 새 버전의 서비스 리비전: Green

두 버전을 동시에 실행하므로, 새 버전을 운영 트래픽에 노출하기 전에 헬스 체크와 테스트를 수행할 수 있습니다.
문제가 발생하면 Blue가 실행 중인 동안 트래픽을 되돌릴 수 있어 롤백 경로가 명확합니다.

```text
배포 전:       운영 트래픽 100% -> Blue  (v1)

Green 준비:    운영 트래픽 100% -> Blue  (v1)
                               Green (v2) 헬스 체크 및 테스트

전환 후:       운영 트래픽 100% -> Green (v2)
              Blue  (v1)는 bake time 동안 유지
```

즉, 기존 서비스(Blue)와 새 버전(Green)을, 2개를 동시에 띄웁니다.

```shell
# 기존 리비전 Blue
Task
Task
Task

# 새 리비전 Green
Task
Task
Task
```

Health Check가 끝나면 ALB가 트래픽을 한 번에 변경합니다.

```shell
Blue
100%

↓

Green
100%
```

## 동작 흐름

### Green 환경 생성

새 태스크 정의로 서비스를 업데이트하면 ECS는 Green 서비스 리비전을 만들고, 필요한 수의 Green 태스크를
시작합니다. 이 단계에서도 Blue는 계속 운영 트래픽을 처리합니다.

Green 태스크가 컨테이너 헬스 체크와 로드 밸런서 대상 그룹의 헬스 체크를 통과해야 다음 단계로
진행할 수 있습니다.

애플리케이션이 실행 상태가 되었다고 해서 즉시 요청을 받을 준비가 된 것은 아니므로,
헬스 체크는 실제 준비 완료 상태를 확인하도록 설계해야 합니다.

### 테스트와 트래픽 전환

- 테스트 리스너 또는 라우팅 규칙을 구성하면 운영 트래픽을 Blue에 유지한 채 테스트 요청만 Green으로 보낼 수 있습니다.
- 예를 들어, 특정 헤더나 별도 도메인의 요청을 Green으로 라우팅해 기능을 확인할 수 있습니다.
- 검증이 끝나면 ECS는 운영 트래픽을 Green으로 한 번에 전환합니다.
- Blue/Green 전략 자체의 운영 트래픽 전환은 All-at-once 방식입니다.
- 일부 트래픽부터 점진적으로 전환하려면 `CANARY` 또는 `LINEAR` 전략을 선택합니다.

### Bake time과 완료

Bake time은 운영 트래픽을 Green으로 전환한 뒤에도 Blue와 Green을 함께 실행하는 시간입니다.
이 기간에는 오류율, 지연 시간, 비즈니스 지표를 관찰하고 문제가 있으면 Blue로 되돌릴 수 있습니다.

Bake time이 끝나고 배포가 성공하면 ECS는 Blue 태스크를 종료합니다.
따라서 빠른 롤백이 필요할수록 관찰에 충분한 bake time을 설정해야 하지만, 그만큼 두 환경을 함께 실행하는
비용도 늘어납니다.

## 트래픽 전환에 필요한 구성

자동 트래픽 전환을 사용하려면 다음 중 하나를 구성해야 합니다.

| 구성                      | 역할                                                          | 참고 사항                                              |
| ------------------------- | ------------------------------------------------------------- | ------------------------------------------------------ |
| Application Load Balancer | HTTP/HTTPS 요청을 Blue 또는 Green 대상 그룹으로 라우팅합니다. | 테스트 리스너와 세밀한 라우팅 규칙에 적합합니다.       |
| Network Load Balancer     | TCP/UDP 수준의 요청을 전환합니다.                             | 라이프사이클 단계에 추가 대기 시간이 생길 수 있습니다. |
| Service Connect           | 서비스 간 통신의 트래픽 전환을 관리합니다.                    | 내부 서비스 통신에 적합합니다.                         |

로드 밸런서를 사용하는 경우에는 Blue와 Green 태스크를 분리해 등록할 대상 그룹이 필요합니다.
운영 요청을 받는 리스너와, 필요에 따라 Green만 검증하기 위한 테스트 리스너 또는 규칙을 준비합니다.

로드 밸런서와 Service Connect를 사용하지 않는 headless 서비스도 Blue/Green 배포를 사용할 수 있습니다.
다만 ECS가 외부 요청의 트래픽 전환을 관리하지 않으므로, 호출 측 전환 방법을 별도로 설계해야 합니다.

## 롤백과 실패 감지

Blue/Green의 핵심은 새 버전이 실패해도 기존 Blue가 남아 있다는 점입니다.

Green의 헬스 체크가 실패하거나 배포 중 설정한 alarm이 경보 상태가 되면 배포 실패로 처리하고 롤백할 수 있습니다.

- CloudWatch alarm에는 5xx 오류 비율, 지연 시간, CPU·메모리 사용률처럼 서비스 장애를 나타내는
  지표를 연결합니다.
- lifecycle hook에는 Lambda 함수 또는 일시 정지 지점을 설정해, 트래픽 전환 전후에 통합 테스트를
  실행할 수 있습니다.
- 일시 정지 지점을 사용했다면 검증을 마친 뒤 명시적으로 배포를 계속 진행해야 합니다.
  방치하면 배포가 시간 초과로 실패할 수 있습니다.
- 롤백 절차는 실제 운영 전에 테스트합니다. Blue가 종료된 뒤에는 이전 태스크 정의를 다시 배포해야 하므로
  즉시 전환할 수 없습니다.

## 장점

- Green을 운영 트래픽과 분리한 상태에서 검증할 수 있습니다.
- 트래픽 전환 전후로 Blue를 유지하므로 롤백 경로가 단순합니다.
- 배포 단계와 상태를 분리해 관찰할 수 있어 장애 원인 파악에 도움이 됩니다.
- 테스트 리스너나 lifecycle hook을 이용해 자동화된 검증 단계를 추가할 수 있습니다.

즉, 다운타임이 없고, 안전한 배포가 가능합니다.

## 단점

배포 중 서버가 2개 필요하여 비용이 증가합니다.

## 주의할 점

- Blue와 Green을 동시에 실행하므로 배포 중에는 CPU, 메모리, IP 주소 등 두 환경을 수용할 용량이 필요합니다.
- v1과 v2가 함께 실행되므로 API 계약과 데이터베이스 스키마는 두 버전에서 호환되어야 합니다.
- 세션을 태스크 메모리에만 저장하면 트래픽 전환 후 로그인 상태가 끊길 수 있습니다.
  세션 저장소를 외부화하거나 재시도 방식을 설계해야 합니다.
- Green의 헬스 체크가 너무 단순하면 실제 오류를 통과시킬 수 있고, 너무 엄격하면 정상 배포도 실패할 수 있습니다.
- Blue를 유지하는 bake time과 종료 시점을 정하지 않으면 비용과 롤백 가능 시간의 균형을 맞추기 어렵습니다.

## 언제 선택할까

다음과 같은 상황에서는 Blue/Green을 우선 검토합니다.

- 배포 실패 시 빠르게 이전 버전으로 돌아가야 하는 서비스입니다.
- 실제 요청과 유사한 트래픽으로 새 버전을 검증해야 합니다.
- API 변경이나 대규모 기능 변경처럼 배포 위험이 큰 경우입니다.
- 배포 중 두 서비스 리비전을 실행할 비용과 클러스터 용량이 있습니다.

반대로, 추가 용량 비용을 최소화해야 하거나 로드 밸런서와 Service Connect 없이 단순히 태스크를 교체하는
서비스라면 Rolling Update가 더 적합할 수 있습니다.

## 배포 전 점검 목록

- Blue와 Green을 동시에 실행할 클러스터 용량과 네트워크 IP 주소를 확인합니다.
- 대상 그룹, 리스너, 보안 그룹이 Green 태스크의 헬스 체크를 통과할 수 있게 설정됐는지 확인합니다.
- 테스트 트래픽의 라우팅 규칙과 운영 트래픽의 전환 규칙을 분리해서 확인합니다.
- CloudWatch alarm, lifecycle hook, bake time, 롤백 조건을 배포 전에 합의하고 테스트합니다.
- Blue와 Green이 같은 데이터베이스를 사용해도 안전한지, 특히 스키마와 데이터 마이그레이션 순서를 확인합니다.

## 참고 자료

- [Amazon ECS blue/green deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-blue-green.html)
- [Required resources for Amazon ECS blue/green deployments](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/blue-green-deployment-implementation.html)
- [Amazon ECS blue/green service deployments workflow](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/blue-green-deployment-how-it-works.html)
- [Amazon ECS service deployment controllers and strategies](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_service-options.html)

확인일: 2026-09-29
