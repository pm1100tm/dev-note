# Project Setting

React 개발 환경 세팅

## 🚀 기본 환경 세팅

### 프로젝트 폴더 생성

```shell
mkdir [proeject name]
cd [proeject name]
```

### npm global 설치

```shell
npm install -g npm
```

### node version 세팅

```shell
node --version > .node-version
```

### npm 초기화

```shell
npm init -y
```

---

## 🚀 Typescript 설치 및 설정

```shell
npm i -D typescript
```

Typescript 설정 파일 생성

```shell
npx tsc --init
```

tsconfig.json 파일에서 jsx 항목 수정

```javascript
{
  "jsx": "react-jsx"
}
```

---

## 🚀 ESLint 설치 및 설정

```shell
npx eslint --init

$ > To check syntax, find problems, and enforce code style
$ > JavaScript modules (import/export)
$ > React
$ > Yes(TypeScript)
$ > Browser
$ > Use a popular style guide
$ > XO
$ > JavaScript
$ > Yes
$ > npm
```

### Airbnb 스타일 적용하기(optional)

TypeScript에서 Airbnb 지원하지 않기 때문에, 기존에 설치한 XO를 삭제하고 Airbnb 스타일을 적용한다.

```shell
npm uninstall eslint-config-xo
npm uninstall eslint-config-xo-typescript

npm i -D eslint-config-airbnb
npm i -D eslint-plugin-import
npm i -D eslint-plugin-react
npm i -D eslint-plugin-react-hooks
npm i -D eslint-plugin-jsx-a11y

# install by one line
npm i -D eslint-config-airbnb eslint-plugin-import eslint-plugin-react eslint-plugin-react-hooks eslint-plugin-jsx-a11y
```

### .eslintrc.js 파일 수정

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    jest: true, // jest 설정은 미리 해둠
  },
  extends: [
    'airbnb',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
  ],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: ['react', 'react-hooks', '@typescript-eslint'],
  settings: {
    'import/resolver': {
      node: {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
      },
    },
  },
  rules: {
    indent: ['error', 2],
    'no-trailing-spaces': 'error',
    curly: 'error',
    'brace-style': 'error',
    'no-multi-spaces': 'error',
    'space-infix-ops': 'error',
    'space-unary-ops': 'error',
    'no-whitespace-before-property': 'error',
    'func-call-spacing': 'error',
    'space-before-blocks': 'error',
    'keyword-spacing': ['error', { before: true, after: true }],
    'comma-spacing': ['error', { before: false, after: true }],
    'comma-style': ['error', 'last'],
    'comma-dangle': ['error', 'always-multiline'],
    'space-in-parens': ['error', 'never'],
    'block-spacing': 'error',
    'array-bracket-spacing': ['error', 'never'],
    'object-curly-spacing': ['error', 'always'],
    'key-spacing': ['error', { mode: 'strict' }],
    'arrow-spacing': ['error', { before: true, after: true }],
    'import/no-extraneous-dependencies': [
      'error',
      {
        devDependencies: [
          '**/*.test.js',
          '**/*.test.jsx',
          '**/*.test.ts',
          '**/*.test.tsx',
        ],
      },
    ],
    'import/extensions': [
      'error',
      'ignorePackages',
      {
        js: 'never',
        jsx: 'never',
        ts: 'never',
        tsx: 'never',
      },
    ],
    'react/jsx-filename-extension': [
      2,
      {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
      },
    ],
    'jsx-a11y/label-has-associated-control': ['error', { assert: 'either' }],
  },
};
```

### prettier 충돌 방지(optional)

```shell
npm install --save-dev eslint-config-prettier
```

.eslintrc.js 파일 수정

```javascript
extends: [
  'airbnb',
  'plugin:@typescript-eslint/recommended',
  'plugin:react/recommended',
  'plugin:react/jsx-runtime',
  'prettier',
],
```

### .eslintignore 파일 생성

```plain
/node_modules/
/dist/
/.parcel-cache/
```

---

## 🚀 React 설치

```shell
npm i react react-dom
npm i -D @types/react @types/react-dom
```

### src 폴더 생성

```shell
mkdir src
```

### App.tsx 파일 생성: ./src/App.tsx

```tsx
export default function App() {
  return <div>Hello, world!</div>;
}
```

### main.tsx 파일 생성: ./src/main.tsx

```tsx
import ReactDOM from 'react-dom/client';

