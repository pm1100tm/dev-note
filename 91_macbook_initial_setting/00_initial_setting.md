# 맥북 초기 설정

## 우클릭 세팅

시스템 환경설정 -> 트랙패스 -> 보조클릭 체크 -> 옵션에 오른쪽 하단 모서리 선택

## 시간 설정

시스템 환경설정 > 날짜 및 시간 > 시간대 설정 현재 위치

## 맞춤법 검사 해제

시스템 환경설정 -> 키보드 -> 맞춤법 자동 수정 체크 해제 -> 맞춤법 -> 한국어 및 영어 전부 해제

## 키 반복 속도 세팅

시스템 환경설정 -> 키보드 -> 반복 및 지연 시간 등 짧게 빠르게

## 언어 설정

일본어 추가

```
언어 바꾸기 단축키 설정
> 키보드 -> 단축기 -> 입력 소스
> 이전 입력 소스 선택에서 cmd + space 로 변경
> Spotlight ctrl + space 로 변경
> 일본어 밑줄 설정
```

## homebrew 설치

```shell
https://brew.sh/index_ko
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

brew update
```

환경 변수 설정:

```shell
export PATH=/opt/homebrew/bin:$PATH
```

기본 z shell 이 설정되어있다면, 홈디렉토리에 .zshrc 파일을 만들어주고
위의 환경 변수 설정 값을 넣어준 후, source ~/.zshrc 해줌

## 수동 다운로드 APP 목록

```
chrome, kakao talk, pycharm, datagrip, web storm, docker, runjs, slack, notion
```

## brew 인스톨 패키지

```
brew install wget
brew install cask
brew install iterm2
brew install —cask sublime-text
brew install —cask visual-studio-code
brew install —cask postman
brew install node (node -v, npm -v)
```

> iterm 을 설치하고 경고 메세지가 나온다면?
>
> Z 쉘 환경변수로 아래의 값을 넣어주고 source
> zsh_disable_compfix="true" 
>
> \*경고 메세지가 나오지 않을 수도 있음

### iterm2 테마설정

- https://github.com/mbadolato/iTerm2-Color-Schemes

```
download -> 압축 풀고 -> iterm preference 에서 프로필 -> 컬러 -> 하단 드롭박스 -> 임포트
*압축해제한 폴더의 schema 전부 임포트해서 테마 바꿀 수 있음
```

### oh-my-zsh 설치

```shell
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

설치하고 나서 brew 환경변수를 한 번 더 잡아줘야 할 수도 있음

### powerlevel10k 설치 후 세팅

```shell
https://github.com/romkatv/powerlevel10k
```

```
1. 위의 링크에서 추천하는 폰트 설치
2. 설치 (git clone —depth=1 ~~)
3. ZSH_THEME=“powerlevel10k/powerlevel10k"
   Z쉘 재시작하면 설정 프롬프트 시작
```

> Oh My zsh, powerlevel10k 삭제 방법은 위의 깃헙에 있음

## 스크롤제한 풀기

```
환경 설정 -> 프로필 -> 터미널 -> 스크롤백 버퍼에서 제한 풀기
```

---

## visual-studio-code extention

```
1. Darcular PyCharm Theme
2. Material Icon Theme
3. Prettier - Code formatter
   cmd ,
   > > quote - single(js), single(ts)
   > > save formatter - check
4. Bracket pair colorizer
5. Indent rainbow
6. auto rename tag
7. css peek
8. live server (cmd + shift + p > live)
9. mark down preview
10. auto import
11. remote ssh
12. code runner (ctrl + option + n)
13. todo highlight (todo, fixme)
14. html to css autocompletion
15. html css support
16. docker
17. docker template
```

### VS Code 단축키 설정

```shell
delete line
-> cmd + d

# 그 외 도움되는 설정
cmd , > Debug › Console: Word Wrap 체크 해제 (debug console auto scroll 기능)
cmd , > Workbench › Tree: Indent (폴더 트리 indent 늘림)
```

## npm

```shell
npm install -g ts-node
```

## JetBrain 프로그램 단축키 설정

```shell
download consolas
edit > font

delete entire line > cmd d
duplicate entire line > option shift down
move line down > option down
move line up > option down

move caret to text start > cmd up
select first row

move caret to text end > cmd down
select last row

move caret to line start > cmd left
move caret to line end > cmd right
move caret to line start with selection > cmd shift left
move caret to line end with selection > cmd shift right

clone caret above, below > option, cmd up down
select next tap > option, cmd right
select previous tap > option, cmd left

> 단축키가 겹치는게 있으면 안먹을 때 있음
```

## MAC

### mac - cmd + l 화면 잠금 단축키 설정

```shell
시스템 환경설정 > 키보드 > 단축키 > 앱 단축키 > + > 화면 잠금 > cmd + l 설정 (Lock Screen)
```
