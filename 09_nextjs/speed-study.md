# Next.js + React 핵심 개념 정리

---

# 1. 렌더링 방식 (Rendering)

## CSR (Client Side Rendering)

- 브라우저에서 JS 실행 → 화면 생성
- 초기 로딩 느림, 이후 빠름
- SEO 불리

---

## SSR (Server Side Rendering)

- 요청마다 서버에서 HTML 생성
- 초기 로딩 빠름
- SEO 유리

---

## SSG (Static Site Generation)

- 빌드 시 HTML 생성
- 가장 빠른 응답
- 데이터는 빌드 시점 기준

---

## ISR (Incremental Static Regeneration)

- SSG + 일정 시간마다 재생성

```ts
export const revalidate = 60; // 60초마다 재생성
```

---

## Streaming

- HTML을 한 번에 보내지 않고 “조각 단위로” 전송
- 빠른 초기 렌더링 가능

### Suspense

- 비동기 로딩 상태를 처리하는 React 기능

```ts
<Suspense fallback={<div>Loading...</div>}>
  <Component />
</Suspense>
```

### PPR (Partial Prerendering)

- 일부는 SSG, 일부는 SSR
- 정적 + 동적 혼합 렌더링

## 2. 성능 지표

## 3. SEO 관련

OpenGraph

- SNS 공유 시 미리보기 정보

JSON-LD

- 구조화된 데이터 (SEO 강화)

```html
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Product",
    "name": "상품명"
  }
</script>
```

## 4. App Router

- 폴더 구조 = URL

```
app/
  products/
    page.tsx   → /products
```

## 5. Hydration (하이드레이션)

정의

- 서버에서 만든 HTML에 React를 붙여서 이벤트와 상태를 복원하는 과정

왜 필요한가?

- SSR / SSG HTML은 정적이다
  - 상태 없음
  - 클릭 안됨
  - React 연결 없음

```ts
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(container, <App />);
```

특징

- 1. 기존 HTML 재사용
  - 다시 렌더링이 아니라 “붙이는 것”
- 2. 서버와 클라이언트 결과 동일해야 함
  - 다르면 에러 발생
  - Hydration failed because the initial UI does not match

## 6. TanStack Query

역할

- 서버 상태 관리

특징

- 캐싱
- 자동 refetch
- 비동기 상태 관리

사용 예시

```ts
import { useQuery } from '@tanstack/react-query';

const { data, isLoading } = useQuery({
  queryKey: ['products'],
  queryFn: fetchProducts,
});
```

Mutation

```ts
import { useMutation } from '@tanstack/react-query';

const mutation = useMutation({
  mutationFn: createProduct,
});
```

## 7. Zustand (클라이언트 상태 관리)

정의

- 브라우저에서만 존재하는 상태 관리

예시

- UI 상태
- 테마
- 폼 입력 상태
- 장바구니 (비로그인)

구조

```
store/
  cart/
    store.ts
    types.ts
    actions.ts
  auth/
    store.ts
    types.ts
  ui/
    modal.ts
    toast.ts
    sidebar.ts
```

사용

```ts
import { create } from 'zustand';

export const useStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
}));
```

## 8. TanStack Query vs Zustand

- TanStack Query: 서버 상태
- Zustand: 클라이언트 상태

## 9. React Hook Form + Zod

문제점 (기본 React Form)

- 필드마다 useState 필요
- 전체 리렌더링 발생
- 유효성 검사 직접 구현
- 타입 안전성 부족

해결

- React Hook Form
- 리렌더링 최소화
- 폼 상태 관리

Zod

- 스키마 기반 검증
- 타입 자동 추론

설치

```shell
npm install zod react-hook-form @hookform/resolvers
```

예시

```ts
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export default function Form() {
  const { register, handleSubmit } = useForm({
    resolver: zodResolver(schema),
  });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input {...register('email')} />
      <input {...register('password')} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

기본 스키마

```ts
import { z } from 'zod';

// 로그인 스키마
export const loginSchema = z.object({
  email: z
    .string()
    .min(1, '이메일을 입력하세요')
    .email('올바른 이메일 형식이 아닙니다'),
  password: z
    .string()
    .min(1, '비밀번호를 입력하세요')
    .max(8, '비밀번호는 8자 이상이어야 합니다'),
});

