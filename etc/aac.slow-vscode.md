# slow vscode

## VSCODE 가 느려질 때

Visual Studio Code로 프로젝트를 진행하다가, 파일이 많아진 것이 문제인지 타이핑 속도가 현저히
느려졌다. 파일 검색은 물론, 파일을 띄우는 전체적인 IDE내의 동작들이 2박자 정도 느리게 반응했다.
검색을 해보니 IDE내의 설정을 통하여 해당 현상을 완화할 수 있었다.

```shell
# vscode 에서 아래의 단축키 입력
cmd + shift + p

# 아래 검색어 입력 후 엔터
> config runtime arguments
```

아래 코드 추가

```json
"disable-hardware-acceleration": true
```

해당 파라미터 값의 적용은 **GPU 가속 비활성화** 설정을 적용시키는 것이라고 하며, 느려지는 현상이
발생하지 않는다면 설정하지 말것을 권장한다.

## ☕️ Ref

- [How can I disable GPU rendering in Visual Studio Code](https://stackoverflow.com/questions/29966747/how-can-i-disable-gpu-rendering-in-visual-studio-code)
