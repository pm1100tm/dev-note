# 🚀 Elastic Beanstalk 배포 전략

Elastic Beanstalk의 배포 전략은 새 애플리케이션 버전을 운영 환경에 어떤 방식으로 반영할지 결정하는
옵션입니다.

배포 전략을 선택할 때는 아래 기준을 함께 봐야 합니다.

- 다운타임 허용 여부
- 배포 속도
- 추가 비용 허용 여부
- 롤백 속도
- 운영 환경 안정성
- 새 버전을 일부 사용자에게만 먼저 노출할 필요 여부

Elastic Beanstalk에서 자주 등장하는 배포 전략은 다음과 같습니다.

```shell
All at once
Rolling
Rolling with additional batches
Immutable
Blue / Green
Traffic Splitting
```

<br>

## 1. 배포 전략이 필요한 이유

애플리케이션을 배포할 때는 단순히 새 코드를 서버에 올리는 것만 생각하면 위험합니다.

운영 환경에서는 다음 문제가 생길 수 있습니다.

- 새 버전에 버그가 있을 수 있습니다.
- 배포 중 일부 인스턴스가 요청을 처리하지 못할 수 있습니다.
- 전체 용량이 일시적으로 줄어들 수 있습니다.
- 롤백이 늦어지면 장애 시간이 길어질 수 있습니다.
- 새 버전을 전체 사용자에게 바로 노출하면 영향 범위가 커질 수 있습니다.

그래서 Elastic Beanstalk는 상황에 맞게 여러 배포 방식을 제공합니다.

<br>

## 2. All at once

All at once는 모든 인스턴스에 새 버전을 한 번에 배포하는 방식입니다.

```shell
v1 v1 v1 v1
    ↓ 한 번에 교체
v2 v2 v2 v2
```

가장 빠른 배포 방식이지만, 배포 중 애플리케이션이 잠시 요청을 처리하지 못할 수 있습니다.

<br>

## 3. All at once 특징

- 가장 빠르게 배포할 수 있습니다.
- 추가 인스턴스를 만들지 않으므로 추가 비용이 거의 없습니다.
- 배포 중 다운타임이 발생할 수 있습니다.
- 개발 환경이나 빠른 반복 테스트에 적합합니다.
- 운영 환경에서는 신중하게 사용해야 합니다.

예를 들어 개발자가 기능을 빠르게 확인하는 `dev` 환경에서는 All at once가 적합할 수 있습니다.

하지만 실제 사용자가 접속하는 `prod` 환경에서는 잠깐의 다운타임도 문제가 될 수 있으므로 적합하지
않을 수 있습니다.

<br>

## 4. Rolling

Rolling은 인스턴스를 몇 개씩 묶어서 순차적으로 배포하는 방식입니다.

이때 묶음 단위를 bucket 또는 batch로 이해하면 됩니다.

```shell
v1 v1 v1 v1
    ↓ 2대 먼저 배포
v2 v2 v1 v1
    ↓ 나머지 2대 배포
v2 v2 v2 v2
```

첫 번째 묶음이 정상 상태가 되면 다음 묶음으로 넘어갑니다.

<br>

## 5. Rolling 특징

- 일부 인스턴스씩 순차적으로 배포합니다.
- bucket size를 설정할 수 있습니다.
- 추가 인스턴스를 만들지 않으므로 추가 비용이 거의 없습니다.
- 배포 중 전체 처리 용량이 줄어들 수 있습니다.
- 배포 중 v1과 v2가 동시에 서비스될 수 있습니다.
- All at once보다 배포 시간이 길어집니다.

Rolling은 다운타임을 줄이지만, 배포 중 일부 인스턴스가 업데이트되므로 애플리케이션이 평소보다
낮은 용량으로 동작할 수 있습니다.

<br>

## 6. Rolling with additional batches

Rolling with additional batches는 Rolling 방식과 비슷하지만, 새 인스턴스를 추가로 띄워서
배포 중에도 기존 용량을 유지하는 방식입니다.

