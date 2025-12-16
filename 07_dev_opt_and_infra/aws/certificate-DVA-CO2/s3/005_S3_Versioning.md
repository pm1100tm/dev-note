# 🚀 S3 Versioning

## 1️⃣ S3 Versioning이란?

같은 Key(Object 이름)에 대해 변경 이력을 모두 보관하는 기능

- Bucket 레벨에서 활성화
- Object 레벨로 버전 관리

| 같은 파일을 덮어써도 기존 파일은 사라지지 않는다

## 2️⃣ 어떻게 동작하는가?

```shell
Key: my_file.txt
Versions:
- v1
- v2
- v3
```

- 같은 Key
- 다른 Version ID
- 👉 덮어쓰기 = 수정 ❌
- 👉 새 버전 생성 ⭕

## 3️⃣ 왜 Best Practice 인가?

| 효과          | 설명                    |
| ------------- | ----------------------- |
| 실수 방지     | 잘못 삭제/덮어쓰기 복구 |
| 롤백          | 이전 버전 즉시 복원     |
| 감사          | 변경 이력 유지          |
| 랜섬웨어 대응 | 이전 정상 버전 복구     |

- 👉 중요 데이터가 있으면 Versioning은 기본

## 4️⃣ 삭제(Delete)의 실제 동작 (시험 단골 ⚠️)

❌ Versioning OFF

- Delete → 파일 완전 삭제

⭕ Versioning ON

- Delete → Delete Marker 생성
- 실제 Object는 안 지워짐
- 최신 버전 위에 Delete Marker만 올라감
- 👉 Delete Marker 제거하면 복구됨

## 5️⃣ “null” Version의 의미 (시험에 나옴)

- Versioning 비활성화 상태에서 업로드
- 이후 Versioning 활성화

## 6️⃣ Versioning 상태 변화 (중요)

| 상태                 | 설명         |
| -------------------- | ------------ |
| Unversioned          | 기본 상태    |
| Versioning Enabled   | 새 버전 생성 |
| Versioning Suspended | 새 버전 중단 |

❗ 핵심 포인트

- Suspended는 삭제가 아니다
- 기존 버전 그대로 유지
- 새 업로드는 다시 null 버전 생성

## 7️⃣ Versioning + Lifecycle (실무 핵심)

```shell
- 최신 버전: S3 Standard
- 이전 버전: Glacier
- 1년 후: Deep Archive
- 5년 후: 삭제
```

👉 비용 관리 필수

## 8️⃣ 비용 관련 주의사항 (현실 ⚠️)

- Versioning은 저장 비용 증가
- 삭제해도 비용 계속 발생
- 반드시 Lifecycle Rule과 함께 사용
