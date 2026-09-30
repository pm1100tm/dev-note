# Q. binpack은 EC2의 CPU·메모리를 100%까지 채우는 전략인가요?

아닙니다. `binpack`의 목적은 EC2 인스턴스에 남는 **예약 자원**을 줄여
필요한 인스턴스 수와 유휴 비용을 낮추는 것입니다.

> binpack 타스트 배치 전략은 하나의 인스턴스에 타스크를 밀집시키는 전략이며, 실제 CPU·메모리 사용률을
> 100%까지 올리는 것이 목적은 아닙니다.

여기서 예약 자원과 실제 사용률을 구분해야 합니다.

즉, ECS는 태스크를 배치할 때, 태스크 정의에 선언한 CPU와 메모리 요구 사항을 기준으로 실행 가능한
EC2 인스턴스를 찾습니다. 즉, 예약 자원이 기준이 되는 것입니다.

예를 들어 메모리가 `8 GiB`인 EC2에서 태스크마다 `1 GiB`를 예약하면,
`binpack`은 같은 EC2에 태스크를 가능한 한 많이 배치하려 합니다.

그러나 운영체제, ECS Agent, 로그·모니터링 Agent도 메모리를 사용하고,
애플리케이션의 실제 사용량은 시간에 따라 변합니다.

애플리케이션이 실행 중에 실제로 사용하는 CPU와 메모리는 CloudWatch 등의 모니터링 지표로 별도로
관찰해야 합니다.

**예약량만 합쳐서 `8 GiB`를 모두 채우면 메모리 부족으로 컨테이너가 종료될 위험이 있습니다.**

메모리는 갑작스러운 사용량 증가와 컨테이너의 메모리 제한을 함께 고려해야 하며,
부족하면 OOM 종료로 이어질 수 있습니다.

따라서, 태스크의 CPU·메모리 예약값을 실제 피크 사용량에 가깝게 잡습니다.

## 어떻게 운영해야 하나요?

태스크 정의의 CPU·메모리 예약값은 평균 사용량만이 아니라 피크 사용량과 여유를 고려해 설정합니다.

EC2의 전체 CPU·메모리를 태스크 예약값으로 100% 채우지 않고, 운영체제와
ECS Agent, 배포 중 추가 태스크를 위한 여유를 남깁니다.

CloudWatch에서 태스크와 EC2의 CPU·메모리 사용률, OOM 종료, 응답 시간을
관찰해 예약값과 인스턴스 유형을 조정합니다.

서비스 Auto Scaling으로 태스크 수를 늘릴 때는, Capacity Provider 또는
Auto Scaling Group도 새 태스크를 수용할 EC2 용량을 확보하도록 구성합니다.

가용성이 중요한 서비스는 가용 영역에 먼저 `spread`하고, 같은 가용 영역
안에서만 `binpack`하도록 두 전략을 조합합니다.

즉, `binpack`은 비용 최적화를 위한 배치 우선순위입니다. 안전한 사용률
상한은 배치 전략 하나로 정하지 않고, 태스크의 예약·제한 값, EC2 여유
용량, 실제 모니터링 결과를 함께 보고 결정합니다.

## 4 vCPU·4 GiB EC2에서는 어떻게 배치하나요?

다음은 태스크 하나가 CPU `1024` units(1 vCPU), 메모리 `1024 MiB`
(1 GiB)를 사용한다는 전제의 ECS on EC2 서비스 예시입니다. 목표는
태스크를 무조건 네 개 넣는 것이 아니라, 호스트용 1 GiB와 CPU 1 vCPU를
남기고 태스크를 최대 세 개까지만 배치하는 것입니다.

### 용량을 먼저 계산합니다

4 vCPU는 ECS CPU units로 `4096`이고, 4 GiB는 `4096 MiB`입니다.
호스트에 1 GiB를 남기면 태스크가 사용할 수 있는 메모리는 `3072 MiB`가
됩니다. 따라서 메모리가 배치 상한이 되어 인스턴스 한 대에는 태스크
세 개까지만 안전하게 들어갑니다.

