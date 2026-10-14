# [조사] 001. 어떤 APM을 써야 하는가

- 작성일: 2026-10-14
- 대상: StoryRoom FE(`my-speech-v2-fe`), BE(`my-speech-v2-be`), STT(`storyroom-stt`)
- 조건: 가능하면 무료

---

## 1. 결론

**Grafana Cloud 무료 플랜 + OpenTelemetry**를 추천한다. FE에는 Grafana의 프론트엔드 SDK인 **Faro**를 쓴다.

- 세 프로젝트를 한 계정의 무료 한도 안에서 시작할 수 있다.
- 운영할 서버가 추가되지 않는다. 앱이 Grafana Cloud로 직접 데이터를 보낸다.
- 계측은 OpenTelemetry 표준으로 하므로, 나중에 백엔드(데이터를 보내는 곳)를 바꿔도 계측 코드는 그대로 둔다.

---

## 2. 배경: 지금 관측성 현황

| 항목            | 현재 상태                                                                                                               | 근거                                                       |
| --------------- | ----------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| 로그            | CloudWatch Logs `/storyroom/{fe,be,stt}/{stg,prd}`, 보관 3일                                                            | `storyroom-stt/deploy/observability/terraform/main.tf`     |
| 알림            | BE 에러 급증 → 메트릭 필터 → SNS → Google Chat                                                                          | `storyroom-stt/deploy/observability/terraform/alerting.tf` |
| FE 에러         | FE 크래시를 BE가 받아 CloudWatch에 기록 (`[FE-CRASH]`)                                                                  | `main.tf` 주석                                             |
| 요청 추적       | BE `TraceMdcFilter`가 요청마다 UUID `requestId`를 만들어 로그에 남김. **BE 안에서만 유효**하고 FE·STT와는 이어지지 않음 | `my-speech-v2-be/.../framework/filter/TraceMdcFilter.java` |
| APM / 분산 추적 | 없음                                                                                                                    | —                                                          |
| 과거 결정       | OpenSearch(ELK)는 2026-09-21 비용 때문에 폐기. "다시 만들지 말 것"                                                      | `main.tf` 주석                                             |
| 비용 절감       | BE access 로그를 CloudWatch 전송에서 빼서 월 약 $70 절감                                                                | `storyroom-stt/deploy/observability/be-agent/README.md`    |

### APM이 있었다면 바로 보였을 사례 (2026-10-14)

"evt1014 cron으로 STT base 파드가 21:00~21:30에 떠 있었는가"를 확인하는 데 EKS audit 로그 JSON을 직접 뒤져야 했다.

- 컨테이너 로그로는 확인할 수 없었다. fluent-bit가 공유 GPU 노드의 taint를 허용하지 않아, 그 노드의 로그가 수집되지 않기 때문이다(`storyroom-stt/deploy/eks/terraform/logging.tf`).
- 결과적으로 파드는 21:00:12에 생성됐지만 **21:03:53에야 처리 가능 상태**가 됐다. 그 사이 약 3분 40초는 노드 준비 약 90초와 이미지 받기 약 2분이었다.
- 분산 추적이 있었다면 STT 요청 하나가 "큐 대기, 콜드스타트, 전사" 중 어디서 시간을 썼는지 한 화면에서 볼 수 있었다.

> 참고: fluent-bit 로그 누락은 APM과 별개 문제이므로 따로 고쳐야 한다(tolerations를 `{ operator = "Exists" }`로 변경).

---

## 3. 프로젝트 특성

| 구분 | 스택                                                       | 실행 환경                                 | APM 관점에서 중요한 점                                                                                |
| ---- | ---------------------------------------------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| FE   | React 19 + Vite SPA                                        | S3 + CloudFront                           | 브라우저 에러, Web Vitals, API 호출 추적이 필요. BE를 거쳐 크래시를 남기는 지금 방식을 대체할 수 있음 |
| BE   | Spring Boot 4.1 / Java 21, JPA + MyBatis, Redis, WebClient | EC2 blue/green (systemd `java -jar`)      | `-javaagent`만 추가하면 코드 수정 없이 계측 가능                                                      |
| STT  | Python FastAPI + Redis Streams 워커 (Whisper / Qwen3-ASR)  | EKS + Karpenter GPU 노드, KEDA로 파드 0↔N | 파드가 자주 생겼다 사라지고, HTTP가 아닌 **큐(Redis Stream)로 요청을 받음**                           |

요청 흐름:

