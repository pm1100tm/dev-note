# git fetch VS pull

fetch 와 pull 은 리모트 저장소에서 로컬로 데이터를 가져오기 위한 명령어 입니다.

## git pull

리모트 저장소의 업데이트를 가져오는 동시에 현재 작업하는 로컬 브랜치로 병합합니다. 즉, 현재 브랜치와
원격 저장소를 동기화하여 최신 변경 사항을 즉시 반영합니다.

## git fetch

리모트 저장소의 업데이트를 확인하고 로컬 저장소에 업데이트된 내용을 다운로드합니다. 하지만 이 내용을 현재
작업 중인 브랜치에 병합하지 않습니다. 따라서 git fetch 명령어를 실행한 이후에는 git merge 또는
git rebase 명령어를 사용하여 개발자가 직접 merge를 진행해야 합니다. 이 때, 브랜치는 FETCH_HEAD
의 이름으로 체크아웃이 가능합니다. fetch 이후 merge를 수행하면 pull 명령과 동일한 수행이 됩니다.

## 언제 어떤 명령어를 사용할까

fetch 와 pull을 각각 언제 사용할지에 대해서 알아봅니다.

- pull: 원격 레포가 로컬 레포에 비해 더 최신 커밋이 존재할때만 내려받도록 합니다.
- fetch: 원격 레포와 로컬 레포의 변경 사항이 다를 때 , 이를 비교 및 대조하는 확인 작업이 필요할 때
  사용합니다. 확인 작업을 꼼꼼히 진행한 후에, git merge를 통하여 최신 커밋 내역을 반영하거나 충돌
  문제를 해결할 수 있습니다.
