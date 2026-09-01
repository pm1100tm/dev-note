# 🚀 iOS 앱 인증서(.p12) Mac Keychain 등록 방법

- macOS의 `키체인 접근` 앱을 사용

> ✍️ 터미널로 비밀번호를 입력하는 방식은 shell history 노출 위험
> 이 있어 초보자에게 권장하지 않습니다.

## 1. 준비물

- .p12 확장자 파일
- 해당 파일의 비밀번호

> ✏️ .p12 비밀번호를 모르면 설치할 수 없습니다. Apple 계정, Mac 로그인 비밀번호와는 별도

> ✏️ 기존 .p12의 비밀번호를 Apple Developer Center에서 확인하거나 초기화할 수는 없습니다

## 2. 파일 권한 보호 - .p12

터미널에서 해당 파일의 권한을 변경하는 것을 권장합니다.

```shell
chmod 600 인증서.p12

# 확인
ls -l 인증서.p12

-rw-------
```

## 3. 키체인 등록

- 1. Command + Space를 눌러 Spotlight를 엽니다.
- 2. 키체인 접근 또는 Keychain Access를 입력합니다.
- 3. 키체인 접근 앱을 실행합니다.
- 4. 왼쪽 상단 키체인 목록에서 로그인 또는 login을 선택합니다.
- 5. 카테고리에서 나의 인증서 또는 My Certificates를 선택합니다.
- 6. 메뉴 > 파일 > Add Keychain > 인증서.p12 선택
- 7. 대상 키체인은 로그인을 선택합니다.
- 8. .p12 비밀번호를 입력합니다.

* 여기서 입력하는 비밀번호는 Mac 로그인 비밀번호가 아니라 .p12 내보내기 비밀번호입니다.

> 시스템 키체인이 아니라 개인 계정의 로그인 키체인에 설치하는 것이 일반적입니다.

## 4. 인증서와 개인 키 확인

- 키체인 접근의 나의 인증서에서 등록한 항목을 찾습니다.
- 인증서 왼쪽에 작은 화살표가 있으면 펼칩니다.
- 아래에 개인 키 또는 private key가 표시되어야 합니다.
  - 인증서만 있고 개인 키가 없다면 서명할 수 없습니다.

이 경우 가능한 원인은 다음과 같습니다.

- .p12에 개인 키가 포함되지 않음
- 인증서 파일만 잘못 전달받음
- 가져오기 과정에서 오류 발생
- 다른 키체인에 개인 키가 설치됨

> 인증서와 개인 키가 함께 들어 있는 올바른 .p12가 필요합니다.

## 5. 인증서 유효기간 확인

키체인 접근에서 인증서를 더블클릭합니다.

다음을 확인합니다.

- 인증서 이름: iPhone Distribution: ~~~
- Team ID/OU: ~~~
- 만료일: ~~~
- 인증서 상태: 유효함

신뢰 설정은 임의로 항상 신뢰로 변경하지 마십시오. 기본 시스템 설정을 유지하는 것이 안전합니다.

## 6. 터미널에서 signing identity 확인

터미널에서 다음 명령을 실행합니다.

```shell
security find-identity -v -p codesigning
```

정상 예시:

```shell
1) B1B05DF46D6699121BFE6743F3850F3126424176 "iPhone Distribution: FOO BAR"
2) 90E4D31963C80485F7F1018C3FC136DFF32344F5 "Apple Development: FOO (BAR)"
  2 valid identities found
```

핵심은 마지막 줄입니다.

```shell
2 valid identities found
```

## 7. 인증서 지문 확인

설치한 인증서가 프로비저닝 프로파일에 포함된 인증서와 동일한지 확인할 수 있습니다.

> LLM 사용하여 인증서와 프로비저닝 프로파일의 지문(보통 SHA-256)을 확인하는 cmd 찾기

## 8. 최초 빌드 시 개인 키 접근 허용

Xcode 또는 별도의 shell script로 처음 서명할 때 다음과 비슷한 창이 나타날 수 있습니다.

```
codesign이 키체인에 있는 개인 키에 접근하려고 합니다.
```

이 경우:

- 1. 요청 프로그램이 codesign 또는 Xcode 관련 프로세스인지 확인합니다.
- 2. Mac 로그인 비밀번호를 입력합니다.
- 3. 회사 보안 정책상 허용된다면 항상 허용을 선택합니다.
- 4. 출처를 알 수 없는 프로그램이면 허용하지 않습니다.

개인 키의 접근 제어에서 “모든 응용 프로그램의 접근 허용”을 직접 설정하는 것은 권장하지 않습니다.

## 9. 자주 발생하는 오류

### .p12 비밀번호가 틀렸다고 나오는 경우

