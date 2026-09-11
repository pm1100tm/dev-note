# 🚀 GCE 에 SSD Disk 를 생성하여 마운트하기

PoC(Proof of Concept) 프로젝트를 진행하였습니다. 간단한 앱이었기 때문에, 환경 구성을 복잡하지
않도록, 그리고 인프라 비용이 적게 발생하도록 하고 싶었습니다.

아무래도 비용을 가장 많이 먹는 것은, RDS 같은 데이터베이스 시스템이니, 이걸 한 번에 EC2 같은 서버에
띄워보도록 하였습니다.

그래서, GCE 로컬에서, Qdrant 와 PostgreSQL, 그리고 Fast API 앱 까지, 컨테이너 3개를 전부
띄우고, 로컬 맥에서 DB에 접속하여 작업을 하는데 성능이 매우 좋지 않았습니다.

## 왜 성능이 안좋은가?

일단 원인은 EBS와 같은 Disk 성능에 있지 않을까 하였습니다. 밸런스드 Persistence Disk 자체의
용량도 크지 않았고, IOPS도 뛰어나진 않았습니다.

DB에 데이터를 대량으로 넣고, 쿼리를 함에 있어 IOPS는 매우 중요하니, Disk 를 새롭게 만들어서,
GCE에 마운트 시켜보고자 하였습니다.

---

## 👏 새로운 SSD 디스크를 추가해서 데이터 전용으로 사용

### 1. 새로운 SSD 디스크를 추가해서 데이터 전용으로 사용

Postgres 데이터만 옮기고 싶다면:

- Disks → Create disk 들어가서:
  - Disk type: Balanced SSD (pd-balanced) to SSD persistent disk (pd-ssd)
  - Size: 원하는 GB (여기서는 100GB로 설정함)
- VM → Edit → Additional disks → Attach 새 디스크
- /mnt/disks/ssd-data 같은 마운트 포인트 잡고 Postgres 데이터 디렉토리
  (/var/lib/postgresql/data) 옮기기.

➡️ 이렇게 하면 OS 부트디스크는 그대로 두고, Postgres I/O 만 SSD 를 쓰게 됩니다.

### 2. 부트 디스크 자체를 SSD 로 교체

- 현재 부트 디스크로 스냅샷 생성
- 새 디스크 생성하면서 Disk type 을 pd-ssd 로 선택 후, 방금 만든 스냅샷을 소스로 지정
- VM 중지 → 새 SSD 디스크를 부트 디스크로 교체 → VM 시작

➡️ 이러면 전체 OS + 데이터 모두 SSD 기반으로 이동.

### 3. IOPS/Throughput 커스터마이즈

- Hyperdisk (Extreme SSD) 같은 신규 타입에서는 IOPS/Throughput 수치를 직접 설정할 수 있어요.
- 다만 표준 pd-ssd, pd-balanced 에서는 디스크 크기에 따라 자동 할당됩니다.
  - 예: pd-ssd → 30 IOPS/GB, 최대 100,000 IOPS
  - 200GB 이상 잡아야 IOPS 가 확 늘어납니다.

---

## 👉 1. 새로운 SSD 디스크를 추가해서 데이터 전용으로 사용

