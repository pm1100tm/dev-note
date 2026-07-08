# 008\_ssh\_options

📌 SSH의 모든 옵션을 보고 싶을 때

```shell
man ssh
```

```shell
ssh -h
ssh --help
```

## 자주 사용하는 옵션 정리

| 옵션 | 의미                                 | 예시                                                                   |
| -- | ---------------------------------- | -------------------------------------------------------------------- |
| -i | 개인 키 파일(Private Key) 지정            | `ssh -i ~/.ssh/keypair.pem ubuntu@1.11.11.111`                       |
| -p | SSH 포트 지정 (기본 22번)                 | `ssh -p 2222 ubuntu@1.11.11.111`                                     |
| -v | 디버그(Verbose) 모드, 연결 과정 상세 출력       | `ssh -v -i key.pem ubuntu@1.11.11.111`                               |
| -L | 로컬 포트 포워딩 (Local → Remote)         | `ssh -L 8080:localhost:80 ubuntu@1.11.11.111` → 내 PC 8080 → 원격 서버 80 |
| -N | 원격 명령 실행하지 않고 포트 포워딩만              | `ssh -N -L 8080:localhost:80 ubuntu@1.11.11.111`                     |
| -C | 압축 전송 (대역폭 줄이기)                    | `ssh -C -i key.pem ubuntu@1.11.11.111`                               |
| -o | 추가 옵션 지정 (StrictHostKeyChecking 등) | `ssh -o StrictHostKeyChecking=no -i key.pem ubuntu@1.11.11.111`      |
