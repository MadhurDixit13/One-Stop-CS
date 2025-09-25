# TypeScript - Language Guide

TypeScript is a statically typed superset of JavaScript that compiles to plain JS. It adds types, interfaces, enums, and modern tooling that improve developer productivity and code quality.

---

## Quick Syntax Tour

```ts
// Basic types
let id: number = 1;
let title: string = 'Hello';
let ok: boolean = true;
let tags: string[] = ['ts', 'js'];
let tuple: [number, string] = [1, 'one'];

// Enums
enum Status { Pending = 'PENDING', Done = 'DONE' }

// Interfaces & types
interface User {
  id: number;
  name: string;
  email?: string; // optional
}

type Result<T> = { ok: true; value: T } | { ok: false; error: string };

// Functions
function greet(name: string): string {
  return `Hello, ${name}`;
}

// Generics
function first<T>(xs: T[]): T | undefined {
  return xs[0];
}

// Narrowing
function length(x: string | string[]): number {
  return typeof x === 'string' ? x.length : x.length;
}

// Classes
class Queue<T> {
  private items: T[] = [];
  enqueue(item: T) { this.items.push(item); }
  dequeue(): T | undefined { return this.items.shift(); }
}
```

---

## tsconfig.json (Recommended)

```json
{
  "compilerOptions": {
    "target": "ES2021",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "exactOptionalPropertyTypes": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "outDir": "dist"
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

---

## Tooling

- Build: `tsc`, Vite, esbuild, SWC
- Lint: ESLint (`@typescript-eslint/parser`, `@typescript-eslint/eslint-plugin`)
- Format: Prettier
- Test: Vitest / Jest
- Types: `npm i -D @types/node` (and other `@types/*` packages)

.eslintrc.cjs
```js
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:import/recommended',
    'plugin:import/typescript',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'prettier'
  ],
  settings: { react: { version: 'detect' } },
};
```

---

## Advanced Types & Patterns

```ts
// Discriminated unions
interface Loading { status: 'loading' }
interface Success<T> { status: 'success'; data: T }
interface Failure { status: 'failure'; error: string }

type RemoteData<T> = Loading | Success<T> | Failure;

function render(user: RemoteData<User>) {
  switch (user.status) {
    case 'loading': return 'Loading...';
    case 'failure': return user.error;
    case 'success': return user.data.name; // fully narrowed
  }
}

// Utility types
type PartialUser = Partial<User>;
type ReadonlyUser = Readonly<User>;

// Mapped types
type APIResponse<T> = { data: T; meta: { page: number; total: number } };

// Infer with generics
function unwrap<T>(p: Promise<T>): Promise<T> { return p; }
```

---

## Working with React

```tsx
import { useState } from 'react';

type ButtonProps = {
  onClick?: () => void;
  children: React.ReactNode;
  disabled?: boolean;
};

export function Button({ onClick, children, disabled = false }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

function Counter() {
  const [count, setCount] = useState<number>(0);
  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}
```

---

## Node.js Interop

```ts
// ESM
import fs from 'node:fs/promises';

export async function read(path: string): Promise<string> {
  return await fs.readFile(path, 'utf8');
}

// CommonJS
// const fs = require('node:fs/promises');
```

---

## Testing (Vitest)

```ts
import { describe, it, expect } from 'vitest';

function add(a: number, b: number) { return a + b }

describe('add', () => {
  it('adds numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
});
```

---

## Best Practices

- Enable `strict` and related compiler flags
- Prefer unions/interfaces over `any`
- Avoid type assertions; use narrowing and generics
- Keep runtime and type-level logic aligned
- Centralize shared types in `types.ts` or a dedicated package
