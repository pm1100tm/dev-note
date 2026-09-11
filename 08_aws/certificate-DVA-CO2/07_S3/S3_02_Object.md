# 🚀 S3 Object

## 1️⃣ S3 Object란?

- S3에 저장되는 실제 데이터 단위(파일)
- S3는 파일 시스템이 아님
- S3는 Object Storage ⭕
- 디렉터리, inode, mount 같은 개념 ❌

## 2️⃣ Object의 핵심 식별자: Key

- Bucket 내에서 Object를 유일하게 식별하는 전체 경로 문자열

```shell
s3://my-bucket/my_file.txt
s3://my-bucket/my_folder1/another_folder/my_file.txt

👉 여기서:
Key = my_folder1/another_folder/my_file.txt
Key = my-bucket/my_folder1/another_folder/my_file.txt
```

## 3️⃣ Key = Prefix + Object Name

```shell
my_folder1/another_folder/   ← Prefix
my_file.txt                  ← Object Name
```

- Prefix는 문자열일 뿐
- Object Name도 문자열일 뿐

👉 / 는 구분자가 아니라 문자

## 4️⃣ ❗ S3에는 “디렉터리”가 없다

가장 중요한 문장

| **_S3에는 디렉터리가 존재하지 않는다_**

콘솔 UI가 헷갈리게 만드는 이유

- / 를 기준으로 UI에서 폴더처럼 보여줄 뿐
- 실제로는:
  - Key: my_folder1/another_folder/my_file.txt

## 5️⃣ Object Value (Body)

Object Value란?

- 실제 파일 내용
- Binary / Text / Image / Video 전부 가능

## 6️⃣ Object Size 제한 (시험 단골)

- 최소 0 bytes
- 최대 5 TB (5000 GB)
- 단일 PUT 업로드 5 GB
- 5GB 초과 Multipart Upload 필수

### Multipart Upload란?

- 파일을 여러 파트로 쪼개 업로드
- 병렬 업로드 가능
- 실패 시 재시도 효율적

👉 대용량 파일 업로드 = Multipart

## 7️⃣ Object Metadata

Metadata란?

- Object에 대한 설명 정보
- 텍스트 기반
- Object 업로드 시 지정
- 업로드 후 수정 ❌ (재업로드 필요)

| 종류            | 설명                  |
| --------------- | --------------------- |
| System Metadata | Content-Type, Size    |
| User Metadata   | 사용자 정의 Key/Value |

## 8️⃣ Object Tags (중요)

- Object에 붙이는 Unicode Key/Value

| 제한 | 값                      |
| ---- | ----------------------- |
| 개수 | **최대 10개**           |
| 용도 | 보안 / 비용 / Lifecycle |

Tag 활용 예

| 목적      | 활용                 |
| --------- | -------------------- |
| Lifecycle | 30일 후 Glacier      |
| 보안      | 특정 Tag만 접근 허용 |
| 비용      | 팀별 비용 추적       |

## 9️⃣ Version ID (Versioning 활성 시)

Versioning이 켜져 있으면

- Object 수정 시 새 버전 생성
- 삭제 시 Delete Marker 생성
- 각 Object는 Version ID 가짐

## 🔟 시험에 나오는 함정 포인트

- “S3 supports directories”

  - ❌ 틀림

- “Object is uniquely identified by bucket + key”

  - ⭕ 맞음

- “Upload object larger than 5GB”

  - 👉 Multipart Upload

- “Lifecycle rules based on folder”
  - 👉 ❌ 폴더 아님
  - 👉 ⭕ Prefix 기반
