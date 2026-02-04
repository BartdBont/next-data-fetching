# Implementation Patterns (Next.js App Router)

These are minimal templates; use them as references, not copy-paste without thinking.

## SSG (Static)

Use when data is static for everyone and safe to cache.

```tsx
// app/(routes)/page.tsx
async function getContent() {
  const res = await fetch('https://example.com/api/content', {
    cache: 'force-cache',
  })
  if (!res.ok) throw new Error('Failed to fetch')
  return res.json()
}

export default async function Page() {
  const content = await getContent()
  return <main>{content.title}</main>
}
```

## ISR (Incremental Static Regeneration)

Use when data is shared across users and changes occasionally.

```tsx
// app/(routes)/page.tsx
export const revalidate = 300 // seconds

async function getProduct(slug: string) {
  const res = await fetch(`https://example.com/api/products/${slug}`, {
    next: { revalidate: 300 },
  })
  if (!res.ok) throw new Error('Failed to fetch')
  return res.json()
}
```

## SEO / generateMetadata Requires Data

If metadata depends on remote data, fetch on the server in `generateMetadata`.

```tsx
// app/(routes)/[slug]/page.tsx
import type { Metadata } from 'next'

async function getProduct(slug: string) {
  const res = await fetch(`https://example.com/api/products/${slug}`, {
    next: { revalidate: 300 },
  })
  if (!res.ok) return null
  return res.json()
}

export async function generateMetadata(
  props: { params: { slug: string } }
): Promise<Metadata> {
  const { slug } = props.params
  const product = await getProduct(slug)
  if (!product) return { title: 'Not found' }
  return {
    title: product.name,
    description: product.seoDescription,
  }
}

export default async function Page(props: { params: { slug: string } }) {
  const product = await getProduct(props.params.slug)
  if (!product) return null
  return <main>{product.name}</main>
}
```

## Blocking Server Fetch (Fast + Small)

Use for first-render data when the request is fast/small and the page is incorrect without it.

```tsx
// app/(routes)/checkout/page.tsx
async function fetchPricing(sessionToken: string) {
  const res = await fetch('https://example.com/api/checkout/totals', {
    headers: {
      Authorization: `Bearer ${sessionToken}`,
    },
    cache: 'no-store',
  })
  if (!res.ok) throw new Error('Failed to fetch')
  return res.json()
}

export default async function Page() {
  // Resolve the session token using your auth setup.
  const sessionToken = 'TODO'
  const pricing = await fetchPricing(sessionToken)
  return <main>Total: {pricing.grandTotal}</main>
}
```

Notes:

- User-specific data implies dynamic rendering and no shared caching.

## Server Fetch + Suspense (Streaming)

Use when some parts of the page can render content-correct before the slow data arrives.

```tsx
// app/(routes)/category/[slug]/page.tsx
import { Suspense } from 'react'

export default function Page() {
  return (
    <main>
      <h1>Category</h1>
      <Suspense fallback={<div>Loading products...</div>}>
        <ProductList />
      </Suspense>
    </main>
  )
}

async function ProductList() {
  const res = await fetch('https://example.com/api/products', {
    // Choose caching intentionally. For non-static, prefer no-store.
    cache: 'no-store',
  })
  if (!res.ok) throw new Error('Failed to fetch')
  const products = await res.json()
  return <div>{products.length} items</div>
}
```

## Client-Side Fetch

Use when the page is content-correct without the data on first render, or when deferring is acceptable.

```tsx
// app/(routes)/account/page.tsx
import AccountClient from './account-client'

export default function Page() {
  return <AccountClient />
}

// app/(routes)/account/account-client.tsx
'use client'

import { useEffect, useState } from 'react'

type Order = { id: string }

export default function AccountClient() {
  const [orders, setOrders] = useState<Order[] | null>(null)

  useEffect(() => {
    let cancelled = false
    fetch('/api/orders')
      .then((r) => r.json())
      .then((data) => {
        if (!cancelled) setOrders(data)
      })
    return () => {
      cancelled = true
    }
  }, [])

  if (!orders) return <div>Loading...</div>
  return <div>{orders.length} orders</div>
}
```

## Hybrid: Server Fetch + Client Hydrate (TanStack Query)

Use when initial server data is needed, but it must stay interactive after load (filters, sorting, refetch).

Requires `@tanstack/react-query` and your app-specific QueryClient setup.

```tsx
// app/(routes)/products/page.tsx
import ProductsClient from './products-client'

async function fetchProducts(filters: Record<string, string | string[] | undefined>) {
  const qs = new URLSearchParams()
  for (const [k, v] of Object.entries(filters)) {
    if (v === undefined) continue
    if (Array.isArray(v)) v.forEach((x) => qs.append(k, x))
    else qs.set(k, v)
  }

  const res = await fetch(`https://example.com/api/products?${qs.toString()}`, {
    cache: 'no-store',
  })
  if (!res.ok) throw new Error('Failed to fetch')
  return res.json()
}

export default async function Page(props: {
  searchParams: Record<string, string | string[] | undefined>
}) {
  const initialData = await fetchProducts(props.searchParams)
  return <ProductsClient initialData={initialData} initialFilters={props.searchParams} />
}
```

```tsx
// app/(routes)/products/products-client.tsx
'use client'

import { useQuery } from '@tanstack/react-query'

type Product = { id: string }

async function fetchProducts(filters: Record<string, string | string[] | undefined>) {
  const qs = new URLSearchParams()
  for (const [k, v] of Object.entries(filters)) {
    if (v === undefined) continue
    if (Array.isArray(v)) v.forEach((x) => qs.append(k, x))
    else qs.set(k, v)
  }

  const res = await fetch(`/api/products?${qs.toString()}`)
  if (!res.ok) throw new Error('Failed to fetch')
  return res.json()
}

export default function ProductsClient(props: {
  initialData: Product[]
  initialFilters: Record<string, string | string[] | undefined>
}) {
  // Wire your filter UI state to this value.
  const filters = props.initialFilters
  const { data, isLoading } = useQuery({
    queryKey: ['products', filters],
    queryFn: () => fetchProducts(filters),
    initialData: props.initialData,
  })
  if (isLoading) return <div>Loading...</div>
  return <div>{data.length} products</div>
}
```

Alternative: use `prefetchQuery` + `dehydrate` + `HydrationBoundary` when you already have a shared QueryClient provider on the client.