export type LoginFormData = z.infer<typeof loginSchema>;
```

회원 가입 스키마

```ts
export const signupSchema = z
  .object({
    email: z.string().email('올바른 이메일을 입력하세요'),
    password: z
      .string()
      .min(8, '8자 이상')
      .regex(/[A-Z]/, '대문자 포함')
      .regex(/[0-9]/, '숫자 포함')
      .password(),
    confirmPassword: z.string(),
    name: z.string().min(2, '이름은 2자 이상'),
    phone: z.string().regex(/^010-\d{4}-\d{4}$/, '010-0000-0000 형식'),
    aggreeTerms: z.literal(true, {
      errorMap: () => ({ message: '약관에 동의해주세요' }),
    }),
  })
  .refine((data) => data.paswword === data.confirmPassword, {
    message: '비밀번호가 일치하지 않습니다',
    path: ['confirmPassword'],
  });

export type signupFormData = z.infer<typeof signupSchema>;
```

주요 Zod 메서드

- min(), max(): 길이/범위 제한
- email(): 이메일 형식
- regex(): 정규식 패턴
- refine(): 커스텀 검증
- optional(): 선택적 필드

### React Hook Form 기초

- React Hook Form 과 Zod를 연결하여 안전한 폼을 만듦
- zodResolver를 사용하면 Zod 스키마로 자동 유효성 검사가 됨

#### 기본 사용법

```ts
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { loginSchema, type LoginFormData } from '@/schemas/auth';

export function LoginForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = async (data: LoginFormData) => {
    // data 는 타입 안전함
    console.log(data.email, data.password);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input {...register('email')} placeholder='이메일' />
        { errros.email && <p>{errors.email.message}</p> }
      <div>

      <div>
        <input {...register('password')} type="password" />
        { errors.password && <p>{errors.password.message}</p> }
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '로그인 중...' : '로그인'}
      </button>
    </form>
  )
}
```

- register: 입력 필드를 폼에 등록
- handleSubmit: 폼 제출 핸들러 래퍼
- formState.errors: 필드별 에러 메세지
- formState.isSubmitting: 제출 중 상태
- watch: 필드 값 실시간 감시
- reset: 폼 초기화

> 성능 이점

- 비제어 컴포넌트로 리렌더링 최소화
- 대규모 폼에서 큰 성능 차이
- 이커머스 주문 폼에 적합

## shadcn/ui Form 컴포넌트

- shadcd/ui 의 Form 컴포넌트로 일관된 스타일의 폼을 만듦
- React Hook Form 과 완벽하게 통합되어 있어, 사용하기 편리

---

## FSD(Feature-Sliced Design) 아키텍쳐

- 프론트엔드 프로젝트를 구조화하는 아키텍처 방법론
- 코드를 기능(Feature) 단위로 분리하여 유지보수와 확장성 높임

### 기존 구조의 문제점

- 대부분의 프로젝트는 기술 유형별로 폴더를 구성합니다. components, hooks, utils 등으로
  나누는 방식
- 프로젝트가 작을 때는 문제없지만, 규모가 커지면 심각한 문제가 발생

```
// ❌ 일반적인 폴더 구조 (기술 유형별)
src/
├── components/           # 모든 컴포넌트가 섞여있음
│   ├── Button.tsx
│   ├── ProductCard.tsx
│   ├── CartItem.tsx
│   ├── OrderSummary.tsx
│   ├── UserProfile.tsx
│   ├── Header.tsx
│   └── ... (100개 이상의 컴포넌트)
├── hooks/                # 모든 훅이 섞여있음
│   ├── useAuth.ts
│   ├── useCart.ts
│   ├── useProduct.ts
│   └── useOrder.ts
├── utils/                # 모든 유틸이 섞여있음
├── services/             # 모든 API 호출이 섞여있음
└── pages/