```shell
# ---------------------------------------------------------------------------
# 1. docker volume mount 경로 확인
docker inspect postgres | grep -A 5 Mounts

## 출력 예시
"Mounts": [
  {
    "Type": "volume",
    "Name": "postgres_data",
    "Source": "/var/lib/docker/volumes/postgres_data/_data",
    "Destination": "/var/lib/postgresql/data",
    "Driver": "local"
  }
]


# ---------------------------------------------------------------------------
# 2. 해당 경로가 어느 디스크에 있는지 확인
df -h /var/lib/docker/volumes

## 출력 예시
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G   3G   25G  12% /
# 👉 이렇게 나오면 /dev/sda1 (부트 디스크)에 올라가 있다는 뜻입니다.
# 📌 결론
# docker volume 의 Source 가 /var/lib/docker/volumes/... 라면 → 지금 Postgres 데이터는 부트 디스크 위에 있습니다.
# 따라서 Balanced persistent disk 성능 한계 때문에 느리게 느껴질 수 있습니다.

# 📌 개선 방향
# 데이터 전용 SSD 디스크(pd-ssd 등)를 추가해서 /var/lib/docker/volumes 또는
# /var/lib/postgresql/data 를 거기로 옮기는 게 가장 효과적입니다.
# Postgres 데이터를 마이그레이션하는 절차는:
# - GCE → Create disk (pd-ssd, 100GB 이상 추천)
# - VM에 attach → /mnt/disks/ssd-data 로 포맷/마운트
# - docker-compose.yml 의 volumes 경로를 /mnt/disks/ssd-data/postgres 로 바꿈
# - 컨테이너 재배포

# 📌 각 옵션 요약
# Standard persistent disk (pd-standard)
# - HDD 기반, 가장 저렴하지만 랜덤 I/O 성능이 낮음 → DB용 ❌

# Balanced persistent disk (pd-balanced)
# - 기본 SSD, 가격 대비 무난. 지금 부트 디스크로 쓰고 있는 타입.
# - IOPS/Throughput 은 pd-ssd 보다는 낮음.

# SSD persistent disk (pd-ssd)
# - 고성능 SSD, IOPS/Throughput 이 크기당 더 높음.
# - Postgres, Qdrant 같은 DB/스토리지 서비스용 추천 ✅

# Extreme persistent disk
# - 초고성능 SSD, IOPS/Throughput 을 원하는 만큼 직접 설정 가능.
# - 엔터프라이즈 환경, 초고부하 DB용. 비용 ↑↑

# Hyperdisk (Balanced / Extreme / Throughput / ML)
# - 차세대 스토리지. 세분화된 성능/비용 조정 가능.
# - 비용은 상황에 따라 오히려 저렴할 수도.
# - 신규 서비스라 안정성/호환성 면에서 보수적으로는 아직 pd-ssd 많이 씀.

# 📌 PostgreSQL 컨테이너 전용 디스크라면?
# ➡️ SSD persistent disk (pd-ssd) 선택하는 게 가장 적절합니다.
# - 가격도 적당하고, 확실히 pd-balanced 대비 랜덤 읽기/쓰기 성능이 높음.
# - 보통 DB 서버는 IOPS 가 중요하니까 pd-ssd 로 가는 게 표준적인 선택입니다.

# ---------------------------------------------------------------------------
# ① VM에 디스크 attach
# 1. GCP Console → Compute Engine → VM instances
# 2. 해당 VM (shop-dev) 클릭
# 3. 상단 메뉴에서 Edit 클릭
# 4. Additional disks → Add new disk
#    - 방금 만든 SSD 디스크 선택
#    - Mode: Read/Write
#    - Device name 확인 (/dev/sdb 같은 이름으로 붙음)
# 5. 저장 후 VM 재시작(보통 필요 없음, 바로 붙습니다).

# ② VM 내부에서 디스크 확인
# SSH 접속 후:
lsblk

# 예시 출력:
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda       8:0    0   30G  0 disk
└─sda1    8:1    0   30G  0 part /
sdb       8:16   0  100G  0 disk   # ← 새로 붙인 SSD

# ③ 디스크 포맷 (ext4 추천)
sudo mkfs.ext4 -m 0 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb

# ④ 마운트할 디렉토리 생성
sudo mkdir -p /mnt/disks/ssd-data

# ⑤ 디스크 마운트
sudo mount -o discard,defaults /dev/sdb /mnt/disks/ssd-data

# ⑥ 재부팅 시 자동 마운트 등록
# /etc/fstab 수정:
sudo blkid /dev/sdb

# 출력 예:
/dev/sdb: UUID="abcd-1234-5678-efgh" TYPE="ext4"

# /etc/fstab 에 등록:
vim /etc/fstab

# 아래와 깉이 등록
UUID=abcd-1234-5678-efgh   /mnt/disks/ssd-data   ext4    discard,defaults,nofail   0   2

# 옵션 설명:
# - discard → SSD TRIM 최적화
# - defaults → 기본 마운트 옵션
# - nofail → 디스크 없어도 부팅 실패하지 않음
# -	0 2 → fsck 순서 (루트는 1, 나머지는 2)

# 저장 후 재적용
sudo mount -a

# 정상 마운트 확인
df -h | grep ssd-data

# 재부팅
sudo reboot

# 재부팅 후 다시 SSH 접속해서 확인:
df -h | grep ssd-data
# 👉 /mnt/disks/ssd-data 가 잘 붙어 있으면 성공 🎉


# ✅ 이후 단계
# - /mnt/disks/ssd-data/postgres 디렉토리 생성
# - docker-compose.yml 수정해서 Postgres volume 경로를 저기로 지정
# - 컨테이너 재배포
```

