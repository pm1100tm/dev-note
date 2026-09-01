# 🚀 MemoryDB For Redis

AWS MemoryDB 는 AWS에서 ElastiCache보다 한 단계 높은 “고가용성 + 내구성(durability)”
을 제공하는 서비스입니다.

이걸 정리하면 “Redis의 빠름 + Aurora의 안정성” 을 결합한 서비스라고 보면 됩니다.

## ⚡ Amazon MemoryDB for Redis

Amazon MemoryDB for Redis 는 Redis 호환형(Redis-compatible) 이면서 영속성(Durability)
과 고가용성(High Availability) 을 강화한 완전관리형 In-Memory Database 서비스입니다.

| 설명                             | 내용                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| **Redis 호환성**                 | Redis API, 데이터 구조, 클라이언트 그대로 사용 가능          |
| **초고속 성능**                  | 초당 1.6억(160M+) 요청 처리 가능                             |
| **데이터 영속성 (Durability)**   | In-memory + Multi-AZ 트랜잭션 로그 저장으로 데이터 손실 방지 |
| **고가용성 (High Availability)** | Multi-AZ 구조로 장애 시 자동 복구                            |
| **확장성 (Scalability)**         | 수십 GB → 수백 TB까지 자동 확장 가능                         |
| **완전관리형 서비스**            | AWS가 패치, 백업, 복제, 복구를 자동으로 수행                 |

## ⚙️ 구조 개념 (Architecture)

```shell
        ┌─────────────────────────────────────────┐
        │         Amazon MemoryDB for Redis        │
        │  (Fully Managed Durable In-Memory DB)    │
        └─────────────────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          │                           │
   [Primary Node]              [Replica Nodes]
          │                           │
     Write + Log                  Read-only Replicas
          │
  Multi-AZ Transaction Log  (Persistent across AZs)
```

- 데이터는 메모리(In-Memory) 에 저장되지만,
  동시에 다중 AZ 트랜잭션 로그(Transaction Log) 로 지속적으로 복제됨.
- 따라서 노드 장애나 AZ 장애가 발생해도 데이터 유실 없음.

## 🧩 ElastiCache for Redis 와의 차이점

| 비교 항목         | ElastiCache for Redis               | MemoryDB for Redis                |
| ----------------- | ----------------------------------- | --------------------------------- |
| **목적**          | 고속 캐시 (Cache)                   | 영속형 In-Memory Database         |
| **데이터 영속성** | 선택적 (RDB / AOF)                  | 기본 제공 (Multi-AZ Log)          |
| **장애 복구**     | Replica Failover (비영속일 수 있음) | 완전 영속 복구 (Durable Log 기반) |
| **데이터 저장소** | 메모리 중심 (임시 저장)             | 메모리 + 지속적 트랜잭션 로그     |
| **사용 패턴**     | 캐시 중심 (TTL, eviction 등)        | 실시간 DB 대체 가능               |
| **성능**          | 매우 빠름 (μs 단위)                 | 거의 동일 (μs 단위)               |
| **비용**          | 상대적으로 저렴                     | 더 비쌈 (데이터 영속 비용 포함)   |

> 💡 한 줄로 요약하면
>
> ElastiCache = 캐싱용,
>
> MemoryDB = “Redis처럼 빠르지만 DB처럼 안정적인 저장소”.

## 💾 데이터 내구성 (Durability Mechanism)

MemoryDB는 데이터가 메모리에만 있는 것이 아니라,
각 쓰기(Write) 연산이 Multi-AZ 트랜잭션 로그로 영구 저장됩니다.

```shell
1️⃣ Client writes → Primary Node
2️⃣ Primary writes to in-memory data
3️⃣ Same write appended to durable, replicated log (across 3 AZs)
```

✅ 따라서 노드 장애, AZ 장애가 발생해도, 데이터는 트랜잭션 로그를 통해 자동 복구 가능.

### 🚀 주요 장점 (Benefits)

| 장점                                   | 설명                                         |
| -------------------------------------- | -------------------------------------------- |
| ⚡ **Ultra-Fast Performance**          | 완전 In-memory 기반으로 지연(Latency) 최소화 |
| 💾 **Durability (Persistent Storage)** | Multi-AZ 로그 저장으로 데이터 유실 방지      |
| 🔁 **High Availability (HA)**          | 자동 장애 복구 및 Replica 지원               |
| 🧠 **Redis 호환성**                    | 기존 Redis 클라이언트 그대로 사용 가능       |
| ☁️ **Fully Managed**                   | AWS가 패치, 모니터링, 스냅샷, 백업 자동 관리 |
| ⚙️ **Scalable**                        | 10GB → 100TB 이상까지 자동 확장 가능         |

### 💡 주요 사용 사례 (Use Cases)

| 사용 사례                          | 설명                                           |
| ---------------------------------- | ---------------------------------------------- |
| 🧾 **세션 스토어 (Session Store)** | 사용자 세션 데이터의 빠른 접근 + 데이터 영속성 |
| 🕹️ **온라인 게임 상태 관리**       | 랭킹, 게임 진행 상태 등 실시간 데이터 저장     |
| 🎬 **미디어 스트리밍 메타데이터**  | 빠른 재생 목록 / 추천 데이터 관리              |
| 🛒 **전자상거래 장바구니**         | 인메모리 속도 + 장애 복구 안정성               |
| 📱 **모바일 실시간 피드**          | 캐시 이상의 일관된 실시간 데이터 접근          |
