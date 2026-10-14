# EKS KEDA cron 트리거 배포 확인 방법

운영 EKS 클러스터에 특정 날짜용 KEDA cron 트리거를 추가하는 배포를 앞두고, 배포가 제대로 반영됐는지와
이벤트 시간에 실제로 파드가 뜨는지를 어떻게 확인할지 정리한 업무 기록입니다.

> 클러스터, 네임스페이스, EC2 이름은 비식별 예시 값으로 바꿨습니다. 실제로 사용할 때는 `my-prd-eks`,
> `my-app`, `my-prd-shared-gpu`를 각자 환경의 이름으로 교체합니다. 워크로드 이름과 트리거
> 이름, 일정은 기록 그대로입니다.

## 요청과 상황

### 요청

- 공유된 일정 안내에 "`*-base` 파드 4개가 Running인지 확인"이라는 항목이 있었고, 콘솔에서 이
  파드를 어떻게 찾는지 확인하고자 했습니다.
- 이어서 콘솔만으로는 확인이 어려운 ScaledObject 설정을 `kubectl`로 확인하는 방법과, 각 명령이
  무엇을 하는지 알고자 했습니다.

### 환경

| 항목            | 값                                             |
| --------------- | ---------------------------------------------- |
| 클러스터        | EKS `my-prd-eks` (리전 `ap-northeast-2`, 서울) |
| 네임스페이스    | `my-app`                                       |
| 오토스케일      | KEDA ScaledObject, cron 트리거                 |
| 공유 GPU 노드   | EC2 `my-prd-shared-gpu`                        |
| 매니페스트 위치 | `deploy/eks/k8s-prd/deployment-*-base.yaml`    |

### 스케일 구조

워크로드는 상시 운영용 base 4개와 부하 대응용 burst 4개로 나뉘고, 각각 같은 이름의 ScaledObject가
있습니다.

| 구분  | Deployment / ScaledObject                                                     |
| ----- | ----------------------------------------------------------------------------- |
| base  | `stt-worker-base`, `export-worker-base`, `media-models-base`, `qwen-tts-base` |
| burst | `stt-worker`, `export-worker`, `media-models`, `qwen-tts`                     |

- 평일 업무 시간대(biz, 07:30~20:00)에는 cron 트리거로 base 파드 4개가 떠 있고,
  20:00에 cron 구간이 끝나면 base 파드가 0개가 되면서 공유 GPU EC2도 내려갑니다.
- 이번 PR은 업무 시간 밖의 이벤트 일정에 맞춰 파드를 띄우려고 `evt1014`, `evt1025` 트리거를
  추가하는 변경입니다.

### 일정

| 시점              | 내용                                |
| ----------------- | ----------------------------------- |
| 10/13 20:00       | PR 배포 (biz cron 종료 시각과 같음) |
| 10/14 21:00~21:30 | `evt1014` 트리거 첫 동작            |
| 10/25 18:10쯤     | `evt1025` 동작 확인 시점            |

`evt1025`의 정확한 시작·종료 시각은 기록에 남아 있지 않고, 확인 시점만 남아 있습니다.

## 확인 1: 콘솔에서 base 파드 찾기

AWS 콘솔의 다음 위치에서 `*-base` 파드 4개가 Running인지 확인했습니다.

```shell
EKS > my-prd-eks > 리소스 > Pods
```

- 네임스페이스 필터에 `my-app`, 이름 필터에 `base`를 넣으면 4개만 남습니다.
- 파드 이름은 `<Deployment 이름>-<ReplicaSet 해시>-<임의 문자열>` 형태라서 콘솔에서는
  아래처럼 보입니다.

| Deployment           | 파드 이름 형태                   |
| -------------------- | -------------------------------- |
| `export-worker-base` | `export-worker-base-xxxxx-yyyyy` |
| `media-models-base`  | `media-models-base-xxxxx-yyyyy`  |
| `qwen-tts-base`      | `qwen-tts-base-xxxxx-yyyyy`      |
| `stt-worker-base`    | `stt-worker-base-xxxxx-yyyyy`    |

스크린샷을 찍은 18:15는 평일 업무 시간대 안이므로 4개가 떠 있는 것이 정상입니다.

## 확인 2: 시점별로 무엇을 볼지 나누기

배포 시각이 biz cron이 끝나는 20:00과 겹치므로, 배포 직후에 파드를 보면 오히려 0개가 정상입니다.
그래서 시점마다 확인 대상을 나눴습니다.

