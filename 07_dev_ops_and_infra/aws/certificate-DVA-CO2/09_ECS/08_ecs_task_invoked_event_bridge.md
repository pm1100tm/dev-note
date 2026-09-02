# 🚀 ECS Tasks Invoked by EventBridge

- ECS는 항상 실행되는 서비스(Service)만 운영할 수 있는 것이 아니다.
- 필요한 순간에만 **Task를 실행**하는 것도 가능하다.
- 이때 가장 많이 사용하는 서비스가 **Amazon EventBridge**이다.

## 전체 구조

```text
Event 발생
        │
        ▼
Amazon EventBridge
        │
        ▼
Run ECS Task
        │
        ▼
Task 실행
        │
        ▼
작업 완료
        │
        ▼
Task 종료
```

즉,

> **EventBridge가 ECS Task를 실행시키는 트리거(Trigger) 역할을 한다.**

---

## ECS Service와 ECS Task 차이

### ECS Service

항상 실행되어야 하는 애플리케이션. 예를 들어,

- Spring Boot API
- Node.js 서버
- 웹 서비스

### ECS Task

필요할 때만 실행되는 작업. 예를 들어,

- 배치 작업
- 데이터 변환
- 이미지 리사이징
- 메일 발송
- 보고서 생성

```
실행 → 완료 → 종료
```

---

## EventBridge가 하는 일

EventBridge는

> 특정 이벤트가 발생하면 ECS Task를 실행한다.

예를 들어

```
S3 업로드
→ EventBridge
→ Image Resize Task 실행
→ 완료 후 종료
```

### 예시 1

- 사용자가 image.jpg 를 S3에 업로드한다.
- S3 -> Object Created -> EventBridge -> Run ECS Task -> Python
  -> Thumbnail 생성 -> 종료

### 예시 2

- 사용자가 CSV 업로드 -> EventBridge -> DB 적재 -> 종료

---

## EventBridge가 받을 수 있는 이벤트

대표적으로

- S3 Object Created
- CodeCommit Push
- ECR Image Push
- CloudTrail Event
- Lambda 실행 결과
- DynamoDB Event
- 직접 만든 Custom Event

등이 있다.

---

## ECS Tasks Invoked by EventBridge Schedule

이번에는 이벤트가 아니라 시간을 기준으로 실행한다.

---

## 전체 구조

```text
매일 새벽 2시
        │
        ▼
EventBridge Schedule
        │
        ▼
Run ECS Task
        │
        ▼
Batch 실행
        │
        ▼
종료
```

즉,

> **정해진 시간마다 ECS Task를 실행한다.**

---

## 대표적인 사용 사례

### 1. 매일 새벽 배치

```
02:00
↓
매출 집계
↓
종료
```

### 2. 로그 압축

```
매일
↓
Log Compress
↓
S3 저장
↓
종료
```

---

### 3. 이메일 발송

```
매일 오전 9시
↓
메일 발송
↓
종료
```

---

### 4. DB 백업

```
매일 새벽
↓
Backup Task
↓
종료
```

---

## EventBridge Rule과 Schedule 차이

### Event 기반

```
무언가 발생
↓
Task 실행
```

예를 들어

```
S3 업로드
↓
Task 실행
```

---

## Schedule 기반

```
시간
↓
Task 실행
```

예를 들어

```
매일 01:00
↓
Task 실행
```

---

## EventBridge Schedule 표현식

### Rate

```
rate(5 minutes)
```

5분마다 실행

```
rate(1 hour)
```

1시간마다 실행

---

### Cron

```
cron(0 2 * * ? *)
```

매일 새벽 2시

---

```
cron(0 9 ? * MON-FRI *)
```

평일 오전 9시

---

## Fargate와 함께 사용하는 경우

실무에서는

Fargate와 EventBridge를 많이 조합한다.

```
EventBridge
↓
Fargate Task 생성
↓
Python 실행
↓
완료
↓
자동 종료
```

사용한 시간만 과금되므로 배치 작업에 매우 적합하다.

---

## EC2 Launch Type인 경우

```
EventBridge
↓
ECS Task
↓
EC2 위에서 실행
↓
종료
```

EC2는 계속 켜져 있어야 한다.

---

## Fargate인 경우

```
EventBridge
↓
AWS가 서버 준비
↓
Task 실행
↓
종료
↓
서버 제거
```

서버를 관리할 필요가 없다.

---

## 실무에서 가장 많이 사용하는 조합

| 이벤트               | 실행되는 Task   |
| -------------------- | --------------- |
| S3 업로드            | 이미지 리사이징 |
| S3 업로드            | AI 추론         |
| ECR Push             | 테스트 실행     |
| CloudTrail Event     | 보안 검사       |
| EventBridge Schedule | 배치 작업       |
| EventBridge Schedule | 보고서 생성     |
| EventBridge Schedule | DB 백업         |

---

## 시험 핵심

## EventBridge

✔ 이벤트 발생 시 ECS Task 실행

---

## EventBridge Schedule

✔ 시간 기반으로 ECS Task 실행

---

## ECS Service

✔ 항상 실행

---

## ECS Task

✔ 필요할 때 실행 후 종료

---

# 한 줄 암기

> **EventBridge = 이벤트가 발생하면 ECS Task 실행**

> **EventBridge Schedule = 정해진 시간마다 ECS Task 실행**

> **Fargate + EventBridge = 서버 관리 없이 실행되는 Serverless Batch 아키텍처**
