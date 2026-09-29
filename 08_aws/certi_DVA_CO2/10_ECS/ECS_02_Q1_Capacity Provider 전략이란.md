# Capacity Provider 전략이란 무엇인가요?

Capacity Provider는 ECS 태스크를 실행할 컴퓨팅 용량을 나타냅니다.

Capacity Provider 전략은 Service 또는 독립 Task가 여러 Provider 중
어느 곳에, 몇 개의 태스크를 배치할지 정하는 규칙입니다.

전략에는 `base`와 `weight`를 지정할 수 있습니다.

| 항목     | 의미                                                   |
| -------- | ------------------------------------------------------ |
| `base`   | 특정 Provider에 먼저 배치할 최소 태스크 수입니다.      |
| `weight` | `base`를 충족한 뒤 Provider 간 태스크 배분 비율입니다. |

예를 들어 `FARGATE`의 `base`를 2로 설정하고, `FARGATE`와
`FARGATE_SPOT`의 `weight`를 각각 1과 3으로 설정할 수 있습니다.

이 경우 처음 2개는 Fargate 온디맨드에 배치하고, 그 이후 태스크는
두 Provider의 가중치에 따라 배분합니다.

```text
Desired Count = 6

FARGATE base = 2
FARGATE weight = 1
FARGATE_SPOT weight = 3

처음 2개 → FARGATE
남은 4개 → FARGATE와 FARGATE_SPOT에 1:3 비율로 배분
```

- `FARGATE_SPOT`은 비용을 줄일 수 있지만 AWS가 용량을 회수하면 태스크가 중단될 수 있습니다.
  따라서 중단되어도 다시 실행할 수 있는 워커나 비핵심 작업에 적합합니다.
- 반드시 실행되어야 하는 API는 `FARGATE`에 최소 수량을 두거나, 중단을 견딜 수 있게 설계해야 합니다.

**EC2 Launch Type**에서는 Auto Scaling Group(ASG)을 연결한 ASG Capacity Provider를 사용합니다.
Service Auto Scaling이 태스크 수를 늘려도 EC2 용량이 부족하면 태스크가 `PENDING` 상태에 남을 수 있으므로,
ASG Capacity Provider의 managed scaling으로 기반 인스턴스 용량도 함께 고려합니다.

하나의 Capacity Provider 전략에는 Fargate 계열과 ASG 계열 Provider를 함께 섞을 수 없습니다.

## `launchType`을 명시한 요청에는 `capacityProviderStrategy`를 함께 지정 불가

ECS Service나 `RunTask`를 실행할 때, 태스크를 어디에서 실행할지 정하는 방법은 둘 중 하나만 선택해야
한다는 뜻입니다.

- `launchType`: 단일 실행 환경을 직접 지정합니다.
- `capacityProviderStrategy`: 실행 환경과 배분 규칙을 전략으로 지정합니다.

예를 들어 아래처럼 `launchType`만 지정할 수 있습니다.

```json
{
  "launchType": "FARGATE"
}
```

이 경우 “모든 태스크를 일반 Fargate에서 실행합니다”라는 뜻입니다.

반면 다음처럼 Capacity Provider 전략만 지정할 수도 있습니다.

```json
{
  "capacityProviderStrategy": [
    {
      "capacityProvider": "FARGATE",
      "weight": 1
    },
    {
      "capacityProvider": "FARGATE_SPOT",
      "weight": 3
    }
  ]
}
```

이 경우 ECS가 `FARGATE`와 `FARGATE_SPOT` 사이에 태스크를 가중치에 따라 배치합니다.

둘을 함께 쓰면 충돌합니다.

```json
{
  "launchType": "FARGATE",
  "capacityProviderStrategy": [
    {
      "capacityProvider": "FARGATE_SPOT",
      "weight": 1
    }
  ]
}
```

위 설정은 “일반 Fargate에서만 실행하라”와 “Fargate Spot에도 배치하라”를 동시에 말하므로 ECS가 거부합니다.

실무에서는 단순히 Fargate만 쓸 때도 `launchType`을 사용할 수 있지만, Spot 혼합,
EC2 Auto Scaling Group 연동, `base`·`weight` 배분처럼 유연한 운영이 필요하면
`capacityProviderStrategy`를 사용합니다.