| 시점            | 확인 대상                                               | 정상 결과                                |
| --------------- | ------------------------------------------------------- | ---------------------------------------- |
| 10/13 배포 직후 | ScaledObject에 `evt1014`, `evt1025` 트리거가 들어갔는지 | 8개 ScaledObject 모두에 두 트리거가 보임 |
| 10/14 21:10쯤   | EC2 `my-prd-shared-gpu` 상태, base 파드 상태            | EC2 '실행 중', base 파드 4개 Running     |
| 10/25 18:10쯤   | 위와 같음                                               | 위와 같음                                |

- 배포 직후에는 파드나 EC2가 아니라 **설정이 반영됐는지**를 봅니다.
- 이벤트 시간에는 원래 일정 안내대로 **EC2와 파드가 실제로 떴는지**를 봅니다.

## 확인 3: kubectl 실행 환경 준비

ScaledObject는 콘솔에서 보기 어려워 `kubectl`로 확인하기로 했습니다. 먼저 로컬 환경을
점검했습니다.

### 로컬 상태

| 항목          | 상태                                                    |
| ------------- | ------------------------------------------------------- |
| aws CLI       | 설치됨 (v2.37.7, `/opt/homebrew/bin/aws`)               |
| kubectl       | 설치됨 (Docker Desktop 포함본, `~/.docker/bin/kubectl`) |
| AWS 자격 증명 | 없음. `~/.aws/credentials`가 없음                       |
| kubeconfig    | 없음. `~/.kube`가 없음                                  |

`aws sts get-caller-identity`를 실행하면 자격 증명이 없어 `NoCredentials`
오류가 났습니다. 도구 설치는 필요 없었고, 막힌 부분은 자격 증명이었습니다.

### 선택: AWS CloudShell

로컬에 access key를 새로 만드는 대신 AWS CloudShell을 쓰기로 했습니다.

