# iOS 앱 엔터프라이즈 서명으로 웹에서 다운로드 및 설치

Apple은 ***Enterprise 앱 배포에 MDM 사용을 권장***합니다. MDM으로 설치된 앱은 관리 대상 앱으로
설치되며, 개발자 신뢰 절차가 자동으로 처리될 수 있습니다.

반면 웹사이트를 통한 수동 설치 방식은 사용자가 직접 설치 링크를 누르고,
**_설치 후 iOS 설정에서 Enterprise 개발자를 신뢰해야 할 수 있습니다._**

## 📒 Enterprise 배포의 핵심 전제

Enterprise 배포는 일반 사용자에게 앱을 배포하기 위한 우회 수단이 아닙니다.

Apple Developer Enterprise Program은 조직 내부에서만 사용하는 전용 앱을 직원에게 비공개로
배포하기 위한 프로그램입니다.

Apple Developer Enterprise Program을 사용하려면 일반적으로 다음 조건을 만족해야 합니다.

- 조직 내부 사용 목적의 전용 앱이어야 합니다.
- 앱은 해당 조직이 직접 개발한 앱이어야 합니다.
- 직원에게만 비공개로 배포해야 합니다.
- 안전한 내부 시스템 또는 MDM을 통해 배포해야 합니다.
- 조직은 직원이 100명 이상인 법인이어야 합니다.
- 직원 외부 사용자가 앱을 다운로드하지 못하도록 통제할 수 있어야 합니다.
- Apple의 검증 및 지속 평가 절차를 통과해야 합니다.

따라서 Enterprise 서명 앱을 외부 고객, 일반 사용자, 불특정 다수에게 배포하면 Apple 정책 위반이
될 수 있습니다.

정책 위반이 발생하면 Enterprise 인증서가 폐기될 수 있으며, 해당 인증서로 서명된 앱들이 더 이상
실행되지 않을 수 있습니다.

---

## 웹 설치 방식의 동작 구조

Enterprise 앱을 웹에서 설치하는 방식은 `.ipa` 파일을 직접 여는 방식이 아닙니다.
일반적으로 다음 흐름으로 동작합니다.

- 1. Enterprise 인증서와 In-House 프로비저닝 프로파일로 앱을 빌드하여 `.ipa` 파일을 생성합니다.
- 2. `.ipa` 파일을 HTTPS 서버에 업로드합니다.
- 3. `.ipa` 파일 정보를 담은 manifest plist 파일을 작성합니다.
- 4. manifest plist 파일도 HTTPS 서버에 업로드합니다.
- 5. 웹 페이지에 `itms-services` 스킴을 사용하는 설치 링크를 제공합니다.
- 6. 사용자가 iPhone 또는 iPad에서 설치 링크를 탭합니다.
- 7. iOS가 manifest plist를 읽고 `.ipa`를 다운로드하여 설치를 시도합니다.
- 8. 수동 설치인 경우 사용자가 설정에서 Enterprise 개발자를 신뢰해야 앱을 실행할 수 있습니다.

핵심은 사용자가 `.ipa` 파일을 직접 다운로드해서 여는 것이 아니라, iOS가 `itms-services` 링크와
manifest plist를 통해 설치 절차를 수행한다는 점입니다.

## 필요한 구성 요소

웹 기반 Enterprise 설치를 구성하려면 다음 요소가 필요합니다.

- Apple Developer Enterprise Program 계정
  - Apple Developer Program 계정만으로는 Enterprise In-House 일반 배포를 할 수 없습니다.
- 앱은 Enterprise 배포용 인증서로 서명되어야 합니다.
  - 개발용 인증서, App Store 배포용 인증서, Ad Hoc 배포용 인증서와 목적이 다릅니다.
- In-House 프로비저닝 프로파일
  - Ad Hoc 배포와 달리 설치 대상 기기의 UDID를 미리 등록하지 않습니다.
  - 하지만 이것이 아무 사용자에게나 배포해도 된다는 의미는 아닙니다. 배포 대상은 조직 내부 직원으로
    제한되어야 합니다.
- `.ipa` 파일
  - Enterprise 방식으로 export한 `.ipa` 파일이 필요합니다.
  - 이 `.ipa`에는 올바른 코드 서명과 프로비저닝 프로파일이 포함되어 있어야 합니다.
- HTTPS 서버
  - `.ipa`, manifest plist, 아이콘 이미지는 HTTPS로 접근 가능해야 합니다.
  - 사내망에서만 배포하는 경우에도 iOS 기기가 해당 HTTPS 서버에 접근할 수 있어야 합니다.