// 이 구조의 문제점:
// 1. ProductCard가 어디서 사용되는지 파악하려면 전체 검색 필요
// 2. 장바구니 기능 수정 시 여러 폴더를 돌아다녀야 함
// 3. 팀원 A가 components/를 수정하면 팀원 B와 충돌
// 4. 어떤 코드가 어떤 기능에 속하는지 불명확
```

#### 👉 실제 발생하는 문제들

```
• "이 컴포넌트 삭제해도 되나요?" - 영향 범위 파악 불가
• "장바구니 관련 코드가 어디있죠?" - 5개 폴더에 분산
• "이 유틸 함수 누가 쓰고 있어요?" - 전체 검색 필요
• "새 기능 추가할 때 어디에 만들어야 하죠?" - 기준 없음
```

### 레이어별 책임 요약

| 레이어   | 책임            | 예시                   | import 가능     |
| -------- | --------------- | ---------------------- | --------------- |
| app      | 앱 초기화       | Providers, 전역 스타일 | 모든 레이어     |
| pages    | 페이지 조합     | ProductePage           | widgets 이하    |
| widgets  | UI 블록         | Header, ProductList    | features 이하   |
| features | 비지니스 기능   | AddToCart, Login       | entities 이하   |
| entities | 비지니스 엔티티 | Product, User          | shared 이하     |
| shared   | 공유 코드       | Button, formatPrice    | 외부 라이브러리 |

### Next.js App Router 와 FSD

```
// Next.js App Router + FSD 구조
project/
├── app/                    # Next.js App Router (pages 역할)
│   ├── layout.tsx          # 루트 레이아웃 (app 레이어)
│   ├── providers.tsx       # 프로바이더 (app 레이어)
│   ├── page.tsx            # 홈페이지
│   └── products/
│       └── [id]/
│           └── page.tsx    # 상품 상세 페이지
├── src/
│   ├── widgets/            # 위젯 레이어
│   ├── features/           # 기능 레이어
│   ├── entities/           # 엔티티 레이어
│   └── shared/             # 공유 레이어
└── ...

// Next.js의 app/ 디렉토리가 FSD의 app + pages 역할을 함
// 나머지 레이어는 src/ 아래에 구성
```

레이어 선택 기준

- shared: 비지니스 로직 없이 어디서든 재사용
- entities: 비지니스 데이터 모델과 기본 표현
- features: 사용자 액션, 비지니스 로직 포함
- widgets: 여러 features/entitis 조합

### 슬라이스와 세그먼트

- FSD에서 레이어 내부는 슬라이스(Slice)로 나뉘고, 각 슬라이스는 세그먼트(Segment)로 구성

```
// 레이어 내 슬라이스 구조
features/                   # 레이어
├── cart/                   # 슬라이스: 장바구니 기능
│   ├── add-to-cart/        # 하위 슬라이스
│   ├── remove-from-cart/
│   └── update-quantity/
├── auth/                   # 슬라이스: 인증 기능
│   ├── login/
│   ├── logout/
│   └── register/
├── checkout/               # 슬라이스: 결제 기능
│   ├── payment/
│   └── shipping/
└── wishlist/               # 슬라이스: 위시리스트 기능

entities/                   # 레이어
├── product/                # 슬라이스: 상품 엔티티
├── user/                   # 슬라이스: 사용자 엔티티
├── order/                  # 슬라이스: 주문 엔티티
└── category/               # 슬라이스: 카테고리 엔티티
```

### 세그먼트 (Segment)

- 세그먼트는 슬라이스 내에서 코드의 목적별로 분리하는 단위
- 표준 세그먼트로는,
  - ui, model, api, lib, config 가 있음

```
// 슬라이스 내 세그먼트 구조
features/cart/add-to-cart/
├── ui/                     # UI 컴포넌트
│   ├── AddToCartButton.tsx
│   └── QuantitySelector.tsx
├── model/                  # 비즈니스 로직, 상태
│   ├── store.ts            # Zustand 스토어
│   ├── types.ts            # 타입 정의
│   └── hooks.ts            # 커스텀 훅
├── api/                    # API 호출
│   └── addToCart.ts
├── lib/                    # 유틸리티 함수
│   └── calculateTotal.ts
└── index.ts                # Public API
```
