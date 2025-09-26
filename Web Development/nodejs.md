# Node.js ⚙️

[![Node.js](https://img.shields.io/badge/Node.js-Runtime-339933)](https://nodejs.org/)

> JavaScript runtime built on V8 for servers, CLIs, and tooling.

## Install
```bash
# Windows/macOS/Linux
nvm install --lts
nvm use --lts
node -v
npm -v
```

## Project Setup
```bash
mkdir my-app && cd my-app
npm init -y
npm i typescript ts-node @types/node -D
npx tsc --init --rootDir src --outDir dist --esModuleInterop true
```

## Scripts
```json
{
  "type": "module",
  "scripts": {
    "dev": "tsx src/index.ts",
    "build": "tsc -p .",
    "start": "node dist/index.js",
    "lint": "eslint .",
    "test": "vitest"
  }
}
```

## HTTP Server
```ts
// src/index.ts
import http from 'node:http';

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({ ok: true }));
});

server.listen(3000, () => console.log('http://localhost:3000'));
```

## ESM vs CJS
- Prefer ESM (`"type": "module"`).
- Use `import`/`export`. For CJS, use `require`/`module.exports`.

## Env & Config
```ts
import 'dotenv/config';
process.env.NODE_ENV; // 'development' | 'production'
```

## Async Patterns
- Prefer `async/await`; wrap with small helpers.
```ts
export async function withTry<T>(p: Promise<T>): Promise<[T|null, Error|null]> {
  try { return [await p, null]; } catch (e: any) { return [null, e]; }
}
```

## Worker Threads
```ts
import { Worker } from 'node:worker_threads';
new Worker(new URL('./worker.js', import.meta.url));
```

## Streaming
```ts
import { createReadStream, createWriteStream } from 'node:fs';
createReadStream('in').pipe(createWriteStream('out'));
```

## Observability
```ts
import pino from 'pino';
const log = pino({ level: process.env.LOG_LEVEL || 'info' });
log.info({ event: 'startup' }, 'app ready');
```

## Testing
```bash
npm i -D vitest @vitest/coverage-v8
npx vitest run
```

## Lint & Format
```bash
npm i -D eslint @eslint/js typescript-eslint prettier
```

## Packaging
- CLIs: `pkg`, `oclif`.
- Libraries: `exports` map, TS types, semantic versioning.

## Best Practices
- **12-factor** envs, small modules, typed public APIs.
- Graceful shutdown: handle `SIGINT`/`SIGTERM`.
- Limit concurrency; use pools/queues.
- Avoid blocking: offload CPU to workers.

## Resources
- Docs: nodejs.org
- Patterns: `https://github.com/goldbergyoni/nodebestpractices`
