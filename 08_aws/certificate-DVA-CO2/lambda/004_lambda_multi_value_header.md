# 🚀 Lambda Multi Value Header

개념

- ALB 설정에서 Multi-Value Headers 활성화 가능
- 동일한 Header / Query String이 여러 값일 때 배열로 전달

## 🔄 단일 vs 멀티 값 비교

| 구분         | 단일 값                 | 멀티 값                           |
| ------------ | ----------------------- | --------------------------------- |
| Headers      | `headers`               | `multiValueHeaders`               |
| Query String | `queryStringParameters` | `multiValueQueryStringParameters` |

## 📌 Multi-Value Event 예시

```json
{
  "multiValueHeaders": {
    "x-forwarded-for": ["1.1.1.1", "2.2.2.2"]
  },
  "multiValueQueryStringParameters": {
    "tag": ["java", "aws", "lambda"]
  }
}
```

- Multi-value headers가 켜져 있으면 배열로 전달된다

## 🔐 ALB + Lambda – Permissions (매우 중요)

핵심 원칙

ALB는 Lambda를 호출하려면 반드시 권한이 필요

Lambda에 Resource-based Policy 를 추가해야 한다.
