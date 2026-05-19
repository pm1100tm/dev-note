# React Suspence

Suspence는 아직 준비되지 않은 컴포넌트를 대신하여 임시 UI를 보여주는 React 의 기능입니다.

대표적으로 다음 상황에서 사용됩니다.

| 사용 상황          | 설명                                                               |
| ------------------ | ------------------------------------------------------------------ |
| 코드 분할          | React.lazy()로 컴포넌트를 늦게 불러올 때                           |
| 데이터 로딩        | Suspense를 지원하는 프레임워크나 라이브러리에서 데이터를 기다릴 때 |
| Next.js App Router | 서버 컴포넌트 로딩, Streaming, loading.tsx 처리                    |

- React 공식 문서 기준으로, Suspence는 모든 비동기 작업을 자동으로 감지하지 않음
- useEffect 안에서 직접 fetch한 데이터 로딩은 Suspence가 감지하지 못함
- Suspence를 지원하는 데이터 소스나 React.lazy, Next.js 같은 프레임워크와 함께 동작

## 1. 단계 기본 형태

```ts
import { Suspense } from 'react'

function App() {
  return (
    <Suspense fallback={<div>로딩 중...</div>}>
      <SomeComponent />
    </Suspense>
  )
}
```

```shell
<Suspense fallback={대기 중에 보여줄 UI}>
  <준비되면 보여줄 컴포넌트 />
</Suspense>
```

즉, SomeComponent가 바로 렌더링될 수 있으면 그대로 보여주고, 아직 준비되지 않았으면 fallback을 보여줍니다.

## 2단계. Suspense가 하는 일

React 입장에서 Suspense는 이런 흐름으로 동작합니다.

```shell
1. SomeComponent 렌더링 시도
2. SomeComponent가 아직 준비되지 않음
3. React가 가장 가까운 Suspense를 찾음
4. Suspense의 fallback UI를 화면에 표시
5. 준비가 끝나면 fallback 제거
6. 실제 SomeComponent 표시
```

## 3단계. React.lazy와 함께 사용하는 경우

가장 흔한 예시는 컴포넌트를 필요한 시점에만 불러오는 것입니다.

```ts
import { lazy, Suspense } from 'react';

const ProductChart = lazy(() => import('./ProductChart'));

export default function ProductPage() {
  return (
    <div>
      <h1>상품 상세</h1>
      <Suspense fallback={<div>차트를 불러오는 중...</div>}>
        <ProductChart />
      </Suspense>
    </div>
  )
```

- ProductChart 컴포넌트를 처음부터 번들에 포함하지 않고, 필요할 때 별도 JavaScript 파일로 불러옵니다.

## 4단계. Suspense를 쓰는 이유

Suspense를 안 쓰는 경우

```ts
function ProductPage() {
  const [isLoading, setIsLoading] = useState(true)
  const [data, setData] = useState(null)

  useEffect(() => {
    fetch('/api/products')
      .then((res) => res.json())
      .then((result) => {
        setData(result)
        setIsLoading(false)
      })
  }, [])

  if (isLoading) {
    return <div>로딩 중...</div>
  }

  return <ProductList data={data} />
}
```

- 이 방식은 컴포넌트 안에서 직접 로딩 상태를 관리합니다.

<br>

Suspense를 쓰는 경우

```ts
<Suspense fallback={<ProductListSkeleton />}>
  <ProductList />
</Suspense>
```

- 로딩 UI를 컴포넌트 내부가 아니라 컴포넌트 바깥의 경계에서 관리합니다.
- 즉, Suspense의 핵심은 이것입니다.
  - 로딩 상태를 개별 컴포넌트가 직접 처리하지 않고, 상위 Suspense Boundary가 대신 처리한다.

## 5단계. Suspense Boundary란?

Suspense Boundary는 Suspense가 감싸고 있는 영역을 말합니다.

```ts
<Suspense fallback={<div>전체 로딩 중...</div>}>
  <Header />
  <ProductList />
  <RecommendList />
</Suspense>
```

- 이 경우 ProductList 하나만 늦어져도, Suspense 안에 있는 전체 영역이 fallback으로 바뀔 수 있습니다.
- 전체가 다음 UI로 대체될 수 있습니다.

## 6단계. 그래서 Suspense는 작게 나누는 게 좋다

좋지 않은 예시입니다.

```ts
<Header />

<Suspense fallback={<ProductListSkeleton />}>
  <ProductList />
</Suspense>

<Suspense fallback={<RecommendSkeleton />}>
  <RecommendList />
</Suspense>

<Footer />
```

- 이렇게 하면 느린 영역만 따로 로딩 UI를 보여줄 수 있습니다.

## 7단계. fallback에는 무엇을 넣는가?

보통 다음과 같은 UI를 넣습니다.

```ts
<Suspense fallback={<div>Loading...</div>}>
  <ProductList />
</Suspense>
```

- 실무에서는 단순 텍스트보다 Skeleton UI를 많이 씁니다.

```ts
function ProductListSkeleton() {
  return (
    <div className="space-y-3">
      <div className="h-6 w-1/3 rounded bg-gray-200" />
      <div className="h-20 rounded bg-gray-200" />
      <div className="h-20 rounded bg-gray-200" />
      <div className="h-20 rounded bg-gray-200" />
    </div>
  )
}
```

