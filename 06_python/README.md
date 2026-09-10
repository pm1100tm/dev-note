# Python

Python 개발에서 자주 사용하는 자료구조, 디버깅, 패키지·가상환경 관리와 면접 대비 주제를 정리한다.
프로젝트별 Python 버전과 의존성 관리 도구는 `pyproject.toml` 또는 팀 표준을 먼저 확인한다.

## 문서

- [heapq](heapq.md)
- [중복 제거에 set을 선호하는 이유](중복제거는set을선호하자.md)
- [딕셔너리 `get`과 `in`의 차이](딕셔너리_get과_in의차이점.md)
- [Miniconda](miniconda.md)
- [uv 패키지 매니저](998_etc_uv_package_manager.md)
- [pdb 단축키](999_etc_pdb_shortcut.md)
- [면접 질문](interview.md)

## 원칙

- 전역 Python 환경보다 프로젝트별 가상환경을 사용한다.
- 의존성 버전은 lock 파일로 고정하고, 운영 배포 전 재현 가능한 설치를 확인한다.
- 성능 최적화는 추측하지 말고 프로파일링 결과를 근거로 적용한다.