```shell
기존 인스턴스: v1 v1 v1 v1
추가 인스턴스: v2 v2

배포 진행 후 기존 v1을 순차 교체
마지막에는 추가 batch 제거
```

핵심은 배포 중에도 애플리케이션이 전체 용량을 유지할 수 있다는 점입니다.

<br>

## 7. Rolling with additional batches 특징

- 배포 중에도 기존 처리 용량을 유지할 수 있습니다.
- bucket size를 설정할 수 있습니다.
- 배포 중 v1과 v2가 동시에 서비스될 수 있습니다.
- 추가 인스턴스를 사용하므로 비용이 조금 더 발생할 수 있습니다.
- 배포가 끝나면 추가 batch는 제거됩니다.
- Rolling보다 시간이 더 걸릴 수 있습니다.
- 운영 환경에 적합한 선택지가 될 수 있습니다.

운영 환경에서 다운타임은 줄이고 싶지만 Immutable만큼 큰 추가 비용을 쓰고 싶지 않을 때 고려할 수 있습니다.

<br>

## 8. Immutable

Immutable은 새 버전을 기존 인스턴스에 직접 덮어쓰지 않고, 새로운 Auto Scaling Group에
새 인스턴스를 만든 뒤 배포하는 방식입니다.

```shell
Current ASG: v1 v1 v1 v1

Temporary ASG 생성
Temporary ASG: v2 v2 v2 v2

v2 정상 확인
  ↓
트래픽 전환
  ↓
v1 종료
```

새 버전이 정상으로 확인되면 기존 인스턴스를 새 인스턴스로 교체합니다.

<br>

## 9. Immutable 특징

- 다운타임 없이 배포할 수 있습니다.
- 새 코드는 임시 Auto Scaling Group의 새 인스턴스에 배포됩니다.
- 실패 시 새 Auto Scaling Group만 종료하면 되므로 롤백이 빠릅니다.
- 배포 중 거의 두 배 용량이 필요할 수 있어 비용이 높습니다.
- 배포 시간이 가장 긴 편입니다.
- 운영 환경에 적합합니다.

Immutable은 안정성을 중시하는 운영 환경에서 강한 선택지입니다.

다만 비용과 시간이 더 필요하므로 모든 환경에 무조건 사용할 필요는 없습니다.

<br>

## 10. Blue / Green

Blue / Green은 기존 환경과 새 환경을 분리해서 운영하는 배포 방식입니다.

Elastic Beanstalk의 직접적인 단일 버튼 배포 기능이라기보다, 새 Environment를 만들고 검증 후
트래픽을 전환하는 릴리스 전략으로 이해하면 됩니다.

```shell
Blue Environment  = 현재 운영 버전 v1
Green Environment = 새 버전 v2

Green 환경 검증 완료
  ↓
CNAME Swap 또는 Route 53 전환
  ↓
Green 환경으로 사용자 트래픽 이동
```

<br>

## 11. Blue / Green 특징

- 다운타임 없이 릴리스할 수 있습니다.
- 새 환경을 독립적으로 검증할 수 있습니다.
- 문제가 있으면 기존 Blue 환경으로 되돌리기 쉽습니다.
- Route 53 weighted routing으로 일부 트래픽만 새 환경에 보낼 수 있습니다.
- Elastic Beanstalk에서는 URL swap 기능을 활용할 수 있습니다.
- 새 환경을 따로 운영하므로 비용이 추가될 수 있습니다.

Blue / Green은 운영 배포에서 영향 범위를 통제하고 싶을 때 유용합니다.

<br>

## 12. Traffic Splitting

Traffic Splitting은 새 버전에 일부 트래픽만 먼저 보내는 Canary 방식입니다.

```shell
Main ASG:      v1, 트래픽 90%
Temporary ASG: v2, 트래픽 10%

헬스 체크 통과
  ↓
v2 인스턴스를 기존 ASG로 이동
  ↓
v1 종료
```

새 버전을 임시 Auto Scaling Group에 배포하고, 일정 시간 동안 작은 비율의 트래픽만 보냅니다.

