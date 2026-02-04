# Review Checklist (Next.js Data Fetching)

- Is the page content-correct without deferred data (no misleading empty state)?
- If metadata/SEO depends on it, is it fetched on the server (`generateMetadata`) and not client-only?
- If data is user-specific, are SSG/ISR avoided and caching disabled (`cache: 'no-store'` or equivalent)?
- Is TTFB impact considered (blocking server fetch size/latency), and are waterfalls avoided (parallelize independent fetches)?
- Is Suspense used only for independently loadable sections with an intentional fallback UI?
- Is the RSC-to-client payload minimized (avoid passing large objects to client components)?
- For interactive lists/filters, is the hybrid pattern used (server prefetch + client cache/hydration) instead of refetching everything?
