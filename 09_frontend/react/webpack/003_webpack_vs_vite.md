# Webpack과 Vite의 차이

최근에는 Webpack 대신 Vite를 사용하는 프로젝트도 많다.

| 항목           | Webpack                             | Vite                             |
| -------------- | ----------------------------------- | -------------------------------- |
| 개발 서버 방식 | 전체 번들링 기반                    | ESM 기반 즉시 로딩               |
| 초기 실행 속도 | 프로젝트가 커질수록 느려질 수 있음  | 빠른 편                          |
| 설정 자유도    | 매우 높음                           | 비교적 단순                      |
| 생태계         | 오래되고 안정적                     | 현대 프론트엔드에 최적화         |
| 사용 사례      | 복잡한 커스텀 빌드, 레거시 프로젝트 | React, Vue, Svelte 신규 프로젝트 |

Webpack은 설정이 복잡할 수 있지만, 커스터마이징 능력이 강하다.
반면 Vite는 개발 서버 속도가 빠르고 설정이 간단해서 신규 프로젝트에서 자주 사용된다.

## ESM(ECMAScript Modules)

- JavaScript의 공식 모듈 시스템
- 과거에는 JavaScript에 공식 모듈 시스템이 없어서 CommonJS(require)나 AMD 같은 방식이
  사용되었지만, ES6(ES2015)부터 import와 export가 표준으로 추가되면서 ESM이 등장했다.

<br>

## 프론트엔드 개발자가 Webpack을 알아야 하는 이유

요즘은 Next.js, CRA, Vite 같은 도구들이 내부 빌드 설정을 숨겨준다. 그래서 Webpack 설정을 직접
작성하지 않아도 프로젝트를 시작할 수 있다.

하지만 프론트엔드 개발자가 Webpack을 이해하면 다음 문제를 더 잘 해결할 수 있다.

| 상황                  | Webpack 이해가 필요한 이유                                      |
| --------------------- | --------------------------------------------------------------- |
| 빌드 에러 발생        | Loader, Plugin, Module Resolution 문제를 추적할 수 있음         |
| 번들 크기 증가        | Code Splitting, Tree Shaking, SplitChunks 전략을 이해할 수 있음 |
| 환경별 설정 필요      | 개발/운영 설정을 분리할 수 있음                                 |
| 이미지/폰트 로딩 문제 | Asset 처리 방식을 이해할 수 있음                                |
| 성능 최적화           | 초기 로딩 속도, 캐싱 전략을 개선할 수 있음                      |

단순히 “Webpack 설정 파일을 외운다”가 중요한 게 아니다.
중요한 것은 프론트엔드 코드가 어떻게 브라우저에서 실행 가능한 결과물로 변환되는지 이해하는 것이다.

<br >

## Webpack을 한 문장으로 정리하면

Webpack은 프론트엔드 프로젝트의 JavaScript, TypeScript, CSS, 이미지 같은 여러 리소스의
의존성을 분석하고, 브라우저가 실행할 수 있는 최종 정적 파일로 묶어주는 모듈 번들러이다.

```
# Webpack이란 무엇인가?

## 1. Webpack을 이해하기 전에 알아야 할 것
- 브라우저는 어떤 파일을 실행하는가
- 프론트엔드 프로젝트는 왜 여러 파일로 나뉘는가

## 2. Webpack의 정의
- 모듈 번들러
- 의존성 그래프
- 정적 파일 생성

## 3. Webpack이 필요한 이유
- TypeScript, JSX 변환
- CSS, 이미지 처리
- 번들 최적화

## 4. Webpack의 핵심 개념
- Entry
- Output
- Loader
- Plugin
- Mode

## 5. Webpack 동작 흐름
- Entry에서 시작
- Dependency Graph 생성
- Loader 처리
- Plugin 처리
- Output 생성

## 6. 간단한 설정 예제
- webpack.config.js
- React + TypeScript 예시

## 7. Webpack과 Babel의 차이
## 8. Webpack과 Vite의 차이
## 9. 프론트엔드 개발자가 Webpack을 알아야 하는 이유
## 10. 결론
```