배포 상태가 정상인지 모니터링하고, 실패하면 자동 롤백이 빠르게 수행됩니다.

<br>

## 13. Traffic Splitting 특징

- Canary Testing에 적합합니다.
- 새 버전을 일부 사용자에게만 먼저 노출할 수 있습니다.
- 임시 Auto Scaling Group을 사용합니다.
- 배포 헬스가 모니터링됩니다.
- 실패 시 자동 롤백이 빠르게 수행됩니다.
- 애플리케이션 다운타임이 없습니다.
- 검증이 끝나면 새 인스턴스가 기존 Auto Scaling Group으로 이동됩니다.

Traffic Splitting은 새 버전의 위험을 작게 시작해서 검증하고 싶을 때 적합합니다.

<br>

## 14. 배포 전략 비교

| 배포 전략                       | 다운타임  | 추가 비용 | 배포 속도             | 운영 적합성 | 핵심 특징                                         |
| ------------------------------- | --------- | --------- | --------------------- | ----------- | ------------------------------------------------- |
| All at once                     | 있음      | 없음      | 가장 빠름             | 낮음        | 전체 인스턴스를 한 번에 교체합니다.               |
| Rolling                         | 거의 없음 | 없음      | 느림                  | 중간        | 일부 인스턴스씩 교체하지만 용량이 줄 수 있습니다. |
| Rolling with additional batches | 거의 없음 | 조금 있음 | 느림                  | 높음        | 추가 인스턴스로 기존 용량을 유지합니다.           |
| Immutable                       | 없음      | 높음      | 가장 느림             | 높음        | 새 ASG에 배포 후 교체합니다.                      |
| Blue / Green                    | 없음      | 있음      | 환경 구성에 따라 다름 | 높음        | 새 환경을 만들고 트래픽을 전환합니다.             |
| Traffic Splitting               | 없음      | 있음      | 중간                  | 높음        | 일부 트래픽만 새 버전에 보냅니다.                 |

<br>

## 15. 어떤 배포 전략을 선택해야 합니까?

개발 환경에서는 All at once가 적합할 수 있습니다.

```shell
목표: 빠른 테스트
선택: All at once
```

운영 환경에서 비용을 어느 정도 허용하면서 용량을 유지하고 싶다면
Rolling with additional batches를 고려할 수 있습니다.

```shell
목표: 운영 배포, 기존 용량 유지
선택: Rolling with additional batches
```

운영 환경에서 안전한 롤백과 무중단 배포가 중요하면 Immutable을 고려할 수 있습니다.

```shell
목표: 무중단, 빠른 롤백
선택: Immutable
```

새 버전을 일부 트래픽에만 먼저 노출하고 싶으면 Traffic Splitting을 고려할 수 있습니다.

```shell
목표: Canary 배포
선택: Traffic Splitting
```

새 환경을 독립적으로 검증한 뒤 전체 트래픽을 전환하고 싶으면 Blue / Green을 고려할 수 있습니다.

```shell
목표: 독립 환경 검증 후 전환
선택: Blue / Green
```

<br>

## 16. DVA-C02 시험 포인트

- All at once는 가장 빠르지만 다운타임이 발생할 수 있습니다.
- All at once는 개발 환경의 빠른 반복에 적합합니다.
- Rolling은 일부 인스턴스씩 배포하며, 배포 중 용량이 줄어들 수 있습니다.
- Rolling은 배포 중 두 버전이 동시에 실행될 수 있습니다.
- Rolling with additional batches는 추가 인스턴스를 사용해 기존 용량을 유지합니다.
- Immutable은 새 Auto Scaling Group에 새 버전을 배포한 뒤 정상 확인 후 교체합니다.
- Immutable은 다운타임이 없고 롤백이 빠르지만 비용이 높고 시간이 오래 걸립니다.
- Blue / Green은 새 Environment를 만들고 CNAME Swap 또는 Route 53 전환을 사용합니다.
- Traffic Splitting은 Canary Testing 방식이며 일부 트래픽만 새 버전에 보냅니다.
- Traffic Splitting은 실패 시 자동 롤백이 빠르게 수행됩니다.

