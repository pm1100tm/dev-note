# Webpack 이란

Webpack 은 프론트엔드 애플리케이션을 위한 모듈 번들러이다.

쉽게 말하면, React 같은 프론트엔드 프로젝트에서 사용하는 여러 파일들을 분석하여 브라우저가 이해할 수 있는
현태로 묶어주는 도구이다.

```
src/
  index.tsx
  App.tsx
  Button.tsx
  style.css
  logo.png
```

개발자는 import를 사용해서 여러 파일을 나누어 작성한다.

```ts
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './style.css'

ReactDOM.createRoot(document.getElementById('root')!).render(<App />)
```

하지만 브라우저가 이 모든 파일을 그대로 효율적으로 처리하는 것은 어렵다. Webpack은 이 파일들의 의존
관계를 분석해서 최종적으로 브라우저에 전달할 파일을 만들어준다.

공식 문서에서도 Webpack의 핵심 개념을 Entry, Output, Loaders, Plugins, Mode로 설명한다.

## Webpack이 필요한 이유

프론트엔드 개발에서는 JavaScript만 작성하지 않는다. 보통 다음과 같은 파일들을 함께 사용한다.

```
JavaScript / TypeScript
React JSX / TSX
CSS / SCSS
이미지
폰트
JSON
환경 변수
```

문제는 브라우저가 이 모든 파일을 개발자가 작성한 형태 그대로 이해하지는 못한다는 점이다.
예를 들어 TypeScript는 브라우저가 직접 실행할 수 없다.

```ts
const message: string = 'Hello Webpack';
```

브라우저에서 실행하려면 JavaScript로 변환되어야 한다.

```js
const message = 'Hello Webpack';
```

Webpack은 이런 파일들을 처리해서 최종 결과물을 만든다.

```
개발자가 작성한 코드
        ↓
Webpack이 의존성 분석
        ↓
Loader / Plugin 처리
        ↓
브라우저가 실행할 수 있는 정적 파일 생성
```

## Webpack의 핵심 역할

Webpack의 핵심 역할은 크게 4가지로 볼 수 있다.

| 역할        | 설명                                                                          |
| ----------- | ----------------------------------------------------------------------------- |
| 모듈 번들링 | 여러 JavaScript 모듈을 하나 또는 여러 개의 파일로 묶는다                      |
| 의존성 분석 | `import`, `require` 등을 따라가며 필요한 파일을 찾는다                        |
| 파일 변환   | TypeScript, JSX, CSS, 이미지 등을 처리한다                                    |
| 최적화      | 코드 분할(Code Splitting), 압축(Minification), 캐싱 등을 통해 성능을 개선한다 |

Webpack은 JavaScript뿐 아니라 CSS의 @import, 이미지 URL, HTML 이미지 경로 등 다양한 형태의 의존성도 모듈로 다룰 수 있다.

## Webpack의 동작 흐름

Webpack은 보통 다음 순서로 동작한다.

```
1. Entry 파일에서 시작한다
2. import 된 파일들을 따라가며 의존성 그래프를 만든다
3. Loader를 통해 파일을 변환한다
4. Plugin을 통해 추가 작업을 수행한다
5. Output 위치에 번들 파일을 생성한다
```

```ts
import App from './App';

import './global.css';
```

그리고 App.tsx, global.css, 그 안에서 import한 다른 파일들까지 계속 따라간다.

이렇게 만들어지는 관계도를 Dependency Graph, 즉 의존성 그래프라고 한다. 공식 문서에서도
Entry는 Webpack이 내부 의존성 그래프를 만들기 시작하는 지점이라고 설명한다.

<br>

## Webpack 핵심 개념

### 1. Entry

entry는 Webpack이 번들링을 시작하는 진입점이다.

```js
// webpack.config.js
module.exports = {
	entry: './src/index.js',
};
```

React 프로젝트라면 보통 index.tsx, main.tsx 같은 파일이 Entry가 된다.

```ts
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'

ReactDOM.createRoot(document.getElementById('root')!).render(<App />)
```

이 파일이 애플리케이션의 시작점 역할을 한다.

### 2. Output

output은 번들링된 결과물이 어디에 어떤 이름으로 생성될지를 정한다.

```js
const path = require('path');

module.exports = {
	entry: './src/index.js',
	output: {
		filename: 'bundle.js',
		path: path.resolve(__dirname, 'dist'),
	},
};
```

위 설정은 다음과 같은 결과물을 만든다.

```
dist/
  bundle.js
```

공식 문서에서도 output은 Webpack이 생성한 번들, 에셋 등을 어디에 어떤 방식으로 내보낼지 지시하는
설정이라고 설명한다.