- manifest plist
  - manifest plist는 iOS에게 어떤 앱을 어디서 내려받아 설치할지 알려주는 XML 파일입니다.
  - 이 파일에는 `.ipa` 파일 URL, 앱 번들 ID, 버전, 앱 이름, 아이콘 정보 등이 포함됩니다.

---

<br>

## 설치 웹 페이지 예시

```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>사내 앱 설치</title>
  </head>
  <body>
    <h1>사내 앱 설치</h1>
    <p>이 앱은 조직 내부 직원 전용 앱입니다.</p>
    <a
      href="itms-services://?action=download-manifest&url=https://example.com/apps/manifest.plist"
    >
      iOS 앱 설치
    </a>
  </body>
</html>
```

- 실제 운영 환경에서는 이 페이지를 반드시 인증된 사용자만 접근할 수 있게 보호해야 합니다.
- 예를 들어 사내 SSO, VPN, 사내망 접근 제한, 다운로드 토큰, 만료 링크 등을 적용해야 합니다.

## manifest plist 예시

아래는 Enterprise 앱 웹 설치에 사용하는 manifest plist 예시입니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>items</key>
  <array>
    <dict>
      <key>assets</key>
      <array>
        <dict>
          <key>kind</key>
          <string>software-package</string>
          <key>url</key>
          <string>https://example.com/apps/my-app.ipa</string>
        </dict>
        <dict>
          <key>kind</key>
          <string>display-image</string>
          <key>needs-shine</key>
          <false/>
          <key>url</key>
          <string>https://example.com/apps/icon-57.png</string>
        </dict>
        <dict>
          <key>kind</key>
          <string>full-size-image</string>
          <key>needs-shine</key>
          <false/>
          <key>url</key>
          <string>https://example.com/apps/icon-512.png</string>
        </dict>
      </array>
      <key>metadata</key>
      <dict>
        <key>bundle-identifier</key>
        <string>com.example.myapp</string>
        <key>bundle-version</key>
        <string>1.0.0</string>
        <key>kind</key>
        <string>software</string>
        <key>title</key>
        <string>My Enterprise App</string>
      </dict>
    </dict>
  </array>
