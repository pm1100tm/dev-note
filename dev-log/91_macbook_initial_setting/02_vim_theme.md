# Vim gruvbox 테마 설정

## 1. colors 폴더 만들기

```shell
mkdir -p ~/.vim/colors
```

## 2. gruvbox 테마 다운로드

```shell
curl -fLo ~/.vim/colors/gruvbox.vim \
  https://raw.githubusercontent.com/morhetz/gruvbox/master/colors/gruvbox.vim
```

## 3. ~/.vimrc 수정

```shell
vim ~/.vimrc
```

아래 내용을 추가합니다.

```shell
syntax on
set background=dark
colorscheme gruvbox
```

밝은 테마로 쓰고 싶으면:

```shell
syntax on
set background=light
colorscheme gruvbox
```

## 4. 적용 확인

Vim을 다시 열면 적용됩니다.

Vim 안에서 바로 확인하려면:

```shell
:colorscheme gruvbox
```

## 전체 예시

```shell
~/.vimrc:

syntax on
set background=dark
colorscheme gruvbox
```

설치 위치는 이렇게 됩니다.

```shell
~/.vim/colors/gruvbox.vim
```

즉, colorscheme gruvbox는 ~/.vim/colors/gruvbox.vim 파일을 찾아 적용합니다.