<br>

## 17. 시험 예상 문제

### 문제 1

개발 환경에서 Elastic Beanstalk 애플리케이션을 가장 빠르게 배포하려고 합니다.
잠깐의 다운타임은 허용됩니다. 가장 적절한 배포 전략은 무엇입니까?

- A. All at once
- B. Immutable
- C. Traffic Splitting
- D. Blue / Green

<br>

### 정답

A. All at once

<br>

### 해설

All at once는 모든 인스턴스를 한 번에 새 버전으로 교체하는 방식입니다.
가장 빠르게 배포할 수 있지만, 배포 중 애플리케이션 다운타임이 발생할 수 있습니다.
따라서 빠른 반복이 중요한 개발 환경에 적합합니다.

<br>

### 문제 2

운영 환경에서 배포 중에도 기존 처리 용량을 유지하고 싶습니다. 추가 인스턴스를 임시로 사용해
Rolling 방식으로 배포하려고 합니다. 가장 적절한 배포 전략은 무엇입니까?

- A. All at once
- B. Rolling
- C. Rolling with additional batches
- D. Single Instance

<br>

### 정답

C. Rolling with additional batches

<br>

### 해설

Rolling with additional batches는 기존 인스턴스를 바로 줄이지 않고 추가 batch를 생성해
배포를 진행합니다. 이 방식은 배포 중에도 애플리케이션이 기존 용량을 유지할 수 있습니다.
추가 인스턴스가 필요하므로 비용이 조금 더 발생할 수 있으며, 배포가 끝나면 추가 batch는 제거됩니다.

<br>

### 문제 3

새 버전을 기존 인스턴스에 직접 배포하지 않고, 임시 Auto Scaling Group에 새 인스턴스를 만든 뒤
정상 확인 후 교체하려고 합니다. 실패 시 새 Auto Scaling Group만 종료해 빠르게 롤백하고 싶습니다.
가장 적절한 배포 전략은 무엇입니까?

- A. Immutable
- B. All at once
- C. Rolling
- D. EB CLI

<br>

### 정답

A. Immutable

<br>

### 해설

Immutable은 새 버전을 임시 Auto Scaling Group의 새 인스턴스에 배포합니다.
정상 상태가 확인되면 기존 인스턴스를 교체하고, 실패하면 새 Auto Scaling Group을 종료해 빠르게
롤백할 수 있습니다. 무중단 배포와 빠른 롤백에 강하지만 비용이 높고 시간이 오래 걸릴 수 있습니다.

<br>

### 문제 4

새 버전을 일부 사용자에게만 먼저 노출하고, 문제가 없으면 전체 배포로 전환하려고 합니다.
실패 시 자동 롤백도 필요합니다. 가장 적절한 Elastic Beanstalk 배포 전략은 무엇입니까?

- A. Traffic Splitting
- B. All at once
- C. Rolling
- D. Single Instance

<br>

### 정답

A. Traffic Splitting

<br>

### 해설

Traffic Splitting은 Canary Testing 방식입니다.
새 버전을 임시 Auto Scaling Group에 배포하고, 작은 비율의 트래픽만 새 버전으로 보냅니다.
배포 상태를 모니터링하며, 실패가 감지되면 자동 롤백이 빠르게 수행됩니다.

<br>

## 요약

- Elastic Beanstalk 배포 전략은 새 버전을 어떤 위험 수준과 속도로 반영할지 결정하는 옵션입니다.
- 개발 환경에서는 빠른 All at once가 적합할 수 있습니다.
- 운영 환경에서는 Rolling with additional batches, Immutable, Blue / Green,
  Traffic Splitting처럼 다운타임과 장애 영향을 줄이는 방식을 고려해야 합니다.
- DVA-C02에서는 각 전략의 다운타임, 비용, 롤백, 운영 적합성을 비교해서 기억해야 합니다.