### 3. Loader

Loader는 Webpack이 JavaScript가 아닌 파일을 이해할 수 있도록 변환해주는 도구이다.

Webpack은 기본적으로 JavaScript와 JSON을 이해한다. 하지만 TypeScript, JSX, CSS, SCSS,
이미지 파일 등을 처리하려면 Loader가 필요하다.

공식 문서에 따르면 Loader는 모듈의 소스 코드에 적용되는 변환 작업이며, 파일을 import하거나 load
할 때 전처리할 수 있게 해준다.

예를 들어 TypeScript를 처리하려면 ts-loader나 babel-loader를 사용할 수 있다.

```js
module.exports = {
	module: {
		rules: [
			{
				test: /\.tsx?$/,
				use: 'ts-loader',
				exclude: /node_modules/,
			},
		],
	},
};
```

CSS를 처리하는 예시는 다음과 같다.

```js
module.exports = {
	module: {
		rules: [
			{
				test: /\.css$/,
				use: ['style-loader', 'css-loader'],
			},
		],
	},
};
```

여기서 각각의 역할은 다음과 같다.

| Loader       | 역할                                                       |
| ------------ | ---------------------------------------------------------- |
| css-loader   | CSS 파일을 JavaScript에서 `import`할 수 있게 처리          |
| style-loader | 처리된 CSS를 브라우저의 `<style>` 태그로 삽입              |
| ts-loader    | TypeScript를 JavaScript로 변환                             |
| babel-loader | 최신 JavaScript 문법을 구형 브라우저에서도 동작하도록 변환 |

### 4. Plugin

Plugin은 Loader보다 더 넓은 범위의 작업을 처리한다.

Loader가 “파일 단위 변환”에 가깝다면, Plugin은 “번들링 전체 과정에 개입하는 확장 기능”에 가깝다.

예를 들어 다음과 같은 일을 할 수 있다.

공식 문서에서도 Plugin은 번들 최적화, 에셋 관리, 환경 변수 주입 등 더 광범위한 작업을 수행할 수 있다고 설명한다.

| Plugin 역할    | 예시                                                       |
| -------------- | ---------------------------------------------------------- |
| HTML 파일 생성 | 번들 파일이 자동 삽입된 `index.html` 생성                  |
| 번들 파일 정리 | 빌드 전에 `dist` 폴더 비우기                               |
| 환경 변수 주입 | `process.env.NODE_ENV` 같은 값 주입                        |
| 번들 최적화    | 압축(Minification), 난독화, 코드 분할(Code Splitting) 보조 |

대표적인 Plugin 예시는 html-webpack-plugin이다.

```js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
	plugins: [
		new HtmlWebpackPlugin({
			template: './public/index.html',
		}),
	],
};
```

이 설정은 Webpack이 생성한 번들 파일을 HTML에 자동으로 연결해준다.

### 5. Mode

mode는 Webpack에게 현재 빌드가 개발용인지 운영용인지 알려주는 설정이다.

```js
module.exports = {
	mode: 'development',
};
```

| Mode        | 특징                                  |
| ----------- | ------------------------------------- |
| development | 빠른 빌드, 디버깅 편의성, 소스맵 중심 |
| production  | 코드 압축, 최적화, 파일 크기 감소     |
| none        | 기본 최적화 비활성화                  |

개발 환경에서는 디버깅이 중요하고, 운영 환경에서는 성능과 파일 크기가 중요하다.

### 간단한 Webpack 설정 예시

아래는 React + TypeScript 프로젝트에서 사용할 수 있는 기본적인 Webpack 설정 예시이다.

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
	mode: 'development',

	entry: './src/index.tsx',

	output: {
		filename: 'bundle.js',
		path: path.resolve(__dirname, 'dist'),
		clean: true,
	},

	resolve: {
		extensions: ['.tsx', '.ts', '.js'],
	},

	module: {
		rules: [
			{
				test: /\.tsx?$/,
				use: 'ts-loader',
				exclude: /node_modules/,
			},
			{
				test: /\.css$/,
				use: ['style-loader', 'css-loader'],
			},
		],
	},

	plugins: [
		new HtmlWebpackPlugin({
			template: './public/index.html',
		}),
	],

	devServer: {
		static: './dist',
		port: 3000,
		hot: true,
	},
};
```

<br>

## Webpack이 해결하는 문제

- 1. 여러 파일을 하나의 번들로 묶는다
- 2. 브라우저가 이해하지 못하는 문법을 변환한다
- 3. CSS와 이미지도 모듈처럼 관리할 수 있다
- 4. 코드 스플리팅으로 초기 로딩 성능을 개선한다
