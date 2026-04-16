---
name: nextjs-14-app-router
description: Patterns, conventions, and gotchas for Next.js 14+ App Router (NOT Pages Router). Use this skill whenever the user mentions Next.js, Next 14, Next 15, App Router, server components, server actions, route handlers, or is building any modern React frontend — especially e-commerce storefronts, marketing sites, or dashboards. Also trigger for The PotFather project or any headless storefront consuming Medusa/Shopify/Strapi APIs. Critical: the App Router is fundamentally different from the Pages Router — don't mix getServerSideProps, _app.tsx, or pages/api with the App Router paradigm.
---

# Next.js 14+ App Router Patterns

The App Router (directory: `app/`) replaces the Pages Router (`pages/`). Rules, mental model, and APIs are different. Always target App Router unless the user explicitly says otherwise.

## Mental Model

Every file in `app/` is a **Server Component by default**. Server Components:
- Run on the server, never ship JS to the client
- Can `async/await` directly in the component body
- Can access DB, env vars, secrets directly
- CANNOT use `useState`, `useEffect`, `useContext`, event handlers, or browser APIs

To opt into client rendering, add `"use client"` at the top of the file. Client components:
- Hydrate on the client
- Can use hooks, event handlers, effects
- CANNOT be `async` at the top level
- CANNOT directly import server-only modules (DB drivers, fs, etc.)

## File Conventions (inside `app/`)

