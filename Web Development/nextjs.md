# Next.js ⚡

[![Next.js](https://img.shields.io/badge/Next.js-React%20Framework-black)](https://nextjs.org/)

> React framework for production: routing, SSR/SSG/ISR, API routes, and edge.

## Create App
```bash
npx create-next-app@latest my-app --ts --eslint --app --src-dir --import-alias '@/*'
cd my-app
npm run dev
```

## App Router Structure
```
src/
  app/
    layout.tsx
    page.tsx
    api/
      health/route.ts
  components/
  lib/
```

## Pages
```tsx
// src/app/page.tsx
export default function Home() {
  return (
    <main className="p-6">
      <h1 className="text-2xl font-bold">Hello Next.js</h1>
    </main>
  );
}
```

## Data Fetching
```tsx
// Server component (default)
async function getPosts() {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts', { cache: 'no-store' });
  return res.json();
}

export default async function Page() {
  const posts = await getPosts();
  return <pre>{JSON.stringify(posts.slice(0, 3), null, 2)}</pre>;
}
```

```tsx
// Client component
'use client';
import { useEffect, useState } from 'react';

export function Clock() {
  const [now, setNow] = useState(Date.now());
  useEffect(() => {
    const id = setInterval(() => setNow(Date.now()), 1000);
    return () => clearInterval(id);
  }, []);
  return <span>{new Date(now).toLocaleTimeString()}</span>;
}
```

## API Routes
```ts
// src/app/api/health/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  return NextResponse.json({ ok: true });
}
```

## Metadata & SEO
```ts
// src/app/page.tsx
export const metadata = {
  title: 'Home',
  description: 'Welcome',
};
```

## Styling
- Tailwind CSS: `npx tailwindcss init -p`
- CSS Modules / global.css

## Auth (NextAuth)
```bash
npm i next-auth
```

```ts
// src/app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import GitHub from 'next-auth/providers/github';

export const { handlers, auth } = NextAuth({
  providers: [GitHub],
});

export const GET = handlers.GET;
export const POST = handlers.POST;
```

## Env Config
```ts
// src/lib/env.ts
import { z } from 'zod';

const Env = z.object({
  NEXT_PUBLIC_API_URL: z.string().url().optional(),
  GITHUB_ID: z.string().optional(),
  GITHUB_SECRET: z.string().optional(),
});

export const env = Env.parse(process.env);
```

## Testing
```bash
npm i -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: { environment: 'jsdom', setupFiles: ['./test/setup.ts'] }
});
```

```ts
// test/setup.ts
import '@testing-library/jest-dom';
```

```tsx
// test/home.test.tsx
import { render, screen } from '@testing-library/react';
import Home from '@/app/page';

test('renders heading', () => {
  render(<Home />);
  expect(screen.getByText('Hello Next.js')).toBeInTheDocument();
});
```

## Deployment
- Vercel: zero-config, edge/serverless runtimes.
- Docker: multi-stage, `node:18-alpine`, `RUN npm ci && npm run build`.

## Best Practices
- Prefer Server Components; client only when needed.
- Cache strategies: `force-cache`, `no-store`, revalidate.
- Schema-validate env and inputs; handle errors.
- Accessibility and performance (Lighthouse).

## Resources
- Docs: nextjs.org/docs
- Learn: nextjs.org/learn