사용 예시는 다음과 같습니다.

```ts
<Suspense fallback={<ProductListSkeleton />}>
  <ProductList />
</Suspense>
```

## 9단계. Next.js의 loading.tsx

Next.js에서는 직접 Suspense를 쓰지 않아도 loading.tsx를 만들면 자동으로 Suspense Boundary가 적용됩니다.

```ts
export default function Loading() {
  return <div>상품 목록을 불러오는 중...</div>
}
```

- 그러면 /products 페이지가 준비되기 전까지 loading.tsx가 표시됩니다.
- Next.js는 이 방식을 통해 서버에서 콘텐츠가 준비되는 동안 fallback UI를 먼저 보여주고, 준비된 콘텐츠로 자동 교체합니다.

## 10단계. 직접 Suspense와 loading.tsx 차이

- loading.tsx: 라우트 단위 로딩 UI
- <Suspense>: 컴포넌트 단위 로딩 UI
- 같이 사용 가능 여부: 가능
- 추천 사용 방식: 페이지 전체는 loading.tsx, 세부 영역은 <Suspense>

예를 들어 다음 구조가 좋습니다.

```shell
app/
  search/
    loading.tsx
    page.tsx
    SearchTrendChart.tsx
    SearchKeywordRank.tsx
```

```ts
// app/search/page.tsx

import { Suspense } from 'react'
import SearchTrendChart from './SearchTrendChart'
import SearchKeywordRank from './SearchKeywordRank'

export default function SearchPage() {
  return (
    <main>
      <h1>검색 트렌드</h1>

      <Suspense fallback={<div>검색량 차트 로딩 중...</div>}>
        <SearchTrendChart />
      </Suspense>

      <Suspense fallback={<div>검색어 순위 로딩 중...</div>}>
        <SearchKeywordRank />
      </Suspense>
    </main>
  )
}
```

## 11단계. Suspense가 useEffect fetch를 자동으로 처리하지는 않는다

아래 코드는 Suspense와 직접 연결되지 않습니다.

```ts
function ProductList() {
  const [products, setProducts] = useState<Product[]>([])
  const [isLoading, setIsLoading] = useState(true)

  useEffect(() => {
    fetch('/api/products')
      .then((res) => res.json())
      .then((data) => {
        setProducts(data)
        setIsLoading(false)
      })
  }, [])

  if (isLoading) {
    return <div>내부 로딩 중...</div>
  }

  return <div>{products.length}개 상품</div>
}
```

- 이 컴포넌트를 Suspense로 감싸도, useEffect의 로딩 상태를 Suspense가 자동으로 감지하지 않습니다.

## 12단계. Suspense와 ErrorBoundary는 같이 쓰는 경우가 많다

- Suspense는 “기다리는 상태”를 처리합니다.
- 하지만 에러는 처리하지 않습니다.
- 그래서 보통 이렇게 나눕니다.

```ts
<ErrorBoundary fallback={<div>에러가 발생했습니다.</div>}>
  <Suspense fallback={<div>로딩 중...</div>}>
    <ProductList />
  </Suspense>
</ErrorBoundary>
```

## 13단계. 실무 기준 사용 패턴

### 1. Lazy loading

```ts
import { lazy, Suspense } from 'react'

const AdminChart = lazy(() => import('./AdminChart'))

export default function AdminPage() {
  return (
    <Suspense fallback={<div>차트 로딩 중...</div>}>
      <AdminChart />
    </Suspense>
  )
}
```

- 관리자 차트, 에디터, 모달처럼 무거운 컴포넌트에 적합합니다.

## 2. Next.js 서버 컴포넌트 분리

```ts
import { Suspense } from 'react'
import SearchStats from './SearchStats'

export default function Page() {
  return (
    <div>
      <h1>검색 통계</h1>
      <Suspense fallback={<div>검색 통계 로딩 중...</div>}>
        <SearchStats />
      </Suspense>
    </div>
  )
}
```

```ts
async function getSearchStats() {
  const res = await fetch('https://example.com/api/search/stats', {
    cache: 'no-store',
  })
  return res.json()
}

export default async function SearchStats() {
  const stats = await getSearchStats()
  return (
    <ul>
      {stats.map((item: { keyword: string; count: number }) => (
        <li key={item.keyword}>
          {item.keyword}: {item.count}
        </li>
      ))}
    </ul>
  )
}
```

- SearchStats가 데이터를 기다리는 동안 Suspense fallback이 표시됩니다.

## 14단계. Suspense를 사용할 때 주의할 점

| 주의점                                         | 설명                                                   |
| ---------------------------------------------- | ------------------------------------------------------ |
| 너무 큰 영역을 감싸지 말 것                    | 작은 지연 때문에 전체 화면이 fallback으로 바뀔 수 있음 |
| fallback을 너무 단순하게 만들지 말 것          | UX가 나빠질 수 있음                                    |
| useEffect fetch는 자동 처리 안 됨              | Suspense 지원 방식으로 데이터 로딩해야 함              |
| ErrorBoundary와 역할이 다름                    | Suspense는 로딩, ErrorBoundary는 에러                  |
| Next.js에서는 `loading.tsx`와 함께 이해해야 함 | App Router의 로딩 UI가 Suspense 기반                   |