| File | Purpose |
|------|---------|
| `page.tsx` | Route UI (renders at the segment's URL) |
| `layout.tsx` | Shared UI wrapping children; persists across navigation |
| `template.tsx` | Like layout but remounts on navigation |
| `loading.tsx` | React Suspense fallback for the segment |
| `error.tsx` | Error boundary (must be `"use client"`) |
| `not-found.tsx` | 404 UI for the segment |
| `route.ts` | API route handler (replaces `pages/api`) |
| `default.tsx` | Fallback for parallel routes |
| `middleware.ts` | At project root, runs before matched requests |

## Data Fetching

### Server Component (preferred)
```tsx
// app/products/[handle]/page.tsx
export default async function ProductPage({
  params,
}: {
  params: { handle: string }
}) {
  const product = await fetch(`${process.env.MEDUSA_URL}/store/products?handle=${params.handle}`, {
    headers: { "x-publishable-api-key": process.env.MEDUSA_PUB_KEY! },
    next: { revalidate: 60 }, // ISR: cache 60s
  }).then(r => r.json())

  return <ProductView product={product} />
}
```

### Caching semantics
`fetch()` in Server Components is auto-cached. Control it with:
- `{ cache: "no-store" }` — always dynamic (SSR on every request)
- `{ cache: "force-cache" }` — default in Next 14, removed default in Next 15
- `{ next: { revalidate: 60 } }` — ISR every 60s
- `{ next: { tags: ["products"] } }` — on-demand revalidation via `revalidateTag("products")`

**Next 15 change**: `fetch` is NOT cached by default anymore. Opt in explicitly.

### Server Actions (form submission, mutations)
```tsx
// app/cart/actions.ts
"use server"

import { revalidateTag } from "next/cache"
import { cookies } from "next/headers"

export async function addToCart(formData: FormData) {
  const variantId = formData.get("variant_id") as string
  const cartId = cookies().get("cart_id")?.value

  await fetch(`${process.env.MEDUSA_URL}/store/carts/${cartId}/line-items`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ variant_id: variantId, quantity: 1 }),
  })

  revalidateTag("cart")
}
```

Use from a form:
```tsx
<form action={addToCart}>
  <input type="hidden" name="variant_id" value={variant.id} />
  <button type="submit">Add to cart</button>
</form>
```

## Route Handlers (API endpoints)

```ts
// app/api/webhook/stripe/route.ts
import { NextRequest, NextResponse } from "next/server"

export async function POST(req: NextRequest) {
  const body = await req.text()
  const sig = req.headers.get("stripe-signature")
  // verify + process
  return NextResponse.json({ received: true })
}
```

Do NOT use `pages/api/*` in App Router projects.

## Metadata (SEO)

```tsx
// app/products/[handle]/page.tsx
import type { Metadata } from "next"

export async function generateMetadata({ params }): Promise<Metadata> {
  const product = await getProduct(params.handle)
  return {
    title: `${product.title} | The PotFather`,
    description: product.description,
    openGraph: {
      images: [product.thumbnail],
    },
  }
}
```

For dynamic OG images use `app/opengraph-image.tsx` with `ImageResponse` from `next/og`.

## Routing Patterns

- **Dynamic segments**: `app/products/[handle]/page.tsx` → `params.handle`
- **Catch-all**: `app/docs/[...slug]/page.tsx` → `params.slug: string[]`
- **Optional catch-all**: `app/shop/[[...filters]]/page.tsx`
- **Route groups** (don't affect URL): `app/(marketing)/about/page.tsx`
- **Parallel routes**: `app/@modal/...` + `{modal}` slot in layout
- **Intercepting routes**: `(.)`, `(..)`, `(...)`

## Middleware

```ts
// middleware.ts (root)
import { NextResponse } from "next/server"
import type { NextRequest } from "next/server"

const BLOCKED_STATES = ["ID", "KS", "AR", "RI", "HI", "CO", "MN", "VT"]

export function middleware(req: NextRequest) {
  const geo = req.geo?.region
  if (geo && BLOCKED_STATES.includes(geo)) {
    return NextResponse.redirect(new URL("/state-restricted", req.url))
  }
  return NextResponse.next()
}

export const config = {
  matcher: ["/((?!_next|api|static).*)"],
}
```

Middleware runs on the Edge runtime — no Node APIs, no DB drivers.

## Common Gotchas

1. **`"use client"` cascades down the tree** — marking a component as client makes its children client too (unless you pass server components as `children` props).
2. **Don't import server-only code into client components.** Wrap in `import "server-only"` to enforce.
3. **`cookies()` and `headers()` are server-only** — use in Server Components, Server Actions, or Route Handlers.
4. **Async client components don't work** — only Server Components can be async.
5. **Images**: always use `next/image`. For external domains, add to `next.config.js` `images.remotePatterns`.
6. **Environment variables**: `NEXT_PUBLIC_*` exposed to client; others server-only.
7. **Streaming**: wrap slow parts in `<Suspense>` with a `loading.tsx` fallback.

## Performance Patterns

- **Prefetching**: `<Link>` prefetches by default in production
- **`revalidatePath('/path')`** and **`revalidateTag('tag')`** in Server Actions for cache busting
- **`generateStaticParams`** for SSG of dynamic routes:
```tsx
export async function generateStaticParams() {
  const products = await getAllProducts()
  return products.map(p => ({ handle: p.handle }))
}
```
- **Partial Prerendering (experimental in 14, stable soon)**: mix static shell + dynamic holes

## Project Structure for E-commerce

```
app/
├── (shop)/
│   ├── layout.tsx                  # shop chrome (header, footer)
│   ├── page.tsx                    # homepage
│   ├── products/
│   │   ├── page.tsx                # PLP
│   │   └── [handle]/page.tsx       # PDP
│   ├── cart/page.tsx
│   └── checkout/
│       ├── page.tsx
│       └── actions.ts              # server actions
├── (account)/
│   ├── layout.tsx
│   ├── login/page.tsx
│   └── orders/page.tsx
├── api/
│   └── webhook/
│       └── stripe/route.ts
├── layout.tsx                      # root layout (html, body)
└── globals.css

lib/
├── medusa/
│   ├── client.ts                   # fetch wrapper
│   └── queries.ts                  # typed helpers
└── utils.ts
```

## Anti-patterns

- **Don't** use `getServerSideProps` or `getStaticProps` in App Router
- **Don't** wrap the whole app in `"use client"`
- **Don't** fetch in `useEffect` when you could fetch in a Server Component
- **Don't** use `next/router` — use `next/navigation` (`useRouter`, `usePathname`, `useSearchParams`)
- **Don't** put mutations in GET route handlers — use Server Actions or POST
- **Don't** forget the `publishable_api_key` header when calling Medusa store endpoints

## Client/Server boundary cheat-sheet

| Need | Where |
|------|-------|
| Fetch data | Server Component |
| Form submit + mutate | Server Action |
| Interactive UI (click, state) | Client Component |
| Read cookies | `cookies()` in server / document.cookie via js-cookie in client |
| Env var secret | Server only; never `NEXT_PUBLIC_*` |
| Database query | Server Component / Server Action / Route Handler |
| Browser API (window, localStorage) | Client Component in `useEffect` |
