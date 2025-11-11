# 🚀 RDS 읽기 전용 복제본과 다중 AZ & Single-AZ → Multi-AZ 전환

---

## 🔥 읽기 전용 복제본(Read Replica)

- 읽기 전용 부하(SELECT 쿼리)를 분산시키기 위해 사용하는 복제 데이터베이스 인스턴스
- 운영 중인 메인 DB 데이터를 비동기적으로(ASYNC) 복제하여, 조회 요청을 별도의 복제본으로 분산

### Read Replica 주요 특징

- 최대 15개의 Read Replica 생성 가능
- 동일 AZ, 다른 AZ, 또는 다른 리전에 생성 가능
- 비동기 복제(Asynchronous Replication) 방식
  - 데이터는 실시간이 아닌 일정 지연(Replication Lag) 후 복제됨
- Read Replica는 읽기 전용(SELECT) 으로만 사용
  - INSERT, UPDATE, DELETE는 지원하지 않음
  - 필요 시 독립적인 DB 인스턴스로 승격(Promotion) 가능
  - Read Replica를 활용하려면 👉 애플리케이션의 DB 연결 문자열(connection string) 을
    수정해야 함

### 사용 시나리오

- 운영 DB 부하 분산
  - 트래픽이 많은 서비스의 경우, 조회 요청(SELECT)을 Read Replica로 분산
  - 메인 DB의 부하 감소 및 응답속도 향상
- 리포팅 및 분석용 쿼리 분리
  - 실시간 트래픽을 받는 운영 DB에서 무거운 JOIN, GROUP BY 등 분석 쿼리를 분리
  - 예: BI, 데이터 시각화, 리포트 대시보드 등
- 리전 간 데이터 복제
  - 다른 리전에 Read Replica를 생성하여, 글로벌 사용자에게 지연시간(Latency) 을 줄임
  - 또는 DR(Disaster Recovery) 용도로 활용 가능

### ⚠️ 주의사항

- Replication은 비동기이므로, 최신 데이터가 약간 지연될 수 있음
  - → “Eventually Consistent”
- Failover 시 자동 전환되지 않음 (Multi-AZ와 다름)
- Read Replica를 승격(Promotion) 하면 해당 인스턴스는 독립적인 DB로 전환되며 복제 관계가 끊어짐

### 💰 RDS Read Replica – Network Cost (네트워크 요금)

Amazon RDS에서 Read Replica를 생성하면, Primary DB의 데이터를 복제하기 위해 데이터가
네트워크를 통해 전송됩니다. 이때 전송 경로가 어느 영역(AZ, Region) 을 통과하느냐에 따라 요금이
달라집니다.

#### ✅ 같은 리전(Same Region) 내 동일 AZ 또는 다른 AZ

- 예: ap-northeast-2a → ap-northeast-2b
- 무료

#### ⚠️ Cross Region (리전 간) 복제

- 예: ap-northeast-2 (서울) → us-west-2 (오레곤)
- 유료 (Data Transfer Cost 발생)

AWS의 네트워크 요금 정책은 다음과 같습니다:

- 같은 리전 내 트래픽은 내부 네트워크(Intra-region traffic) 으로 간주되어 무료
- 다른 리전 간 트래픽은 인터넷 백본을 거치는 외부 전송(Inter-region transfer) 으로 간주되어 과금 대상

### Read Replica 유지에 대한 비용

RDS Read Replica는 단순한 “복제 파일”이 아니라 실제로 돌아가는 독립적인 RDS 인스턴스(즉, 서버)
입니다.

- AWS는 Primary와 동일한 리소스를 할당해야 하므로
- CPU / 메모리 / EBS 스토리지 자원이 그대로 필요함

→ 따라서 인스턴스 시간당 요금 + 스토리지 요금이 그대로 청구됩니다.

| 항목          | 원본 DB         | Read Replica    |
| ------------- | --------------- | --------------- |
| 엔진          | PostgreSQL      | PostgreSQL      |
| 인스턴스 타입 | db.t3.medium    | db.t3.medium    |
| 스토리지      | 100GB           | 100GB           |
| 위치          | ap-northeast-2a | ap-northeast-2a |

👉 이 경우

- 매 시간당 인스턴스 요금이 2개(DB + Replica)에 대해 각각 부과됩니다.
- 스토리지 100GB × 2개 = 200GB 분량의 EBS 요금 발생.
- 단, 데이터 복제 트래픽 요금은 무료.

#### 💰 RDS Read Replica 비용 구조