import App from './App';

function main() {
  const element = document.getElementById('root');

  if (!element) {
    return;
  }

  const root = ReactDOM.createRoot(element);

  root.render(<App />);
}

main();
```

### index.html 파일 생성: ./index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="./src/main.tsx"></script>
  </body>
</html>
```

---

## 🚀 Parcel 설치 및 설정

```shell
npm i -D parcel
```

### parcel-reporter-static-files-copy 설치

```shell
npm install -D parcel-reporter-static-files-copy
```

### .parcelrc 생성: ./.parcelrc

```javascript
{
  "extends": ["@parcel/config-default"],
  "reporters":  ["...", "parcel-reporter-static-files-copy"]
}
```

### static folder 생성: ./static

- 프로젝트 root path 에 static 폴더 생성
- 정적 이미지 등 저장 폴더

```shell
mkdir static
```

---

### package.json 수정

main 삭제 후 아래의 코드 추가

```json
"source": "./index.html",
```

scripts 속성 수정(\*jest는 추후 설치)

```json
"scripts": {
  "start": "parcel --port 8080",
  "build": "parcel build",
  "check": "tsc --noEmit",
  "lint": "eslint --fix --ext .js,.jsx,.ts,.tsx .",
  "test": "jest",
  "coverage": "jest --coverage --coverage-reporters html",
  "watch:test": "jest --watchAll"
},
```

---

## 🚀 프로젝트 구동 테스트

```shell
# 로컬 실행
npm run start

# 빌드
npx parcel build

# 정적 서버 실행
npx servor ./dist
```

---

## 🚀 Jest 와 React Testing Library 설치

```shell
npm i -D jest @types/jest
npm i -D @swc/core @swc/jest jest-environment-jsdom
npm i -D @testing-library/react @testing-library/jest-dom@5.16.4
```

### jest.config.js: ./jest.config.js

```js
module.exports = {
  // 테스트 환경 설정
  testEnvironment: 'jsdom',

  // Jest를 실행하기 전 설정 파일
  setupFilesAfterEnv: ['<rootDir>/src/jest.setup.js'],

  // 트랜스파일러 설정
  transform: {
    '^.+\\.(t|j)sx?$': [
      '@swc/jest',
      {
        jsc: {
          parser: {
            syntax: 'typescript',
            jsx: true,
            decorators: true,
          },
          transform: {
            react: {
              runtime: 'automatic',
            },
            legacyDecorator: true,
            decoratorMetadata: true,
          },
        },
      },
    ],
  },
};
```

### jest.setup.js 파일 생성: ./src/jest.setup.js

- 일단 생성만 해두고, 나중에 필요할 때 코드 추가

### eslintrc.js 파일 수정

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    jest: true, // 추가(이전 설정에서 미리 해두었지만 한 번 더 확인)
  },
  ...
}
```

### 테스트 구동 테스트

```shell
npm run test

# 아래와 같이 현재 만들어진 테스트 파일이 없다고 나오면 완료
No tests found, exiting with code 1
Run with `--passWithNoTests` to exit with code 0
In /.../workspace/react-project
  9 files checked.
  testMatch: **/__tests__/**/*.[jt]s?(x), **/?(*.)+(spec|test).[tj]s?(x) - 0 matches
  testPathIgnorePatterns: /node_modules/ - 9 matches
  testRegex:  - 0 matches
Pattern:  - 0 matches
```