</dict>
</plist>
```

- `software-package`의 `url`은 실제 `.ipa` 파일 URL입니다.
- `bundle-identifier`는 앱의 Bundle Identifier와 일치해야 합니다.
- `bundle-version`은 배포하는 앱 버전입니다.
- `title`은 설치 화면에 표시될 앱 이름입니다.

## 앱 설치

- 사용자가 iPhone 또는 iPad의 Safari에서 이 링크를 탭하면 iOS가 manifest plist를 읽고 앱 설치를
  시도합니다.
- 주의할 점은 `url` 파라미터에 들어가는 manifest plist URL도 HTTPS여야 한다는 점입니다.

## 사용자 설치 후 신뢰 절차

웹사이트에서 수동으로 Enterprise 앱을 설치한 경우, 사용자가 앱을 처음 실행할 때 신뢰되지 않은
개발자라는 메시지가 표시됩니다.
이 경우 사용자는 iOS 설정에서 Enterprise 개발자를 신뢰해야 합니다.

일반적인 절차는 다음과 같습니다.

- 1. 앱 설치 후 앱을 실행합니다.
- 2. 신뢰되지 않은 Enterprise 개발자 메시지가 표시되면 취소합니다.
- 3. 설정 앱을 엽니다.
- 4. `일반`으로 이동합니다.
- 5. `VPN 및 기기 관리`로 이동합니다.
- 6. Enterprise App 섹션에서 개발자 이름을 선택합니다.
- 7. 해당 개발자를 신뢰합니다.
- 8. iOS 18, iPadOS 18, visionOS 2 이상에서는 `허용 및 재시동` 절차가 필요할 수 있습니다.
- 9. 기기가 재시동된 뒤 화면 안내에 따라 신뢰 절차를 완료합니다.

개발자를 한 번 신뢰하면 같은 개발자가 서명한 다른 Enterprise 앱은 바로 실행될 수 있습니다.
다만 iOS는 Enterprise 개발자 인증서를 주기적으로 재검증합니다. 재검증을 위해 기기가 인터넷에
연결되어 있어야 할 수 있습니다.

<br>

## 유의 사항

### Bundle Identifier와 manifest 값 일치

- manifest plist의 `bundle-identifier`는 실제 앱의 Bundle Identifier와 일치해야 합니다.
- 값이 다르면 설치 실패 또는 업데이트 실패가 발생할 수 있습니다.

### 인증서 만료 관리

- Enterprise 인증서나 프로비저닝 프로파일이 만료되면 앱 설치 또는 실행에 문제가 발생할 수 있습니다.
- 운영 환경에서는 만료일을 반드시 모니터링해야 합니다.
- 인증서를 갱신할 때는 새 인증서로 앱을 다시 서명하고 재배포하는 절차를 준비해야 합니다.

### 인증서 폐기 리스크

- Apple이 Enterprise 인증서를 폐기하면 해당 인증서로 서명된 앱은 더 이상 신뢰되지 않을 수 있습니다.
- 특히 외부 배포, 불특정 다수 배포, 앱 마켓처럼 사용하는 방식은 정책 위반 리스크가 큽니다.

### 업데이트 정책

- 웹 수동 설치 방식은 App Store처럼 자동 업데이트 체계가 기본 제공되지 않습니다.
- 업데이트를 제공하려면 새 버전의 `.ipa`와 manifest plist를 배포하고, 사용자가 다시 설치하도록
  안내해야 합니다.
- 앱 내부에서 버전 체크 API를 호출해 업데이트 페이지로 이동시키는 방식을 별도로 구현할 수 있습니다.

---

<br>

## 설치 실패 시 확인할 항목

설치가 실패하면 다음 항목을 우선 확인합니다.

- `.ipa`가 Enterprise 인증서로 올바르게 서명되어 있는지 확인합니다.
- In-House 프로비저닝 프로파일이 포함되어 있는지 확인합니다.
- manifest plist의 XML 문법이 올바른지 확인합니다.
- manifest plist의 `.ipa` URL이 HTTPS인지 확인합니다.
- `.ipa` 파일 URL에 iOS 기기에서 접근 가능한지 확인합니다.
- 서버 인증서가 iOS에서 신뢰 가능한 인증서인지 확인합니다.
- `bundle-identifier` 값이 실제 앱과 일치하는지 확인합니다.
- `bundle-version` 값이 의도한 버전인지 확인합니다.
- 설치 링크가 Safari에서 열리고 있는지 확인합니다.
- 사내망, VPN, 방화벽 정책이 다운로드를 막고 있지 않은지 확인합니다.
- 설치 후 개발자 신뢰 절차가 완료되었는지 확인합니다.
- Apple 검증 서버 접근이 가능한지 확인합니다.

## 보안적으로 권장하는 배포 구조

운영 환경에서는 다음 구조를 권장합니다.

```text
사용자
-> 사내 SSO 로그인
-> 설치 페이지 접근
-> 권한 확인
-> 만료 시간이 짧은 manifest URL 발급
-> manifest plist 다운로드
-> 만료 시간이 짧은 ipa URL 접근
-> iOS 설치
```

다운로드 권한은 최소한 다음 기준으로 검증하는 것이 좋습니다.

- 사용자가 조직 내부 직원인지 확인합니다.
- 사용자가 해당 앱을 설치할 권한이 있는지 확인합니다.
- 다운로드 요청 시각과 IP, User-Agent를 로그로 남깁니다.
- 다운로드 링크에 만료 시간을 설정합니다.
- 퇴사자나 권한 해제 사용자는 즉시 접근하지 못하게 합니다.

## 정리

Enterprise 서명으로 iOS 앱을 웹에서 설치하는 것은 가능합니다. 하지만 이것은 `.ipa` 파일을 단순히
웹에서 다운로드해서 실행하는 방식이 아닙니다.

정확한 구조는 Enterprise 인증서로 서명된 `.ipa`를 HTTPS 서버에 올리고, manifest plist와
`itms-services` 링크를 통해 iOS 설치 프로세스를 호출하는 방식입니다.

이 방식은 조직 내부 직원에게 사내 전용 앱을 배포하기 위한 제한적인 방법입니다.

일반 사용자, 고객, 불특정 다수에게 앱을 배포하기 위한 목적으로 사용하면 Apple 정책 위반이 될 수 있습니다.

운영 환경에서는 가능하면 MDM을 사용하는 것이 좋습니다.

웹 수동 설치 방식을 사용해야 한다면 설치 페이지, manifest plist, `.ipa` 파일에 대한 접근 통제와
인증서 만료 관리, Apple 정책 준수 여부를 반드시 관리해야 합니다.

## 참고 문서

- [Apple Developer Enterprise Program](https://developer.apple.com/programs/enterprise/)
- [Install custom enterprise apps on iOS, iPadOS, and visionOS](https://support.apple.com/en-mide/118254)
- [ManifestURL.ItemsItem.AssetsItem](https://developer.apple.com/documentation/devicemanagement/manifesturl/itemsitem/assetsitem)
- [Installing, managing, updating, and removing apps](https://developer.apple.com/documentation/devicemanagement/installing-managing-updating-and-removing-apps)