| 자원   | EC2 전체 | 호스트 여유 | 태스크 배치 가능량 | 태스크당 예약 | 최대 태스크 수 |
| ------ | -------: | ----------: | -----------------: | ------------: | -------------: |
| CPU    |   4 vCPU |      1 vCPU |             3 vCPU |        1 vCPU |            3개 |
| 메모리 |    4 GiB |       1 GiB |              3 GiB |         1 GiB |            3개 |

CPU 여유는 ECS가 직접 빼는 값이 아니라, 메모리 제한으로 태스크 수를
세 개로 제한한 결과로 함께 남습니다. CPU가 실제로 1 vCPU를 넘어서
계속 사용될 수 있는 애플리케이션이라면, CPU 부하 시험 결과에 맞춰
태스크 CPU 예약을 더 크게 잡거나 인스턴스 유형을 키워야 합니다.

### 태스크 정의와 EC2를 설정합니다

태스크 정의의 컨테이너 정의에는 다음 값을 설정합니다. `memoryReservation`은
ECS가 배치할 때 차감하는 메모리이고, `memory`는 컨테이너가 넘으면
종료되는 hard limit입니다. 이 예시는 태스크가 1 GiB를 넘으면 안전하게
재시작되도록 두 값을 같게 잡습니다.

```json
{
  "cpu": 1024,
  "memoryReservation": 1024,
  "memory": 1024
}
```

EC2의 Launch Template 사용자 데이터에서 ECS Agent 설정 파일에
`ECS_RESERVED_MEMORY=1024`를 추가합니다. 이 값은 1,024 MiB를 ECS의
태스크 배치 가능 메모리에서 제외합니다. 운영체제, ECS Agent, 로그와
모니터링 Agent가 이 공간을 사용합니다.

```shell
cat <<'EOF' >> /etc/ecs/ecs.config
ECS_CLUSTER=example-cluster
ECS_RESERVED_MEMORY=1024
EOF
```

`ECS_RESERVED_MEMORY`가 적용된 새 EC2가 클러스터에 등록되면 ECS는
이 인스턴스의 가용 메모리를 약 3 GiB로 보고합니다. 콘솔의
**Cluster → Infrastructure → Container instances**에서 해당 인스턴스의
남은 CPU와 메모리를 확인한 뒤 서비스 배포를 시작합니다.

### Capacity Provider와 서비스의 값을 설정합니다

ASG를 먼저 만들고, 그 ASG를 Capacity Provider에 연결합니다.
다음은 두 가용 영역에 걸쳐 최소 두 EC2를 유지하면서도, 추가 용량이 필요하면
ECS가 ASG를 확장하게 하는 시작값입니다.

| 설정 위치               | 설정값                                        | 이유                                                                  |
| ----------------------- | --------------------------------------------- | --------------------------------------------------------------------- |
| Auto Scaling Group      | `MinSize: 2`, `MaxSize: 8`                    | 최소 두 EC2를 유지하고, 필요 시 최대 여덟 대까지 확장합니다.          |
| Capacity Provider       | managed scaling: 활성화                       | ECS가 태스크 예약량을 보고 ASG를 확장·축소합니다.                     |
| Capacity Provider       | `targetCapacity: 75`                          | 필요한 EC2 용량이 전체 ASG 용량의 약 75%가 되도록 여유를 둡니다.      |
| ECS 서비스              | `desiredCount: 3`                             | 평상시 태스크 세 개를 실행합니다. 한 EC2에 모두 들어갈 수 있습니다.   |
| ECS 서비스              | Capacity provider strategy에 위 Provider 지정 | 서비스가 만든 태스크가 이 ASG의 EC2를 사용하게 합니다.                |
| ECS 서비스 Auto Scaling | 최소 3, 최대 12 태스크                        | 부하에 따라 태스크 수를 3~12개 범위에서 조정합니다.                   |
| 배치 전략               | `binpack` + `memory`                          | 메모리가 가장 적게 남는 EC2부터 채워 한 인스턴스의 상한을 활용합니다. |

