# Examples (Decision Tree In Practice)

Use these as references, not as copy-paste without thinking.

## Product Detail Page

Data:

- Product
- Price

Decision path:

- Needed for first render: yes
- SEO/metadata depends on it: yes
- User-specific: no
- Static/semi-static: yes

Choice:

- ISR

See `references/patterns-app-router.md` (ISR + generateMetadata).

## Dashboard / Account Page

Data:

- Orders
- Account details

Decision path:

- Needed for first render: no
- User-specific: yes

Choice:

- Client-side fetch

See `references/patterns-app-router.md` (Client-side fetch).

## Category Page With Product List

Data:

- Product list

Decision path:

- Needed for first render: yes
- SEO/metadata depends on it: yes
- User-specific: no
- Not static
- Latency: 300-700ms

Choice:

- Server fetch + Suspense (streaming)

See `references/patterns-app-router.md` (Streaming Suspense).

## Checkout - Pricing & Totals

Data:

- Products
- Quantities
- VAT/discounts

Decision path:

- Needed for first render: yes
- SEO/metadata depends on it: no
- User-specific: yes
- Fast/small enough: yes

Choice:

- Blocking server fetch (with auth)

See `references/patterns-app-router.md` (Blocking server fetch).

## Product Overview With Filters (Hybrid Pattern)

Data:

- Product list
- Filters

Decision path:

- Needed for first render: yes
- SEO/metadata depends on it: yes
- User-specific: no
- Not static
- Data remains interactive after load: yes

Choice:

- Hybrid: server fetch + client hydrate

See `references/patterns-app-router.md` (Hybrid TanStack Query).

## CMS Content Page

Data:

- Page content

Decision path:

- Needed for first render: yes
- SEO/metadata depends on it: yes
- User-specific: no
- Static

Choice:

- SSG

See `references/patterns-app-router.md` (SSG).
