# Apache 서버에서 앱 버전 정보 가져오기

상황:

Window 에 D:\Mobile 폴더를 만들고, 그 안에 아래와 같이 구성하였다.

```shell
├── images
│   ├── btn_android.gif
│   └── btn_iphone.gif
├── index_operation.html
├── index.html
└── install
    ├── dev
    │   ├── build-info.txt
    │   ├── hihr-dev-1.0.5-6.aab
    │   ├── hihr-dev-1.0.5-6.apk
    │   ├── hihr-dev-1.0.5-6.ipa
    │   └── manifest.plist
    ├── prod
    │   ├── build-info.txt
    │   ├── hihr-prod-1.0.5-6.aab
    │   ├── hihr-prod-1.0.5-6.apk
    │   ├── hihr-prod-1.0.5-6.ipa
    │   └── manifest.plist
    └── qa
        ├── build-info.txt
        ├── hihr-qa-1.0.5-6.aab
        ├── hihr-qa-1.0.5-6.apk
        ├── hihr-qa-1.0.5-6.ipa
        └── manifest.plist
```

그 후, Apache24 의 httpd.conf 를 구성하여 index.html 를 띄울 수 있도록 구성하고,
ngrok 을 사용하여 임시 https 도메인을 만들었다.

이 때 AOS나 iOS에서, 앱 자체의 버전 정보과, Apache 서버의 버전 정보를 비교하여, 버전 정보가
다를 경우, 특히, 버전 정보가 올라간(업데이트) 경우, 업데이트를 유도하기 위한 프로세스를 정리한다.

## 1. window 에서 apache 구동

```shell
cd C:\Apache24\bin
httpd.exe
```

## 2. ngrok 실행

```shell
# Apache24 bin httpd.conf 에 설정한 listen 포트로 실행
ngrok http 8080
```

## 3. 버전 정보 가져오기

```shell
# 접속 확인
https://166d-210-222-118-66.ngrok-free.app

# 버전 정보 가져오기
https://166d-210-222-118-66.ngrok-free.app/install/dev/build-info.txt

curl -fsS https://166d-210-222-118-66.ngrok-free.app/install/dev/build-info.txt

curl -fsS https://166d-210-222-118-66.ngrok-free.app/install/dev/build-info.txt \
    | sed -nE 's/^버전[[:space:]]*:[[:space:]]*//p'
```

모바일에서 호출할 계획이면 궁극적으로는 build-info.txt보다 JSON 파일을 같이 두는 게 더 안전합니다.

```shell
{
  "env": "dev",
  "appName": "Hi HR(DEV)",
  "versionName": "1.0.5",
  "versionCode": 6
}
```

```shell
https://166d-210-222-118-66.ngrok-free.app/install/dev/build-info.json

{
  "appStartUrl": "https://~~~",
  "version": "1.0.5 (6)"
}
```

```shell
❯ curl -fsS https://166d-210-222-118-66.ngrok-free.app/install/dev/build-info.json

{
	"appStartUrl": "https://10.173.4.153:44300/sap/bc/ui5_ui5/sap/zhressapp/index.html?sap-client=100&saml2=disabled&sap-language=KO",
	"version": "1.0.5 (6)"
}
```

이제 이걸 이용해서 모바일 앱에서 활용하면 된다.
