# IntelliJ Java 코드 컨벤션

팀의 formatter, import 순서, 들여쓰기 규칙은 저장소에 설정 파일로 함께 관리하는 것이 가장
안전하다. IDE 설정만 공유하면 신규 구성원과 CI 환경에서 규칙이 달라질 수 있다.

- IntelliJ의 `Settings > Editor > Code Style > Java`에서 팀 규칙을 적용한다.
- `File > Manage IDE Settings > Export Settings`로 설정을 내보낼 수 있다.
- Checkstyle, Spotless, EditorConfig 등을 빌드에 연결하면 CI에서 일관되게 검증할 수 있다.
- IntelliJ Java 컨벤션 적용 방법은 팀에서 사용하는 formatter 문서를 함께 참고한다.
