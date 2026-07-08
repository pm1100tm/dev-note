# VSCODE 단축키 설정(Macbook)

## cmd + n > 파일 생성, cmd + shipt + n > 폴더 생성

- Cmd + Shift + P
  - Preferences: Open Keyboard Shortcuts (JSON) 실행

- 다음 내용을 keybindings.json에 추가

```json
[
  {
    "key": "cmd+n",
    "command": "explorer.newFile",
    "when": "explorerViewletVisible && filesExplorerFocus"
  },
  {
    "key": "cmd+shift+n",
    "command": "explorer.newFolder",
    "when": "explorerViewletVisible && filesExplorerFocus"
  }
]
```