서비스가 Capacity Provider strategy를 사용하면 `launchType: EC2`는
함께 지정하지 않습니다. Capacity Provider의 managed scaling은 ASG의
원하는 용량을 직접 관리하므로, ECS가 만든 ASG 스케일링 정책을 별도로
수정하지 않습니다.

`targetCapacity: 75`는 개별 EC2의 CPU·메모리를 75%로 자르는 설정이
아닙니다. 개별 인스턴스의 안전 상한은 `ECS_RESERVED_MEMORY`와 태스크
정의의 1 GiB 예약값이 만듭니다. `targetCapacity`는 클러스터 전체에서
새 EC2를 어느 정도 미리 확보할지를 정합니다.

### 태스크 수가 늘어날 때의 배치 예시

아래 예시는 `binpack`만 사용하여 빈 인스턴스보다 이미 태스크가 있는
인스턴스를 우선 채운다고 가정합니다. `MinSize: 2`이므로 초기에는
EC2-1과 EC2-2가 있습니다. 각 EC2의 태스크 슬롯은 세 개입니다.

| 서비스 `desiredCount` | 필요한 태스크용 EC2 | Capacity Provider/ASG 동작                                                     | 배치 결과 예시                                   |
| --------------------: | ------------------: | ------------------------------------------------------------------------------ | ------------------------------------------------ |
|                     3 |                 1대 | ASG 최소값 때문에 2대를 유지합니다.                                            | EC2-1: 3개, EC2-2: 0개                           |
|                   4~6 |                 2대 | 두 EC2가 모두 필요해지면 75% 목표를 위해 ASG가 3번째 EC2를 요청할 수 있습니다. | EC2-1: 3개, EC2-2: 1~3개, EC2-3: 0개             |
|                   7~9 |                 3대 | 필요한 용량이 세 대가 되면 ASG가 4번째 EC2를 요청해 여유를 만듭니다.           | EC2-1: 3개, EC2-2: 3개, EC2-3: 1~3개, EC2-4: 0개 |
|                 10~12 |                 4대 | 75% 목표를 맞추기 위해 ASG는 일반적으로 6대까지 늘어날 수 있습니다.            | EC2-1~4: 각 3개, EC2-5~6: 0개                    |

예를 들어 Auto Scaling 정책이 `desiredCount`를 3에서 7로 높이면,
처음 세 태스크는 EC2-1에, 다음 세 태스크는 EC2-2에 배치됩니다.
일곱 번째 태스크는 EC2-3에 배치됩니다. Capacity Provider는 세 대가
모두 업무 태스크를 수용하게 된 사실을 바탕으로 ASG에 EC2-4 생성을
요청해 75% 수준의 클러스터 여유를 회복하려고 합니다.

새 EC2는 부팅, ECS Agent 시작, 클러스터 등록 과정을 거친 뒤에야
태스크를 실행할 수 있습니다. 따라서 갑작스러운 부하에도 즉시 용량이
필요한 서비스라면 `MinSize`를 더 크게 잡거나, warm pool을 별도로
검토합니다. ASG의 `MaxSize`에 도달하면 새 태스크는 `PROVISIONING`
상태에 남을 수 있으므로, 서비스 최대 태스크 수 12개에 필요한 EC2 수와
여유분을 감당할 수 있게 `MaxSize`를 정해야 합니다.

가용 영역 장애도 견뎌야 한다면 `binpack`만 쓰지 않고 배치 전략 첫 번째에
`spread`의 `attribute:ecs.availability-zone`을 둔 뒤, 두 번째에
`binpack`의 `memory`를 둡니다. 이 경우 태스크는 먼저 가용 영역에
나뉘어 배치되므로 위 표처럼 한 EC2에 완전히 몰리지 않을 수 있습니다.
대신 한 가용 영역의 EC2가 사라져도 태스크가 다른 가용 영역에 남는
장점이 있습니다.

관련 AWS 공식 문서:

- [ECS Linux 컨테이너 인스턴스 메모리 예약](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/memory-management.html),
- [EC2용 태스크 정의 메모리 설정](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters_ec2.html),
- [Capacity Provider managed scaling 동작](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/managed-scaling-behavior.html)
