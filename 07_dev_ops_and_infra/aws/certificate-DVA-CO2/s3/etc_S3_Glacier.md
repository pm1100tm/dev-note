# 🧊 S3 Glacier

## 1️⃣ 한 문장 정의

- Amazon S3 Glacier는 “거의 안 꺼내보는 데이터를 매우 싸게 저장하는 장기 보관용 스토리지”다.

## 2️⃣ 왜 Glacier가 필요할까?

일반 S3의 문제

- 자주 안 쓰는 데이터도 계속 비싼 저장 비용
- 법적 보관, 로그, 백업 데이터는:
  - 읽지 않는다
  - 지우면 안 된다
  - 👉 접근은 느려도 되니, 최대한 싸게 저장
  - 👉 그 역할이 Glacier

## 3️⃣ Glacier는 “별도 서비스”가 아니다 (중요 ❗)

- Glacier는 S3 Storage Class 중 하나

```shell
S3 Standard
S3 Standard-IA
S3 One Zone-IA
S3 Glacier Instant Retrieval
S3 Glacier Flexible Retrieval
S3 Glacier Deep Archive
```

- 👉 전부 S3 Object
- 👉 Bucket도 동일

## 4️⃣ Glacier 계열 Storage Class 전체 정리 (시험 핵심)

### 🔹 1. S3 Glacier Instant Retrieval

| 항목      | 내용                        |
| --------- | --------------------------- |
| 접근 속도 | **밀리초 (즉시)**           |
| 비용      | 저렴                        |
| 용도      | 드물게 접근하지만 즉시 필요 |
| 예시      | 의료 이미지, 감사 로그      |

### 🔹 2. S3 Glacier Flexible Retrieval (구 Glacier)

| 항목           | 내용                        |
| -------------- | --------------------------- |
| 접근 속도      | **분~시간**                 |
| Retrieval 옵션 | Expedited / Standard / Bulk |
| 비용           | 더 저렴                     |
| 용도           | 백업, DR 데이터             |

### 🔹 3. S3 Glacier Deep Archive (제일 쌈)

| 항목      | 내용                 |
| --------- | -------------------- |
| 접근 속도 | **12~48시간**        |
| 비용      | **가장 저렴**        |
| 용도      | 법적 보관, 장기 기록 |
| 접근 빈도 | 거의 없음            |

## 5️⃣ 접근 속도 비교 (한눈에)

| Storage Class        | 접근 시간       |
| -------------------- | --------------- |
| S3 Standard          | 즉시            |
| Standard-IA          | 즉시            |
| Glacier Instant      | 즉시            |
| Glacier Flexible     | 분 ~ 시간       |
| Glacier Deep Archive | **반나절~이틀** |

## 6️⃣ Glacier의 핵심 제약 (⭐️⭐️⭐️)

| ❗ Glacier 객체는 바로 읽을 수 없다 (일부 제외)

<br>

Flexible / Deep Archive:

- Restore 작업 필요
- Restore 완료 후에만 접근 가능

Restore 요청

- → 기다림
- → 임시 복구
- → 읽기 가능

## 7️⃣ Glacier는 어떻게 쓰는가? (실무)

- 직접 업로드 ❌

```shell
S3 Standard
 → (30일)
 → S3 Glacier Flexible
 → (365일)
 → S3 Glacier Deep Archive
```

👉 Lifecycle Rule로 자동 이동
