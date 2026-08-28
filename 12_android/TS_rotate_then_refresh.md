# TS - Trouble Shooting

이슈 상황:

WebView 로 만든 앱을 에뮬레이터/실기기에서 실행하여, 특정 메뉴 화면을 표시한 후, rotate 를 변경
하면 초기 화면으로 돌아간다.

- 1. 사용자가 로그인
- 2. 첫 화면 진입
- 3. 메뉴 클릭으로 특정 내부 페이지 표시
- 4. 기기 회전
- 5. Android가 MainActivity 재생성
- 6. 새 WebView 또는 복원된 WebView가 다시 로딩
- 7. 웹앱 내부 상태가 초기화됨
- 8. 로그인 세션 쿠키는 살아 있어서 로그인 화면은 안 나오지만, 앱의 첫 화면으로 이동

## 권장 해결방법

해당 프로젝트 기준으로 1차 수정안은 MainActivity 가 회전으로 재생성되지 않도록 하는 것입니다.

```xml
<!-- 기존 -->
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:theme="@style/Theme.App.NoActionBar"
>

<!-- 수정 -->
<activity
    android:name=".MainActivity"
    android:exported="true"
    android:theme="@style/Theme.App.NoActionBar"
    android:configChanges="orientation|screenSize|keyboardHidden"
>
```

영향받는 activity 의 속성 값 configChanges 를 추가합니다.

그리고 필요하면 MainActivity에 onConfigurationChanged()를 추가해서 회전 후 인셋/레이아웃
보정만 처리합니다.

이 방식의 장점은 기존 WebView 인스턴스를 유지하므로 SPA 내부 JavaScript 상태와 현재 화면이
그대로 남을 가능성이 높다는 점입니다. WebView 기반 앱에서는 흔히 쓰는 실용적인 해결책입니다.

다만 주의점도 있습니다.

- layout-land, values-land 같은 회전별 리소스를 자동 재적용하지 않습니다.
- 회전마다 완전히 다른 네이티브 레이아웃을 써야 하는 앱이면 부적합할 수 있습니다.
- 현재 프로젝트는 화면 대부분이 WebView라 이 리스크는 상대적으로 작습니다.

## 더 근본적인 해결

웹앱 쪽에서 메뉴 이동 시 URL hash/path를 명확히 갱신하고, 새로고침/재진입 시 그 URL로 동일
페이지를 복원하게 만드는 것입니다.

예를 들어 /home만 유지하고 내부 상태로만 페이지를 바꾸면 Android 쪽에서 아무리 restoreState를
해도 첫 화면으로 돌아갈 수 있습니다.
