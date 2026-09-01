# 🚀 RDS Amazon Aurora

## 👉 Aurora 개요

- AWS 가 자체 개발한 클라우드 최적화(Cloud-Optimized) 관계형 데이터베이스 서비스
- MySQL, PostgreSQL 과 완전 호환
- RDS 보다 훨씬 빠르고, 안정적, 자동 확장성(Auto Scaling) 제공

### 주요 특징

- 엔진 종류: AWS 독자 기술(Proprietary)
- 호환성: MySQL, PostgreSQL 완전 호환
- 성능: MySQL 대비 최대 5배, PostgreSQL 대기 최대 3배 빠름
- 스토리지 자동 확장: 10GB 단위로 자동 증가(최대 256TB)
- 복제 속도: 10ms 이하 초저지연(Replication Lag)
- Replica 수: 최대 15개
- Failover: 거의 즉시(수초 내 전환)
- 비용: RDS 보다 약 20% 비쌈, 하지만 더 효율적
- 관리형 HA 기본 탑제: Multi-AZ 및 자동 복구 기능 내장

## Aurora 구조

- 6개의 데이터 복제본을 3개의 AZ에 분산 저장(각 AZ마다 2개씩 복제본)

```shell
           ┌──────────────┐
           │   AZ-a       │
           │  2 Copies    │
           └──────────────┘
                 │
                 ▼
           ┌──────────────┐
           │   AZ-b       │
           │  2 Copies    │
           └──────────────┘
                 │
                 ▼
           ┌──────────────┐
           │   AZ-c       │
           │  2 Copies    │
           └──────────────┘
```

- 총 6개의 복제본 중
  - 4개 이상이 일치하면 쓰기 가능 (Write Quorum)
  - 3개 이상이 일치하면 읽기 가능 (Read Quorum)
- Self-Healing: 손상된 블록은 자동 복구 (Peer-to-Peer Repair)
- 데이터는 100개 이상의 볼륨(Volume)에 분산 저장되어 I/O 부하를 줄임

### 🔹 Aurora의 고가용성 (High Availability)

- Failover 속도
  - 자동 장애조치 시 30초 미만
- Failover 대상
  - 기존 Read Replica 중 하나가 자동으로 승격
- 내장형 HA (High Availability)
  - 별도의 Multi-AZ 설정 불필요
- 복제 영역
  - 3개의 AZ에 자동 분산
- Cross Region 복제 지원
  - ✅ 지원 (DR/글로벌 읽기용 Replica 가능)

### 🔹 Aurora Read Scaling (읽기 확장성)

- 하나의 Master (Writer) 인스턴스만 쓰기 수행
- 최대 15개의 Read Replica가 읽기 전용으로 요청 처리
- 복제 지연이 10ms 이하 (MySQL보다 훨씬 빠름)
- 읽기 부하(SELECT)를 Replica로 분산 가능

### ⚡ Aurora vs RDS 비교

| 항목          | Amazon RDS                  | Amazon Aurora                   |
| ------------- | --------------------------- | ------------------------------- |
| 엔진          | MySQL, PostgreSQL 등        | Aurora (AWS 독자 엔진)          |
| 복제 방식     | AZ 단위 복제 (2개 인스턴스) | 3 AZ × 6 Copy 구조              |
| 스토리지      | 인스턴스 단위 EBS           | 자동 확장 스토리지 (최대 256TB) |
| Read Replica  | 최대 5개                    | 최대 15개                       |
| 복제 지연     | 수백 ms                     | 10ms 이하                       |
| Failover      | 수십 초~수분                | 수초 내 전환                    |
| Multi-AZ 설정 | 수동 옵션                   | 기본 내장                       |
| 성능          | 기준 수준                   | 최대 5배 성능 향상              |
| 비용          | 상대적으로 저렴             | RDS보다 약 20% 비쌈             |

### 📘 Aurora의 장점 요약

- ✅ 초고가용성 (HA 기본 내장)
- ✅ 자동 스토리지 확장 (최대 256TB)
- ✅ 읽기 성능 확장 (최대 15 Replica)
- ✅ 초저지연 복제 (10ms 이하)
- ✅ 자동 복구(Self-healing)
- ✅ Cross-Region 복제 가능 (DR 구조)

💡 한 줄 요약

- Amazon Aurora = RDS의 고성능, 고가용성, 자동확장 버전
- RDS보다 비싸지만, 빠르고 안전하고 자동화된 “클라우드 네이티브 DB” 입니다.
