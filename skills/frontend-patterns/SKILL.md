---
name: frontend-patterns
description: React, Next.js App Router, TypeScript, and Tailwind CSS patterns for building modern web applications and data dashboards.
origin: dham-claude-code
---

# Frontend Patterns

Use this skill for building web UIs, data dashboards, and APIs with React, Next.js, and TypeScript.

## Next.js App Router Structure

```
src/
├── app/
│   ├── layout.tsx              # Root layout
│   ├── page.tsx                # Home page (Server Component)
│   ├── (dashboard)/            # Route group — no URL segment
│   │   ├── orders/
│   │   │   ├── page.tsx        # Server Component: fetch + render
│   │   │   ├── loading.tsx     # Suspense fallback
│   │   │   └── error.tsx       # Error boundary
│   │   └── analytics/
│   └── api/
│       └── orders/
│           └── route.ts        # Route Handler
├── components/
│   ├── ui/                     # Shadcn/Radix primitives
│   └── features/               # Domain-specific components
├── lib/
│   ├── db.ts                   # Database client
│   └── validations.ts          # Zod schemas
└── types/
    └── index.ts
```

## Server vs Client Components

```typescript
// Server Component: data fetching, no hooks, no event handlers
// app/orders/page.tsx
async function OrdersPage({ searchParams }: { searchParams: { status?: string } }) {
  const orders = await db.orders.findMany({
    where: { status: searchParams.status },
    orderBy: { createdAt: 'desc' },
    take: 50,
  })
  return (
    <div>
      <h1>Orders</h1>
      <OrdersFilter />          {/* Client Component */}
      <OrdersList orders={orders} />  {/* Can be Server */}
    </div>
  )
}

// Client Component: only for interactivity
// components/features/OrdersFilter.tsx
'use client'
import { useRouter, useSearchParams } from 'next/navigation'

export function OrdersFilter() {
  const router = useRouter()
  const searchParams = useSearchParams()
  return (
    <select
      value={searchParams.get('status') ?? ''}
      onChange={e => router.push(`?status=${e.target.value}`)}
    >
      <option value="">All</option>
      <option value="pending">Pending</option>
    </select>
  )
}
```

## API Route Handler

```typescript
// app/api/orders/route.ts
import { z } from 'zod'
import { NextResponse } from 'next/server'

const CreateOrderSchema = z.object({
  customerId: z.string().uuid(),
  items: z.array(z.object({
    productId: z.string(),
    quantity: z.number().int().positive(),
  })).min(1),
})

export async function POST(req: Request) {
  const body = await req.json()
  const parsed = CreateOrderSchema.safeParse(body)
  if (!parsed.success) {
    return NextResponse.json({ error: parsed.error.flatten() }, { status: 400 })
  }
  const order = await createOrder(parsed.data)
  return NextResponse.json(order, { status: 201 })
}
```

## Data Fetching with React Query

```typescript
// For client-side data that needs caching/refetching
'use client'
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'

export function useOrders(status?: string) {
  return useQuery({
    queryKey: ['orders', status],
    queryFn: () => fetch(`/api/orders?status=${status}`).then(r => r.json()),
    staleTime: 1000 * 60,  // 1 minute
  })
}

export function useUpdateOrderStatus() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: ({ id, status }: { id: string; status: string }) =>
      fetch(`/api/orders/${id}`, { method: 'PATCH', body: JSON.stringify({ status }) }).then(r => r.json()),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['orders'] }),
  })
}
```

## Tailwind Component Patterns

```typescript
// Reusable status badge
const statusStyles: Record<string, string> = {
  pending:   'bg-yellow-100 text-yellow-800',
  shipped:   'bg-blue-100 text-blue-800',
  delivered: 'bg-green-100 text-green-800',
  cancelled: 'bg-red-100 text-red-800',
}

function StatusBadge({ status }: { status: string }) {
  return (
    <span className={`inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium ${statusStyles[status] ?? 'bg-gray-100 text-gray-800'}`}>
      {status}
    </span>
  )
}
```

## Key Rules

- Default to Server Components; add `'use client'` only when needed
- Validate all API inputs with Zod before processing
- Use `loading.tsx` and `error.tsx` for every route segment
- `next/image` for all images; `next/link` for all internal navigation
- Environment variables: server-only vars never imported in Client Components
