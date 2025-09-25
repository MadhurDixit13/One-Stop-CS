# JavaScript - Language Guide

JavaScript is the language of the web. Modern JS (ES2015+) provides powerful syntax, modules, async/await, classes, and a massive ecosystem for frontend, backend (Node.js), tooling, and beyond.

---

## Quick Syntax Tour

```js
// Variables
const pi = 3.14159;     // immutable binding
let count = 0;          // mutable binding

// Strings & templates
const name = 'Alice';
const msg = `Hello, ${name}!`;

// Arrays & objects
const nums = [1, 2, 3];
const user = { id: 1, name: 'Alice' };
const { id, name: fullName } = user;  // destructuring
const doubled = nums.map(n => n * 2);

// Functions
function add(a, b) { return a + b }
const mul = (a, b) => a * b;

// Classes
class Queue {
  #items = []; // private field
  enqueue(x) { this.#items.push(x) }
  dequeue() { return this.#items.shift() }
}

// Modules (ESM)
export function greet(name) { return `Hi, ${name}` }
// import { greet } from './utils.js'
```

---

## Async Programming

```js
// Promises
function fetchJson(url) {
  return fetch(url).then(r => {
    if (!r.ok) throw new Error('Network');
    return r.json();
  });
}

// async/await
async function getUser(id) {
  const r = await fetch(`/api/users/${id}`);
  if (!r.ok) throw new Error('Not found');
  return await r.json();
}

// Parallel
async function load() {
  const [a, b] = await Promise.all([
    fetch('/a').then(r => r.text()),
    fetch('/b').then(r => r.text())
  ]);
  return { a, b };
}
```

---

## Tooling

- Package manager: `npm`, `yarn`, `pnpm`
- Bundler/Dev server: Vite, Webpack, esbuild, Parcel
- Lint/Format: ESLint + Prettier
- Test: Vitest, Jest, Playwright

package.json
```json
{
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint .",
    "format": "prettier -w .",
    "test": "vitest"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0",
    "vitest": "^1.5.0"
  }
}
```

.eslintrc.cjs
```js
module.exports = {
  root: true,
  env: { browser: true, es2023: true, node: true },
  extends: ['eslint:recommended', 'prettier'],
};
```

.prettierrc
```json
{ "singleQuote": true, "semi": true, "printWidth": 100 }
```

---

## DOM & Events (Browser)

```js
// Query
const button = document.querySelector('#submit');
const list = document.getElementById('items');

// Events
button.addEventListener('click', (e) => {
  e.preventDefault();
  const li = document.createElement('li');
  li.textContent = 'Clicked!';
  list.appendChild(li);
});

// Fetch
async function loadUsers() {
  const res = await fetch('/api/users');
  const users = await res.json();
  list.innerHTML = users.map(u => `<li>${u.name}</li>`).join('');
}
loadUsers();
```

---

## Node.js Basics

```js
// ESM
import fs from 'node:fs/promises';

export async function read(path) {
  return await fs.readFile(path, 'utf8');
}

// HTTP server
import http from 'node:http';

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ ok: true }));
});

server.listen(3000);
```

---

## Testing (Vitest)

```js
import { describe, it, expect } from 'vitest';

function add(a, b) { return a + b }

describe('add', () => {
  it('adds numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
});
```

---

## Best Practices

- Prefer ESM (`type: module`) and modern syntax
- Keep functions small and pure; handle errors with `try/catch`
- Avoid global mutable state; use modules for encapsulation
- Lint and format in CI; pin tool versions
- For large apps, adopt TypeScript or JSDoc types for safety
