---
name: frontend-framework-nextjs
description: React & Next.js Development Guide. Covers rendering strategies, App Router, and component patterns.
version: 1.0.0
---

# Next.js Development Guide

Guide for React and Next.js development.

---

## 1. Rendering Strategies

### SSR (Server-Side Rendering)

Use for:
- Dynamic content
- Personalized pages
- SEO-critical pages

```typescript
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic';

async function DashboardPage() {
  const data = await fetchData(); // Runs on server
  return <Dashboard data={data} />;
}
```

### SSG (Static Site Generation)

Use for:
- Marketing pages
- Documentation
- Blog posts

```typescript
// app/about/page.tsx
export default function AboutPage() {
  return <About />;
}

// This page is statically generated at build time
```

### ISR (Incremental Static Regeneration)

Use for:
- Content that updates occasionally
- E-commerce product pages

```typescript
// app/products/[id]/page.tsx
export async function generateStaticParams() {
  const products = await getProducts();
  return products.map(p => ({ id: p.id }));
}

export const revalidate = 60; // Revalidate every 60 seconds
```

---

## 2. App Router Architecture

### Layouts

```typescript
// app/layout.tsx - Root layout
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <Header />
        {children}
        <Footer />
      </body>
    </html>
  );
}

// app/dashboard/layout.tsx - Nested layout
export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <Sidebar />
      {children}
    </div>
  );
}
```

### Loading States

```typescript
// app/dashboard/loading.tsx
export default function Loading() {
  return <Skeleton />;
}

// Components show while data loads
```

---

## 3. Server vs Client Components

### Server Components (Default)

```typescript
// app/page.tsx - Server Component
async function Page() {
  const data = await db.query('SELECT * FROM posts');
  return <PostList posts={data} />;
}
```

### Client Components

```typescript
// app/components/Interactive.tsx - Client Component
'use client';

import { useState } from 'react';

export function Interactive() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### Decision Guide

| Use Server When | Use Client When |
|-----------------|-----------------|
| Fetch data | User interaction |
| Access backend directly | Event handlers |
| Sensitive info (API keys) | useState, useEffect |
| Large dependencies | Browser APIs |

---

## 4. Data Fetching

### Server-Side Fetching

```typescript
async function getData() {
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }
  });
  return res.json();
}
```

### Client-Side Fetching

```typescript
'use client';

import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId)
  });

  if (isLoading) return <Skeleton />;
  return <Profile user={data} />;
}
```

---

## Checklist

- [ ] Rendering strategy chosen per page
- [ ] Server/Client boundaries defined
- [ ] Layouts properly nested
- [ ] Loading states implemented
- [ ] Data fetching optimized
- [ ] SEO considered for public pages