```
FE ──HTTP──▶ BE ──XADD──▶ Redis Stream(stream:stt:requests) ──XREADGROUP──▶ STT 워커
                                                                              │
BE ◀──────────────────── Redis Stream(결과) ◀─────────────────────────────────┘
```

중간에 HTTP가 아닌 큐 구간이 있다. 그래서 추적 정보(trace context)를 **메시지 필드에 직접 실어 보내야** 한 요청이 끝까지 하나로 이어진다.

---

## 4. 선정 기준

1. **무료로 쓸 수 있는가**: 팀 여러 명이 함께 쓸 수 있어야 한다.
2. **FE·BE·STT(Java·Python·브라우저)를 모두 다루는가**
3. **운영 부담이 없는가**: 서버나 스토리지를 새로 띄우지 않아야 한다. OpenSearch 폐기 결정과 맞아야 한다.
4. **벤더에 묶이지 않는가**: 한도 초과나 보안 정책 때문에 바꿀 때 계측을 다시 하지 않아도 되어야 한다.
5. **큐(Redis Stream) 구간까지 하나의 추적으로 이을 수 있는가**

---

## 5. 후보 비교

| 후보                                                        | 무료 범위 (2026 기준)                                                                                                     | 장점                                                                         | 이 프로젝트에서 아쉬운 점                                                                                           | 평가                            |
| ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| **Grafana Cloud (Free)** + OpenTelemetry                    | 트레이스 50GB/월, 로그 50GB/월, 메트릭 1만 시리즈, 프론트엔드 세션 5만, **사용자 3명**, 보관 14일, 카드 불필요, 기한 없음 | FE·BE·STT를 한 곳에서 봄. OpenTelemetry 표준이라 종속 없음. 운영할 서버 없음 | 사용자 3명 제한. 트래픽이 커지면 한도를 넘을 수 있어 샘플링이 필요                                                  | **추천**                        |
| Sentry (Developer)                                          | **사용자 1명**, 에러 5천/월, span 500만/월, 리플레이 50/월                                                                | FE 에러 추적과 세션 리플레이가 가장 좋음                                     | 무료는 1명만 쓸 수 있어 팀 APM으로는 부족. 유료 Team은 $26/월부터                                                   | FE 에러 보조용으로만 고려       |
| New Relic (Free)                                            | 수집량 100GB/월, **전체 기능 사용자 1명**(기본 사용자는 무제한), 보관 8일                                                 | 수집량이 가장 넉넉함                                                         | 전체 기능을 쓸 수 있는 사용자가 1명뿐. 보관 8일                                                                     | 보류                            |
| AWS X-Ray / CloudWatch Application Signals + CloudWatch RUM | 일부만 무료                                                                                                               | 데이터가 AWS 계정 안에 남음. IAM으로 권한 관리                               | 대부분 사용량 과금이라, CloudWatch 비용을 줄여온 방향과 맞지 않음                                                   | **외부 SaaS가 안 될 때의 대안** |
| 직접 설치형 (SigNoz, Pinpoint, Scouter, Elastic APM)        | 라이선스 무료                                                                                                             | 데이터를 직접 관리                                                           | 서버·스토리지 비용과 운영 부담. OpenSearch 폐기와 같은 길. Pinpoint·Scouter는 Java 중심이라 Python·FE를 다루지 못함 | 제외                            |
| Datadog                                                     | APM은 무료 범위 없음                                                                                                      | 기능이 가장 풍부함                                                           | 비용                                                                                                                | 제외                            |

---

## 6. 추천 이유

### 6.1 무료 한도로 세 프로젝트를 한 계정에서 다룬다

트레이스, 로그, 메트릭, 프론트엔드 세션이 모두 무료 한도에 들어 있다. 사용자 3명까지 쓸 수 있어 FE·BE·STT 담당자가 하나씩 계정을 가질 수 있다.

### 6.2 FE → BE → Redis Stream → STT를 하나의 추적으로 본다

이 구조에서 APM의 가치가 가장 큰 부분이다.

- OpenTelemetry는 표준 추적 헤더(W3C `traceparent`)를 쓴다.
- BE가 Stream에 메시지를 넣을 때(XADD) 이 값을 필드로 같이 넣고, STT 워커가 꺼내서 이어 붙이면 된다.
- 그러면 "이 STT가 왜 늦었나"를 큐 대기, 콜드스타트, 전사 시간으로 나눠 볼 수 있다.

### 6.3 운영할 서버가 없다

- 앱이 Grafana Cloud로 직접 데이터를 보낸다(OTLP). 중간 수집기(Collector) 없이 시작할 수 있다.
- 수집기를 EKS에 DaemonSet으로 깔지 않으므로, fluent-bit처럼 taint 때문에 일부 노드에서 빠지는 문제도 생기지 않는다.