- 콘솔 상단 검색창 오른쪽의 `>_` 아이콘을 누르면 열립니다.
- 콘솔에 로그인한 권한을 그대로 쓰므로 키를 만들거나 설정할 필요가 없습니다.
- aws CLI와 `kubectl`이 미리 설치되어 있습니다.
  ([CloudShell 기본 설치 소프트웨어](https://docs.aws.amazon.com/cloudshell/latest/userguide/vm-specs.html#pre-installed-software))

단, 콘솔 로그인 권한이 EKS 클러스터에 접근할 수 있도록 등록되어 있어야 `kubectl` 조회가 됩니다. 이번
기록에는 이 권한 확인 결과가 남아 있지 않습니다.

## 확인 4: 명령별 의미와 기대 결과

CloudShell에서 아래 순서로 실행합니다. 모두 조회 명령이며 클러스터 상태를 바꾸지 않습니다.

### kubeconfig 생성

```shell
aws eks update-kubeconfig --region ap-northeast-2 --name my-prd-eks
```

- `kubectl`이 어느 클러스터에 접속할지 정하는 설정 파일 `~/.kube/config`를 만들거나
  갱신합니다.
- `--region`은 클러스터가 있는 리전, `--name`은 EKS 클러스터 이름입니다.
- 클러스터 주소, 인증서, 인증 방식(현재 AWS 로그인 정보로 토큰 발급)이 기록됩니다.
- 클러스터를 바꾸지 않고 CloudShell의 로컬 설정 파일만 씁니다.
- 성공하면 다음과 같은 메시지가 나옵니다.
  `Added new context arn:aws:eks:...:cluster/my-prd-eks to ...`

참고: [aws eks update-kubeconfig](https://docs.aws.amazon.com/cli/latest/reference/eks/update-kubeconfig.html)

### 접속 확인

```shell
# CloudShell에 설치된 kubectl 버전만 출력
kubectl version --client

# kubectl 버전과 클러스터(서버) 버전을 함께 출력
kubectl version
```

- `kubectl version`에 `Server Version`까지 나오면 클러스터 접속과 인증이 성공했다는
  뜻이므로, kubeconfig를 만든 직후 접속 확인용으로 씁니다.
- `kubectl`은 클러스터 버전과 마이너 버전 차이가 ±1 이내일 때 지원됩니다.
  ([Version Skew Policy](https://kubernetes.io/releases/version-skew-policy/#kubectl))

### ScaledObject 목록 조회

```shell
kubectl get scaledobject -n my-app
```

- `my-app` 네임스페이스의 KEDA ScaledObject(오토스케일 규칙) 목록을 조회합니다.
- 결과에 base 4개와 burst 4개가 모두 있어야 하고, 그 밖의 항목(예: `bgm-http`)도 함께
  나옵니다.

| 컬럼       | 의미                                |
| ---------- | ----------------------------------- |
| `READY`    | 규칙이 정상인지. `True`여야 합니다. |
| `ACTIVE`   | 지금 파드를 띄우는 중인지           |
| `TRIGGERS` | 트리거 종류                         |

### 트리거 상세 조회

처음에는 YAML 전체에서 `evt`가 들어간 부분만 잘라 보는 방법을 썼습니다.

```shell
kubectl get scaledobject stt-worker-base -n my-app -o yaml | grep -A6 evt
```

- `get scaledobject stt-worker-base`는 ScaledObject 하나만 조회합니다.
- `-o yaml`은 목록이 아니라 전체 설정을 YAML로 출력합니다.
- `grep -A6 evt`는 `evt`가 들어간 줄과 그 뒤 6줄만 보여 줍니다.
- 매니페스트 기준으로 기대한 `evt1014` 트리거는 다음과 같습니다. `evt1025`도 같은 형태입니다.

```yaml
- type: cron
  name: evt1014
  metadata:
    timezone: Asia/Seoul
    start: 0 21 14 10 *
    end: 30 21 14 10 *
    desiredReplicas: '1'
```

cron 표현식은 `분 시 일 월 요일` 순서이므로, 이 트리거는 10월 14일 21:00부터 21:30까지
replica를 1로 유지합니다.
([KEDA Cron scaler](https://keda.sh/docs/latest/scalers/cron/))

나머지 ScaledObject도 이름만 바꿔 같은 방법으로 확인합니다.

```shell
kubectl get scaledobject stt-worker-base -n my-app -o yaml | grep -A6 evt
kubectl get scaledobject export-worker-base -n my-app -o yaml | grep -A6 evt
kubectl get scaledobject media-models-base -n my-app -o yaml | grep -A6 evt
kubectl get scaledobject qwen-tts-base -n my-app -o yaml | grep -A6 evt
```

### jsonpath로 트리거만 한 줄씩 보기

`grep`은 출력 형태에 따라 결과가 흔들릴 수 있어, 트리거 이름과 시작·종료 시각만 뽑는 `jsonpath`
방식도 정리했습니다.

```shell
kubectl get scaledobject qwen-tts-base -n my-app \
  -o jsonpath='{range .spec.triggers[*]}{.name}{"\t"}{.metadata.start}{" ~ "}{.metadata.end}{"\n"}{end}'
```

- `.spec.triggers[*]`의 각 트리거마다 이름, `start`, `end`를 한 줄로 출력합니다.

8개 ScaledObject를 한 번에 확인할 때는 반복문을 씁니다.

```shell
for so in stt-worker-base export-worker-base media-models-base qwen-tts-base \
  stt-worker export-worker media-models qwen-tts; do
  echo "== $so"
  kubectl get scaledobject "$so" -n my-app \
    -o jsonpath='{range .spec.triggers[*]}{.name}{"\t"}{.metadata.start}{" ~ "}{.metadata.end}{"\n"}{end}'
done
```

각 ScaledObject 아래에 `evt1014`, `evt1025` 줄이 보이면 설정이 반영된 것입니다.

## 주의할 점

- 배포 전에는 트리거 상세 조회 결과에 `evt`가 나오지 않는 것이 정상입니다. 트리거는 20시 배포 뒤에
  생깁니다. 배포 전에 미리 테스트할 때는 kubeconfig 생성, `kubectl version`,
  ScaledObject 목록 조회까지만 결과가 나오는지 봅니다.
- 배포 직후 base 파드가 0개이고 공유 GPU EC2가 내려가 있어도 장애가 아닙니다. biz cron이
  20:00에 끝났기 때문입니다.
- `kubectl get -o yaml` 출력은 키가 알파벳 순으로 정렬되어 `metadata`가 `name`보다
  먼저 나올 수 있습니다. 이 경우 `grep -A6 evt`는 위 예시와 다른 줄을 보여 줄 수 있으니, 결과가
  이상하면 `jsonpath` 방식으로 확인합니다. 실제 출력 결과는 이 기록에 남아 있지 않습니다.
- (개인 의견) `0 21 14 10 *`에는 연도가 없어서 매년 10월 14일에 다시 동작합니다. 일회성
  이벤트라면 이벤트가 끝난 뒤 트리거를 제거하는 후속 작업이 필요합니다.

## 참고 자료

- [AWS CloudShell 기본 설치 소프트웨어](https://docs.aws.amazon.com/cloudshell/latest/userguide/vm-specs.html#pre-installed-software)
- [aws eks update-kubeconfig](https://docs.aws.amazon.com/cli/latest/reference/eks/update-kubeconfig.html)
- [KEDA Cron scaler](https://keda.sh/docs/latest/scalers/cron/)
- [Kubernetes Version Skew Policy - kubectl](https://kubernetes.io/releases/version-skew-policy/#kubectl)

확인일: 2026-10-14
