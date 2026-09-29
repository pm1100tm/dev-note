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
    "key": "cmd+shift+n",
    "command": "explorer.newFolder",
    "when": "explorerViewletFocus"
  },
  {
    "key": "cmd+n",
    "command": "explorer.newFile",
    "when": "explorerViewletFocus"
  }
]
```

## Editor Minimap 미표시 설정

미니맵 미표시 처리

```shell
cmd + shift + p
> Minimap
> Toggle Minimap
```

## Open User Settings (JSON) 설정

```shell
`Cmd + Shift + P` → `Preferences: Open User Settings (JSON)`
```

```json
{
  "js/ts.preferences.quoteStyle": "single",
  "editor.formatOnSave": true,
  "editor.unicodeHighlight.nonBasicASCII": false,
  "workbench.tree.indent": 14,
  "terminal.integrated.fontFamily": "'MesloLGS NF', monospace",
  "editor.accessibilitySupport": "on",
  "editor.stickyScroll.enabled": false,
  "workbench.editor.sharedViewState": false,
  "editor.minimap.enabled": false
}
```

- terminal.integrated.fontFamily
  - vscode 터미널 폰트 깨짐 현상 -> 폰트 설정
- workbench.editor.sharedViewState:
  - 탭 스플릿 후 스크롤 위치가 공유되는 현상 -> false
  - vscode 재시동 필요