### 6.4 벤더에 묶이지 않는다

- 계측은 OpenTelemetry 표준으로 한다.
- 한도를 넘거나 사내 보안 정책 때문에 바꿔야 하면, **데이터를 보내는 주소만 바꾸면 된다**. 예: AWS X-Ray/Application Signals(ADOT), 직접 설치한 Grafana Tempo.

### 6.5 세 스택 모두 표준 방식으로 붙는다

| 대상 | 방법                                                                                                                | 자동으로 추적되는 것                                     |
| ---- | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| BE   | OpenTelemetry **Java agent** (`-javaagent`). 대안으로 Spring Boot 4 전용 `spring-boot-starter-opentelemetry`가 있음 | HTTP 요청, JDBC(JPA/MyBatis), Redis(Lettuce), WebClient  |
| STT  | `opentelemetry-instrumentation-*` (fastapi, redis, botocore) + 워커에서 직접 만드는 span                            | HTTP 요청, Redis·S3 호출. 워커의 작업 단위는 코드로 추가 |
| FE   | Grafana **Faro** Web SDK + 추적 확장                                                                                | JS 에러, Web Vitals, fetch/XHR 호출(BE 추적과 연결)      |

OpenTelemetry 측은 Java에서 agent를 기본 방식으로 권장하고, agent를 쓸 수 없을 때 starter를 쓰라고 안내한다.

---

## 7. 도입할 때 주의할 점

### 7.1 무료 한도 안에서 쓰기

- **샘플링은 필수다.** BE에서 걸러낸 access 로그만으로 월 약 $70였다. CloudWatch 수집 단가를 GB당 $0.5~0.76으로 잡으면 대략 월 90~140GB 규모다(요청·응답 바디 포함 기준의 추정치). 트레이스를 100% 보내면 50GB 한도를 넘을 수 있다.
  - 상위 요청의 샘플링 결정을 따르는 방식(parent-based)에 **10~20%**로 시작한다.
  - 사용량 대시보드를 보면서 조정한다.
- **헬스체크는 추적에서 뺀다.** 로그에서 뺀 것과 같은 이유다.
  - BE: `/management/health/*` (ALB 헬스체크)
  - STT: `/health` (liveness가 파드마다 20초씩 호출)

### 7.2 개인정보와 보안

- 요청·응답 바디, STT 전사 텍스트, SSO(SAML) 정보, S3 키 같은 값은 span 속성에 넣지 않는다.
- **사내 보안 검토가 필요할 수 있다.** 데이터가 외부 SaaS로 나가기 때문이다. 승인이 안 되면 같은 계측을 그대로 두고 AWS 쪽으로 보낸다.
- 가입할 때 리전을 고를 수 있다. 가까운 아시아 리전이 있는지 확인한다.

### 7.3 STT 파드 특성

- GPU 파드는 KEDA가 0으로 내리면 2~3초 만에 종료된다(2026-10-14 audit 로그 실측). 종료할 때 남은 span을 flush하는지 확인한다.
- 컨테이너는 이미 떴지만 모델 로딩 중이라 처리할 수 없는 구간이 있다. 이 구간은 `STT Worker started` 같은 이벤트를 span 이벤트나 메트릭으로 남겨 두면 보인다.

### 7.4 BE 로그와 추적 ID 연결

`TraceMdcFilter`의 `requestId` 대신(또는 함께) OpenTelemetry의 `trace_id`를 로그에 남긴다. 그러면 CloudWatch 로그에서 바로 해당 트레이스로 이동할 수 있다.

---

## 8. 적용 순서 (제안)

단계마다 바로 효과가 보이도록 BE부터 시작한다.

### 1단계: BE (코드 수정 없음)

systemd 유닛(`scripts/{env}/story-room-{blue,green}.service`)의 `ExecStart`에 agent를 추가하고 환경변수를 설정한다.

```ini
# 예시 — 값은 Grafana Cloud 콘솔의 OpenTelemetry 연결 정보로 채운다
Environment=OTEL_SERVICE_NAME=storyroom-be
Environment=OTEL_RESOURCE_ATTRIBUTES=deployment.environment=prd
Environment=OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
Environment=OTEL_EXPORTER_OTLP_ENDPOINT=https://<grafana-cloud-otlp-endpoint>/otlp
Environment="OTEL_EXPORTER_OTLP_HEADERS=Authorization=Basic <base64(instanceId:token)>"
Environment=OTEL_TRACES_SAMPLER=parentbased_traceidratio
Environment=OTEL_TRACES_SAMPLER_ARG=0.2
ExecStart=/usr/bin/java -javaagent:/home/ubuntu/apps/otel/opentelemetry-javaagent.jar ... -jar app-blue.jar
```