- ✅ DB 인스턴스 비용
  - Read Replica도 독립된 RDS 인스턴스이므로, EC2와 동일하게 인스턴스 시간당 요금이 부과됨
- ✅ 스토리지 비용 (EBS)
  - 원본 DB와 동일한 스토리지 크기만큼 요금 부과
- ✅ 백업 스토리지 (Backup Storage)
  - 자동 백업이 활성화되어 있으면 백업 스냅샷 비용 발생
- ✅ IO 요청 (I/O Request)
  - 데이터 복제 및 쿼리 실행 중 발생하는 읽기/쓰기 I/O 비용
- ✅ 데이터 전송 비용 (Network Transfer)
  - 같은 리전/같은 AZ 내에서의 복제는 무료 (Intra-Region Free)

---

## 🔥 다중 AZ(Multi AZ)

Multi-AZ 배포(Multi-Availability Zone) 는 데이터베이스의 가용성(Availability) 과
복원력(Resiliency) 을 높이기 위한 Disaster Recovery(재해 복구) 구조입니다.

즉, 한 AZ에 장애가 발생하더라도 다른 AZ에 있는 대기용(Standby) DB 인스턴스로 자동
장애조치(Failover) 가 이루어집니다.

### ⚙️ 주요 특징

- 복제 방식
  - ✅ 동기(Synchronous) 복제
- 엔드포인트
  - ✅ 하나의 공통 DNS 이름 사용 (Failover 시 자동으로 전환)
- Failover 동작
  - ✅ AZ, 네트워크, 인스턴스, 스토리지 장애 시 자동 전환
- 관리자 개입
  - ❌ 불필요 (자동 처리됨)
- 목적
  - 🔹 고가용성(HA), 장애 복구(DR)
- 확장성
  - ❌ 읽기 부하 분산(Scaling) 용도는 아님
- 읽기 전용 Replica
  - ❌ Multi-AZ 자체는 Read Replica로 쿼리 불가능
- DR을 위한 Replica 구성
  - ✅ Read Replica를 Multi-AZ 로 설정 가능 (DR용으로 활용)

```shell
                ┌──────────────────────────────┐
                │          AWS RDS             │
                └──────────────────────────────┘
                           │
        ┌──────────────────┴──────────────────┐
        ▼                                     ▼
Primary DB (AZ-a)                    Standby DB (AZ-b)
     ↑                                        ↑
     │                                        │
     └─────── SYNC Replication ───────────────┘
```

- Primary(운영 DB)는 읽기/쓰기 모두 처리
- Standby는 동기 복제(Sync) 를 통해 실시간 데이터 미러링
- 장애 발생 시 자동으로 Standby가 Primary로 승격

### ⚠️ 주의사항

- Multi-AZ는 성능 확장이 아닌 안정성 보장용
  - → 읽기 전용 부하 분산을 원하면 Read Replica 사용
- Standby 인스턴스는 직접 접근 불가 (Read 전용으로 쿼리 불가능)
- 복제 비용 및 추가 인스턴스 요금 발생 (Primary + Standby 모두 과금)

---

## 🛠️ Single-AZ → Multi-AZ 전환

RDS에서는 DB를 중단하지 않고 Multi-AZ로 전환할 수 있습니다.

단계별 내부 동작

- 1. AWS가 현재 DB의 Snapshot 생성
- 2. 다른 AZ에 새로운 Standby 인스턴스 복원
- 3. Primary ↔ Standby 간 동기 복제(Sync Replication) 설정
- 4. 전환 완료 후, Multi-AZ 활성화 상태로 전환

✅ 즉, 클릭 한 번(Modify → Multi-AZ 활성화)으로 무중단 전환 가능

---

## Read Replica 와 Multi AZ 비교

| 항목      | Multi AZ                      | Read Replica                   |
| --------- | ----------------------------- | ------------------------------ |
| 목적      | 고가용성(HA) / 장애 복구(DR)  | 읽기 부하 분산(Scaling)        |
| 복제방식  | 동기                          | 비동기                         |
| 징애 시   | 자동 Failover (DNS 자동 변경) | 자동 전환 없음                 |
| 읽기 쿼리 | 불가능 (Standby 접근 불가)    | 가능 (Replica에서 SELECT 가능) |
| 사용 요금 | 2개 인스턴스 비용 발생        | Replica 1개당 별도 비용        |
| 복제 위치 | 동일 리전 내 다른 AZ          | 동일 리전 또는 다른 리전       |
