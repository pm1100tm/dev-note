# vscode 단축키 - 파일/폴더 생성

- ⌘⇧P → Preferences: Open Keyboard Shortcuts (JSON) 실행
- 기존 배열 안에 아래 항목 추가

```json
[
  {
    "key": "cmd+n",
    "command": "explorer.newFile",
    "when": "sideBarFocus && activeViewlet == 'workbench.view.explorer' && !textInputFocus"
  },
  {
    "key": "cmd+shift+n",
    "command": "explorer.newFolder",
    "when": "sideBarFocus && activeViewlet == 'workbench.view.explorer' && !textInputFocus"
  }
]
```

## 현재까지의 설정

```json
// Place your key bindings in this file to override the defaults
[
  {
    "key": "cmd+d",
    "command": "editor.action.deleteLines",
    "when": "textInputFocus && !editorReadonly"
  },
  {
    "key": "shift+cmd+k",
    "command": "-editor.action.deleteLines",
    "when": "textInputFocus && !editorReadonly"
  },
  {
    "key": "cmd+n",
    "command": "explorer.newFile",
    "when": "sideBarFocus && activeViewlet == 'workbench.view.explorer' && !textInputFocus"
  },
  {
    "key": "cmd+shift+n",
    "command": "explorer.newFolder",
    "when": "sideBarFocus && activeViewlet == 'workbench.view.explorer' && !textInputFocus"
  }
]
```
