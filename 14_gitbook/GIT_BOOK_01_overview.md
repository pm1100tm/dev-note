# GitBook 개요

> 확인일: 2026-09-11. 기능과 화면은 GitBook 공식 문서 기준이며, 계정·요금제에 따라
> 제공 범위가 달라질 수 있다.

## GitBook이란?

- GitBook은 팀과 개발자가 제품 문서, 개발 가이드, 운영 매뉴얼, API 문서를 작성하고 웹사이트로 게시할 수 있는 문서화
  플랫폼이다.
- 웹에서 블록 기반으로 편집할 수도 있고, Markdown 파일을 GitHub 또는 GitLab 저장소와 동기화하는 docs-as-code
  방식으로 관리할 수도 있다.

이 저장소처럼 문서를 코드와 함께 관리할 때는 Markdown을 Git에 두고, GitBook을 검색과 탐색, 미리 보기, 게시를 제공하는
문서 사이트로 사용하는 구성이 실용적이다.

## 왜 사용하는가?

- 문서 탐색: 계층형 목차와 검색으로 긴 Markdown 모음을 문서 사이트로 제공한다.
  이 저장소에서는 `SUMMARY.md`로 학습 노트의 탐색 순서를 관리한다.
- 협업 검토: 변경 요청과 미리 보기로 게시 전 문서 품질을 확인할 수 있다.
  링크·이미지·목차 변경을 검토한 뒤 공개한다.
- Git 기반 관리: GitHub·GitLab과 동기화해 코드와 문서의 변경 이력을 함께 관리한다.
  pull request 또는 commit으로 문서를 검토할 수 있다.
- 낮은 작성 진입 장벽: 시각 편집과 Markdown 방식을 함께 사용할 수 있다.
  개발자는 로컬 편집기를, 비개발자는 웹 편집기를 선택할 수 있다.
- 게시와 외부 공유: 문서를 사이트로 게시하고 도메인·공개 범위를 설정할 수 있다.
  개인 학습 노트를 읽기 쉬운 형태로 공유한다.

GitBook은 문서의 내용을 자동으로 정확하게 만드는 도구는 아니다. 최신성, 기술적 정확성, 보안 정보 제외, 링크 유효성은
작성자와 리뷰 과정에서 계속 확인해야 한다.

## 핵심 구성

- Site: 방문자에게 게시하는 문서 사이트와 탐색 구조를 관리한다.
- Space: 문서 콘텐츠를 담는 단위다. 하나의 Site는 여러 Space를 가질 수 있다.
- Change request: 편집 내용을 검토하고 병합하는 변경 단위다.
- Git Sync: 저장소의 브랜치·디렉터리와 GitBook 콘텐츠를 동기화하는 기능이다.
- `SUMMARY.md`: 한 Space의 문서 탐색 순서를 정의하는 Markdown 목차 파일이다.
- `gitbook-docs.yaml`: Site 구조와 Space별 저장소 디렉터리 매핑을 설정하는 파일이다.
- `.gitbook.yaml`: 개별 Space의 콘텐츠 읽기 설정을 지정하는 파일이다.

`gitbook-docs.yaml`의 Space `key`는 제목이나 경로가 아니라 기존 Space를 식별하는
값이다. 연결 후 임의로 바꾸면 GitBook은 새 Space로 인식할 수 있으므로 변경 전 영향을
확인한다.

## 작성·배포 흐름

```shell
Markdown 수정 → Git commit 또는 pull request → Git Sync → GitBook Preview → 게시
```

- 웹 편집을 선택하면 GitBook Change request에서 수정·Preview·Merge를 진행한다.
- Git 기반 편집을 선택하면 저장소에서 Markdown을 수정하고 pull request를 통해 검토한다.
- 병합된 GitHub 변경은 GitBook으로 동기화되며, 게시된 Site는 최신 콘텐츠를 제공한다.
- 최초 동기화에서는 어느 쪽을 원본으로 삼을지 반드시 확인한다. 빈 저장소를 GitBook
  콘텐츠의 원본으로 선택하면 기존 문서가 대체될 수 있다.

## 사용할 때 판단 기준

GitBook은 외부에 제공할 개발 문서, 여러 사람이 함께 유지하는 운영 문서, 코드 변경과
연결되는 API·사용자 가이드에 적합하다. 반대로 민감한 내부 정보, 자주 바뀌는 비공개
작업 메모는 공개 범위와 접근 제어를 확인한 뒤 사용해야 한다.

## 참고 자료

- [GitBook Quickstart](https://gitbook.com/docs/getting-started/quickstart)
- [GitHub Sync 활성화](https://gitbook.com/docs/docs-as-code/git-sync/enabling-github-sync)
- [Git Sync 콘텐츠 설정](https://gitbook.com/docs/docs-as-code/git-sync/content-configuration)
