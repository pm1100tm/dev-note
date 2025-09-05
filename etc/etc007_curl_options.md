# 🚀 CURL 옵션 정리

## 1. 기본 정보 및 헬프

```shell
curl -h    # 사용가능 옵션 간단한 설명
curl -V    # curl, libcurl 버전 정보 확인
curl --man # curl 메뉴얼 전체보기
```

## 2. 다운로드 / 업로드 관련

```shell
curl -o <file>, --output <file> # 지정한 파일명으로 저장
curl -O                         # 원격 파일명을 그대로 사용해 저장
curl -C, --continue-at <offset> # 중단된 다운로드 이어받기
curl -T, --upload-file <file>   # 파일 업로드(FTP 등에서 사용)
```

## 3. HTTP 요청 구성

```shell
curl -X <METHOD>                    # HTTP 메서드 지정 (GET, POST, PUT, DELETE 등)
curl -d <data>, --data              # POST 데이터 전송 (Content-Type: application/x-www-form-urlencoded)
curl --data-urlencode               # URL 인코딩된 POST 데이터 전송
curl -H <header>, --header <header> # 커스텀 HTTP 헤더 추가
```

## 4. 리다이렉션 및 헤더

```shell
curl -L, --location # 리다이렉션 자동 따라가기
curl -I, --head     # HEAD 요청 (응답 헤더만 확인)
curl -i, --include  # 응답 헤더를 출력 내용에 포함
curl -D <file>      # 응답 헤더를 파일로 저장
```

## 5. 인증 및 보안

```shell
curl -u user:pass, --user user:pass # HTTP Basic 인증
curl --digest                       # HTTP Digest 인증
curl -k, --insecure                 # SSL 인증서 확인 건너뛰기
curl --cacert <file>                # 특정 CA 인증서 파일 지정
curl --cert <cert[:password]>       # 클라이언트 인증서 지정
```

## 6. 출력 및 디버깅

```shell
curl -v, --verbose        # 요청/응답 디버깅 정보 출력
curl -#, --progress-bar   # 진행률 막대 표시
curl -w <format>          # 실행 결과 출력 포맷 지정
curl --trace <file>       # 전체 요청/응답 로그 파일 기록
curl --trace-ascii <file> # ASCII 로그 기록
curl --trace-time         # trace 파일에 시간 스탬프 추가
```

## 7. 네트워크 및 타임아웃

```shell
curl --connect-timeout <sec>    # 연결 시도 제한 시간
curl -m <sec>, --max-time <sec> # 전체 요청 제한 시간
curl --limit-rate <speed>       # 전송 속도 제한 (예: 100K, 1M)
curl --interface <name>         # 특정 네트워크 인터페이스 사용
curl -4, --ipv4                 # IPv4 강제 사용
curl -6, --ipv6                 # IPv6 강제 사용
```

## 8. 쿠키 관련

```shell
curl -b <data|file>, --cookie <data|file>  # 쿠키 데이터 전송 (직접 입력 또는 파일 사용)
curl -c <file>, --cookie-jar <file>        # 쿠키를 파일에 저장
```

## 9. 압축 및 HTTP 버전

```shell
curl --compressed            # 압축된 응답 자동 해제
curl --http1.0               # HTTP/1.0 사용
curl --http1.1               # HTTP/1.1 사용
curl --http2                 # HTTP/2 사용
curl --http2-prior-knowledge # HTTP/2 직접 사용 (Upgrade 없이)
```

## 10. 자주 사용하는 예제

```shell
# 기본 GET 요청
curl https://example.com

# 리다이렉션 따라가면서 응답 헤더까지 출력
curl -L -i https://example.com

# POST 요청 + 헤더 지정
curl -X POST -d "name=test" -H "Content-Type: application/x-www-form-urlencoded" https://api.example.com

# 파일 다운로드 (원격 파일명 그대로)
curl -O https://example.com/file.zip

# 쿠키 저장 & 재사용
curl -c cookies.txt -b cookies.txt https://example.com

# SSL 인증 무시 (개발용)
curl -k https://self-signed.example.com

# 연결 타임아웃 10초
curl --connect-timeout 10 https://example.com
```
