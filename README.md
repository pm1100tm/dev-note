# 개발 학습 노트

개발자로서 학습하고, 운영 환경에서 경험한 문제와 해결 과정을 기록하는 GitBook 문서 저장소입니다.
개념을 단순히 나열하기보다, 실제 개발과 운영에서 다시 활용할 수 있는 기준과 예시를 남기는 것을 목표로 합니다.

## 문서 보기

GitBook은 [SUMMARY.md](SUMMARY.md)를 기준으로 문서의 탐색 목차를 생성합니다.
전체 주제와 세부 문서는 목차에서 확인할 수 있습니다.

- [전체 목차 보기](SUMMARY.md)
- [작성할 항목 보기](TODO.md)

## 주요 주제

- [알고리즘](01_algorithms/README.md)
- [데이터베이스](02_database/README.md)
- [Docker](03_docker/README.md)
- [Git](04_git/README.md)
- [Java](05_java/README.md)
- [Python](06_python/README.md)
- [React](07_react/README.md)
- [AWS](08_aws/README.md)
- [GCP](08_gcp/README.md)
- [Kubernetes](08_kubernetes/README.md)
- [CI/CD](08_cicd/README.md)
- [디자인](10_design/README.md)
- [AI Agent](11_ai_agent/README.md)
- [Android](12_android/README.md)
- [iOS](13_ios/README.md)
- [GitBook](14_gitbook/README.md)
- [MacBook 초기 설정](91_macbook_initial_setting/README.md)
- [용어와 개념](99_term-and-concept/README.md)
- [기타](99_etc/README.md)

## 문서 작성 원칙

- 문서는 가능한 한 한글로 작성하고, 처음 접하는 사람도 이해할 수 있게 배경과 예시를 함께 적습니다.
- 제목과 파일명은 주제를 명확히 드러내도록 정하고, 문서 이동 경로는 상대 경로를 사용합니다.
- 새 문서를 추가하거나 문서 위치를 바꾸면 `SUMMARY.md`의 링크도 함께 갱신합니다.
- 이미지와 첨부 파일은 문서와 가까운 `assets` 디렉터리에 두고, 참조 경로가 유효한지 확인합니다.
- 계정, 비밀번호, 접근 키 같은 민감한 정보는 문서와 예시에 포함하지 않습니다.

## 문서 반영 절차

- [1.] 주제에 맞는 디렉터리에 Markdown 문서를 작성하거나 수정합니다.
- [2.] 새 문서이거나 경로가 바뀐 경우 `SUMMARY.md`에 제목과 링크를 추가합니다.
- [3.] 상대 링크, 이미지 경로, 코드 예시를 확인합니다.
- [4.] 변경 사항을 검토한 뒤 기본 브랜치에 반영하면 GitBook 동기화 설정에 따라 문서가 갱신됩니다.
- [5.] 연동 테스트