📌 컨테이너 재배포시 volume 지정

```shell
services:
  db:
    image: postgres:16.9
    container_name: postgres
    ports:
      - "6543:5432"
    environment:
      - POSTGRES_USER=
      - POSTGRES_PASSWORD=
      - POSTGRES_DB=postgres
    volumes:
      - /mnt/disks/ssd-data/postgres:/var/lib/postgresql/data
    restart: always
```

📌 기존 데이터 옮기기 (선택 사항)

```shell
docker stop postgres
sudo rsync -av /var/lib/docker/volumes/shop-dev_postgres_data/_data/ /mnt/disks/ssd-data/postgres/
```

📌 컨테이너 재배포

```shell
docker compose -f docker-dev/compose-dev.yml down
docker compose -f docker-dev/compose-dev.yml up --build -d
```

👉 여기서 중요한 건, 기존 데이터를 살릴지/완전히 새로 시작할지 입니다.
데이터를 새로 시작해도 된다면 바로 재배포하면 되고, 기존 데이터를 꼭 살려야 한다면 제가 rsync
절차를 조금 더 디테일하게 써드려야 해요.

### rsync 설치

```shell
sudo apt-get update
sudo apt-get install -y rsync

docker stop postgres
sudo rsync -av /var/lib/docker/volumes/shop-dev_postgres_data/_data/ /mnt/disks/ssd-data/postgres/
# 옵션 설명:
# -a : 권한, 소유자, 타임스탬프 등 모두 유지
# -v : 진행 상황 출력
```

---

## 그 외

### PostgreSQL 설정 튜닝

#### postgresql.conf 에서 주요 파라미터를 조정

Postgres 는 메모리를 많이 줘도 기본값은 작습니다.

```shell
# 메모리 관련
shared_buffers = 25% of RAM      # 예: 8GB RAM이면 2GB 정도
work_mem = 64MB                  # 쿼리별 sort/hash 연산 메모리
maintenance_work_mem = 512MB     # VACUUM, CREATE INDEX 등에 사용

# 캐시 관련
effective_cache_size = 75% of RAM  # OS 파일시스템 캐시 활용 예상치

# WAL / Write 관련
wal_buffers = 16MB
synchronous_commit = off           # (안정성보다 성능 우선이면)
checkpoint_completion_target = 0.9
```

👉 GCE 컨테이너에서는 /var/lib/postgresql/data/postgresql.conf 수정 후 컨테이너 재시작.

#### 컨테이너 자원 고정

- docker-compose.yml 에서 Postgres 컨테이너에 CPU/RAM limit/예약을 걸어주는 게 좋습니다.

```shell
  db:
    image: postgres:16
    deploy:
      resources:
        limits:
          cpus: '4'
          memory: 6G
        reservations:
          cpus: '2'
          memory: 4G
```

#### 모니터링

- pg_stat_activity, pg_stat_statements 로 가장 무거운 쿼리 확인.
- iotop, htop, docker stats 로 CPU/RAM/I/O 병목 지점 확인.
