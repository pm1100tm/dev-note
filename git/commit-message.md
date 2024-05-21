# git-commit-message template

## 1. .gitmessage 파일 생성하기

```shell
touch .gitmessage
```

## 2. .gitconfig 파일 수정하여 템플릿 적용하기

### Global 적용

```shell
git config --global commit.template [template 파일]
git config --global commit.template .gitmessage
```

### Repository 별 적용

```shell
git config commit.template [template 파일]
git config commit.template .gitmessage
```

## .gitmessage template 예시

```shell
# <branch-name>: <subject>
# - content
# - content

######## Maximum length of content is 72 ########### -> |
# issue track number (#number)
# --- COMMIT END ---
# <type> Title
#   feature : New feature
#   fix     : Fix bug
#   refactor: Refactor code
#   style   : Change style of code(white space, semicolon etc)
#   test    : Add/Change/Delete test case
#   docs    : Add/Change/Delete document
#   build   : Change in build script
#   ci      : Change in ci script
#   chore   : etc
# ------------------
#     Capitalize the subject line
#     Use imperitive mood in subject
#     Do not use period at the end of subject
#     Seperate subject and content with a blank line
#     Use the body explain "what" and "why" vs "how"
#     Use "-" when content contains multiline
# ------------------
```
