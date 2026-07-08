# VSCODE 저장 시 자동 Prettier 포메팅

코드 저장(cmd + s) 할 때 Prettier가 자동으로 적용되도록 하려면, 확장 프로그램 설치 후 설정 창에서
'저장 시 포맷'과 '기본 포매터'를 지정해주면 됩니다.

<br>

## Prettier 확장 프로그램 설치

VS Code 확장 프로그램 탭을 열고 Prettier - Code formatter를 검색하여 설치합니다.

<br>

## 기본 포맷터 지정

- Cmd + Shift + P를 누르고 Preferences: Open User Settings를 입력
- default formatter
- Editor: Default Formatter 항목을 esbenp.prettier-vscode로 변경합니다

<br>

## 저장 시 포맷 설정

- 설정 검색창에 format on save를 검색합니다.
- Editor: Format On Save 항목을 체크합니다.