Apple ID나 Mac 로그인 비밀번호가 아니라 .p12 생성 시 설정한 비밀번호가 필요합니다.

### 설치했지만 0 valid identities found

확인할 사항:

- 인증서 아래에 개인 키가 있는지
- 로그인 키체인이 잠겨 있지 않은지
- 인증서가 만료되지 않았는지
- 인증서가 신뢰되지 않는 상태인지
- 같은 인증서를 인증서 카테고리에서만 보고 있지 않은지
- 나의 인증서 카테고리에 표시되는지

### 개인 키가 보이지 않는 경우

현재 .p12에 개인 키가 없는 것입니다. 원본 인증서를 발급하거나 내보낸 Mac에서 개인 키를 포함해 다시 내보내야 합니다.

### 인증서가 이미 존재한다고 나오는 경우

기존 항목을 바로 삭제하지 마십시오. 기존 인증서를 사용하는 다른 프로젝트가 있을 수 있습니다.

먼저 다음을 비교해야 합니다.

- 만료일
- SHA-256 지문
- 개인 키 포함 여부

## 10. 프로비저닝 프로파일 설치

프로비저닝 프로파일을 Xcode가 인식하도록 설치합니다. 프로파일 파일이 프로젝트에 존재하는 것만으로는
충분하지 않습니다. Xcode 프로파일 저장소에 설치되어야 합니다.

Xcode 프로파일 저장소에 설치 방법:

먼저 Xcode가 열려 있다면 종료한 다음, 터미널에서 아래 명령을 실행해 주세요.

```shell
mkdir -p "$HOME/Library/Developer/Xcode/UserData/Provisioning Profiles"
```

이제 프로파일을 Xcode 저장소에 UUID 파일명으로 복사합니다.

```shell
cp "/Users/<username>/Desktop/work/ios/config/HLAHiHR.mobileprovision" \
"$HOME/Library/Developer/Xcode/UserData/Provisioning Profiles/<UUID>.mobileprovision"
```

이제 설치된 프로파일을 검증합니다. 터미널에서 실행해 주세요.

```shell
security cms -D -i \
"$HOME/Library/Developer/Xcode/UserData/Provisioning Profiles/<UUID>.mobileprovision" \
| plutil -p - \
| grep -E '"Name"|"UUID"|"TeamIdentifier"|"application-identifier"|"ProvisionsAllDevices"|"ExpirationDate"'
```

정상인 경우 TEAM ID 값 출력

```
<TEAM ID>
```

---

## 트러블슈팅

인증서를 keychain 에 등록했지만, 터미널에서 signing identity 확인 시, 등록된 것이 없다고
확인되었습니다.

```shell
security find-identity -v -p codesigning

0 valid identities found
```

### 원인

원인은 Apple WWDR G3 중간 인증서 누락입니다.

확인 결과:

- 배포 인증서 존재:
  iPhone Distribution: ~~~

- 유효기간 정상:
  2025-09-15 ~ 2028-09-14 UTC

- 개인키 존재 및 공개키 식별자 일치:
  5716B417...ABC0504B

- 로그인 키체인 잠금/검색 목록 정상
- 사용자 지정 신뢰 설정 충돌 없음
- 하지만 시스템에는 구형 WWDR 인증서만 있고, 배포 인증서가 요구하는 OU=G3 중간 인증서가 없음

.p12 가져오기 성공은 인증서와 개인키를 등록했다는 뜻일 뿐, 전체 신뢰 체인까지 유효하다는 뜻은 아닙니다.

### 해결방법

- Apple WWDR G3 인증서 (https://www.apple.com/certificateauthority/AppleWWDRCAG3.cer)
- 다운로드한 AppleWWDRCAG3.cer를 더블클릭해 로그인 키체인 또는 시스템 키체인에 추가하세요.
- 키체인 접근에서 신뢰 설정은 임의로 항상 신뢰로 바꾸지 말고 시스템 기본값 사용으로 둡니다.

설치 후 확인:

```shell
security find-identity -v -p codesigning

1) 5DAB6A5E... "iPhone Distribution: ~~~"
  1 valid identities found
```

그래도 0개라면 키체인 접근에서 해당 배포 인증서를 펼쳤을 때 Imported Private Key가 하위에
표시되는지와, 인증서 상단에 “이 인증서는 유효합니다”가 표시되는지 확인하면 됩니다.

Apple도 소프트웨어 서명 인증서에는 WWDR G3가 사용되며 2030년까지 유효한 갱신 인증서를 설치해야
한다고 안내합니다.

- Apple WWDR 인증서 안내 (https://developer.apple.com/help/account/certificates/wwdr-intermediate-certificates)
- Apple PKI (https://www.apple.com/certificateauthority/)