- 토큰은 유닛 파일에 직접 쓰지 말고 `EnvironmentFile`이나 Secrets Manager로 분리한다.
- PoC에서 확인할 것:
  - 사용하는 agent 버전이 Spring Boot 4.1 / Spring 7을 지원하는가
  - 헬스체크를 어떻게 제외할 것인가. Java agent에는 URL 제외 옵션이 기본으로 없어서, sampler 확장이나 starter의 `ObservationPredicate`를 검토한다.

### 2단계: STT (Redis Stream 구간 연결)

- FastAPI 자동 계측을 켜고, 헬스체크를 제외한다(`OTEL_PYTHON_FASTAPI_EXCLUDED_URLS=health`).
- **BE (XADD 하는 쪽)**: 현재 추적 정보를 메시지 필드에 넣는다.
  ```java
  GlobalOpenTelemetry.getPropagators().getTextMapPropagator()
      .inject(Context.current(), fields, Map::put);   // fields: XADD 할 Map
  ```
- **STT 워커 (`workers/stt_worker.py`의 `_handle`)**: 필드에서 꺼내 consumer span을 만든다.

  ```python
  from opentelemetry import trace
  from opentelemetry.propagate import extract
  from opentelemetry.trace import SpanKind

  ctx = extract(data)  # data: XREADGROUP 메시지 필드 (bytes 면 str 로 디코드)
  with tracer.start_as_current_span("stt.job", context=ctx, kind=SpanKind.CONSUMER):
      ...  # 다운로드 / 전사 / 업로드 를 하위 span 으로
  ```

- 결과를 돌려주는 Stream(STT → BE)에도 같은 방식을 적용한다.

### 3단계: FE

- `@grafana/faro-web-sdk`와 `@grafana/faro-web-tracing`을 추가한다.
- 추적 헤더를 붙일 대상 URL로 BE API 도메인만 지정한다.
- BE CORS 설정에서 `traceparent` 요청 헤더를 허용한다.
- 안정되면 BE를 거쳐 FE 크래시를 남기는 지금 경로(`[FE-CRASH]`)를 정리할지 결정한다.

### 4단계: 정리

- BE 로그에 `trace_id`를 남긴다(log4j2 패턴).
- 기존 Google Chat 알림과 Grafana 알림이 겹치는 부분을 정리한다.

---

## 9. 결정이 필요한 사항

| 항목        | 내용                                                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------------------------------------- |
| 보안 승인   | 외부 SaaS(Grafana Cloud)로 트레이스와 에러 데이터를 보내도 되는가. 안 되면 AWS(ADOT → X-Ray/Application Signals)로 보낸다 |
| 사용자 3명  | 무료 계정을 쓸 담당자. 예: FE·BE·STT 각 1명                                                                               |
| 샘플링 비율 | 시작값 10~20%. 사용량을 본 뒤 조정                                                                                        |
| 적용 범위   | stg에서 먼저 PoC를 할지, prd에 바로 넣을지                                                                                |

---

## 참고 자료

- [Grafana Cloud Free Tier 2026: Limits, Pricing & What Changed](https://agentdeals.dev/vendor/grafana-cloud)
- [Grafana Cloud Pricing In 2026 (CloudZero)](https://www.cloudzero.com/blog/grafana-cloud-pricing/)
- [Sentry Free Tier 2026: Limits, Pricing & What Changed](https://agentdeals.dev/vendor/sentry)
- [Is Sentry Free? Developer Plan Limits (Costbench)](https://costbench.com/software/developer-tools/sentry/free-plan/)
- [New Relic Free Plan 2026 (Costbench)](https://www.costbench.com/software/developer-tools/new-relic/free-plan/)
- [New Relic Pricing 2026 (SigNoz)](https://signoz.io/guides/new-relic-pricing/)
- [OpenTelemetry with Spring Boot (spring.io)](https://spring.io/blog/2025/11/18/opentelemetry-with-spring-boot/)
- [OpenTelemetry Spring Boot: Java Agent, Starter, and Manual Instrumentation (Uptrace)](https://uptrace.dev/blog/opentelemetry-spring-boot.html)

> 무료 한도 수치는 2026년 10월 기준 공개 자료다. 가입하기 전에 각 사 가격 페이지에서 다시 확인한다.
