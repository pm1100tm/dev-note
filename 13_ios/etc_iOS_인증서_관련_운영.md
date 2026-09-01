# iOS 애플리케이션 인증서(디지털 서명) 관리

- 앱을 만든 개발자 또는 조직의 신원을 증명
- 앱 위변조를 방지하는 디지털 서명
- iOS 설치 파일(.ipa)을 생성할 때 Apple 배포 인증서와 프로비저닝 프로파일 사용
- 이 프로젝트는 Enterprise 배포 방식으로 서명된 .ipa 파일을 생성

## 서명 정보 및 보관

- `인증서_2028.p12` (인증서)
- `<app>.mobileprovision` (프로비저닝 프로파일)

배포 인증서 또는 프로비저닝 프로파일이 만료되면 새 IPA를 만들거나 설치할 수 없습니다. 이미 기기에
설치된 앱은 계속 실행될 수 있지만, 새 버전 업데이트 설치는 실패할 수 있습니다.

같은 앱을 계속 업데이트하려면 동일한 Bundle ID와 Team ID를 유지하고, 유효한 배포 인증서와
프로비저닝 프로파일로 .ipa 파일을 다시 생성해야 한다.

### 인증서.p12 파일

- Apple 배포 인증서와 그에 대응하는 private key(비밀번호)를 같이 담은 파일
- Xcode가 앱에 실제로 서명할 때 사용
- 이 파일이 없다면 `누가 서명했는지` 증명할 수 없어 빌드나 export 단계 실패
- 보통 비밀번호가 걸려있고, 빌드 Mac의 Keychain에 import 하여 사용

### <>.mobileprovision 파일

- Apple이 발급하는 프로비저닝 프로파일
- 이 앱을 어떤 Bundle ID, Team ID, 어떤 권한(entitlements), 어떤 방식으로 설치 허용할지
  정해둔 허가서
- 안에는 보통 다음 정보가 들어있음
  - App ID(Bundle ID)
  - Team ID
  - 허용된 인증서 목록
  - 앱 권한 정보
  - 배포 방식 정보
  - 필요하면 허용 기기 목록
  - 만료일
- 이 파일이 없거나 만료되면, 인증서가 있어도 해당 설정으로는 앱을 서명하거나 설치할 수 없습니다.
- 보통 이 파일은 인증서보다 만료기간이 짧아(약 1년), 만료 전 갱신해야 함

### 둘의 관계를 한 줄로 정리

- .p12는 서명하는 주체이고
- .mobileprovision은 어떤 앱을 어떤 조건으로 배포할 수 있는지를 정하는 파일입니다.

즉 iOS 서명 빌드는 보통 이렇게 맞아야 합니다.

- .p12의 인증서가 유효할 것
- .p12에 private key가 포함되어 있을 것
- .mobileprovision이 그 인증서를 허용할 것
- Bundle ID가 프로파일과 일치할 것
- Team ID가 일치할 것
- 필요한 capability가 프로파일에 포함될 것

## 앱 업데이트

인증서 또는 프로비저닝 프로파일이 만료되기 전에 갱신하여 사용합니다. 앱 업데이트가 가능하도록 하려면,
다음의 정보가 일치해야 합니다.

- Bundle ID가 동일해야 함
- Team ID가 동일해야 함
- 새 mobileprovision이 새 인증서를 포함하고 있어야 함
- 앱의 capability / entitlement 구성이 기존과 충돌하지 않아야 함
- 배포 방식이 기존 설치 방식과 맞아야 함

### 배포 방식

- development
- ad-hoc
- app-store
- enterprise
- Developer ID는 iOS가 아니라 macOS 전용입니다

정리하면,

- development
  - 개발/테스트용
  - 등록된 기기에 설치
  - 일반 사용자 배포용은 아님

- ad-hoc
  - App Store 심사 없이
  - 등록한 제한된 기기들에만 설치
  - 기기 UDID 등록이 필요

- app-store
  - App Store, TestFlight, 그리고 App Store Connect를 통한 배포에 사용
  - 일반 사용자 배포의 표준 방식
  - 공개 앱, 비공개 커스텀 앱, unlisted 앱도 여기 계열로 봄

- enterprise
  - Apple Developer Enterprise Program에서만 사용
  - 조직 내부 직원용 in-house 앱 배포
  - 외부 고객용 배포용으로 쓰면 안 됨

App Store 쪽은 다시 나뉩니다.

- Public Distribution: App Store 공개
- Private Distribution: Apple Business Manager / Apple School Manager용 커스텀 앱
- Unlisted: App Store에는 안 뜨고 직접 링크로만 접근
- TestFlight: 베타 테스트 배포

핵심만 말하면, 지금 당신 프로젝트의 IOS_EXPORT_METHOD에 대응되는 실제 값은 보통 이 4개입니다.

`development / ad-hoc / app-store / enterprise`

#### REF

- Xcode 배포 메서드: Apple Developer Documentation - Distributing your app for beta testing and releases
- 등록 기기 배포: Apple Developer Documentation - Distributing your app to registered devices
- Enterprise 배포: Apple Developer Enterprise Program
- App Store 공개/비공개/Unlisted: App Store Connect - Set distribution methods
- TestFlight: TestFlight overview

---

<br>

## 인증서 및 프로파일 만료일 확인

배포 인증서는 빌드 Mac의 Keychain Access 또는 아래 명령으로 확인한다.

```shell
security find-identity -v -p codesigning
```

프로비저닝 프로파일은 빌드 Mac의 아래 경로에서 확인한다.

```shell
ls ~/Library/MobileDevice/Provisioning\ Profiles
```

특정 프로파일의 상세 정보는 아래 명령으로 확인한다.

```shell
security cms -D -i ~/Library/MobileDevice/Provisioning\ Profiles/<profile-uuid>.mobileprovision
```

운영 문서에는 확인한 값을 별도로 기록한다.

- 인증서 이름:
- 인증서 Team ID:
- 인증서 만료일:
- 프로비저닝 프로파일 이름:
- 프로비저닝 프로파일 UUID:
- 프로비저닝 프로파일 만료일:
- 대상 Bundle ID:
- 배포 방식:

## 운영 시 IPA 파일(iOS 앱 설치 파일) 업데이트 시 체크 항목

- Bundle ID가 기존 앱과 동일한지 확인한다.
- Team ID가 기존 앱과 동일한지 확인한다.
- 프로비저닝 프로파일에 Bundle ID가 포함되어 있는지 확인한다.
- 배포 인증서와 private key(인증서 비번)가 빌드 Mac의 Keychain에 함께 설치되어 있는지 확인한다.
- 프로비저닝 프로파일과 배포 인증서가 만료되지 않았는지 확인한다.
- `IOS_EXPORT_METHOD=enterprise`로 빌드했는지 확인한다.
- 새 IPA를 만드는 경우, 기존 설치된 앱 위에 업데이트 설치가 되는지 실기기에서 확인한다.
- 배포 인증서, private key, 프로비저닝 프로파일의 원본과 백업본을 분리 보관한다.
- 인증서 private key가 유실되면 동일 인증서로 서명할 수 없으므로 담당자 개인 PC에만 보관하지 않는다.
