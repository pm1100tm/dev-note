# Parcel 기반 React 개발 환경 설정

React, TypeScript, ESLint, Jest, Parcel을 사용해 학습용 프로젝트를 구성하는 절차를
정리한다. 새 프로젝트에는 Vite를 더 자주 사용하지만, 이 문서는 Parcel 기반 프로젝트를
유지하거나 학습할 때 참고한다.

## 준비 사항

- Node.js LTS 버전을 설치한다.
- 프로젝트별 Node.js 버전을 고정하려면 [FNM](react_03_fnm.md)을 사용한다.
- 아래 명령은 npm을 기준으로 설명한다.

## 1. 프로젝트 생성과 npm 초기화

```shell
mkdir react-parcel-app
cd react-parcel-app
npm init -y
```

프로젝트 루트에 `.node-version` 파일을 만들면 FNM이 프로젝트에 맞는 Node.js 버전을
선택할 수 있다.

```shell
node --version > .node-version
```

## 2. React와 TypeScript 설치

런타임에 필요한 React 패키지와 개발 도구인 TypeScript를 구분해 설치한다.

```shell
npm install react react-dom
npm install --save-dev typescript @types/react @types/react-dom
npx tsc --init
```

`tsconfig.json`에서 JSX 변환 방식을 React의 자동 런타임에 맞춘다.

```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

## 3. Parcel 설치와 실행 스크립트 추가

Parcel은 개발 서버와 번들 생성을 담당하는 번들러다. 프로젝트에 개발 의존성으로 설치한다.

```shell
npm install --save-dev parcel
```

`package.json`에 개발, 빌드, 타입 검사 스크립트를 추가한다.

```json
{
  "source": "index.html",
  "scripts": {
    "start": "parcel --port 8080",
    "build": "parcel build",
    "check": "tsc --noEmit"
  }
}
```

## 4. ESLint 설정

ESLint는 문법 오류와 일관되지 않은 코드 스타일을 조기에 찾는 도구다. 설치 후 대화형
설정을 실행하고 React, TypeScript, Browser 환경을 선택한다.

```shell
npm install --save-dev eslint
npx eslint --init
```

생성된 설정 파일 형식에 맞춰 lint 스크립트를 추가한다.

```json
{
  "scripts": {
    "lint": "eslint --ext .js,.jsx,.ts,.tsx ."
  }
}
```

Prettier를 함께 쓴다면 충돌하는 ESLint 규칙을 끄는 설정을 추가한다.

```shell
npm install --save-dev eslint-config-prettier
```

`.eslintignore`에는 번들 결과물과 의존성 디렉터리를 제외한다.

```text
/node_modules/
/dist/
/.parcel-cache/
```

## 5. Jest와 Testing Library 설치

Jest는 테스트 실행기이고, Testing Library는 사용자 관점에서 React 화면을 테스트하는
도구다.

```shell
npm install --save-dev jest @types/jest @swc/core @swc/jest \
  jest-environment-jsdom @testing-library/react @testing-library/jest-dom
```

`package.json`에 테스트 스크립트를 추가한다.

```json
{
  "scripts": {
    "test": "jest",
    "coverage": "jest --coverage --coverageReporters html"
  }
}
```

## 6. 진입 파일 만들기

Parcel이 읽을 `index.html`과 React 진입점 `src/main.tsx`를 만든다.

```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>React Demo App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="./src/main.tsx"></script>
  </body>
</html>
```

```tsx
import ReactDOM from 'react-dom/client';

import App from './App';

const element = document.getElementById('root');

if (!element) {
  throw new Error('root 요소를 찾을 수 없습니다.');
}

ReactDOM.createRoot(element).render(<App />);
```

```tsx
export default function App() {
  return <p>Hello, world!</p>;
}
```

## 7. 개발 서버와 빌드 확인

```shell
npm run start
npm run check
npm run lint
npm run build
```

- 개발 서버는 기본적으로 `http://localhost:8080`에서 확인한다.
- `dist`는 빌드 결과물이며 일반적으로 Git에 포함하지 않는다.
- `.parcel-cache`는 Parcel 캐시이므로 Git에 포함하지 않는다.
