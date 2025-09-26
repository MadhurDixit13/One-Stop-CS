# Express.js 🚏

[![Express](https://img.shields.io/badge/Express.js-Web%20Framework-black)](https://expressjs.com/)

> Minimal, unopinionated web framework for Node.js.

## Install
```bash
npm i express zod helmet morgan cors dotenv
npm i -D tsx typescript @types/node @types/express supertest vitest
```

## App Skeleton
```ts
// src/app.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';

const app = express();
app.use(helmet());
app.use(cors());
app.use(morgan('dev'));
app.use(express.json());

app.get('/health', (_req, res) => res.json({ ok: true }));

export default app;
```

```ts
// src/server.ts
import 'dotenv/config';
import app from './app';

const port = Number(process.env.PORT || 3000);
const server = app.listen(port, () => console.log(`http://localhost:${port}`));

process.on('SIGINT', () => server.close(() => process.exit(0)));
process.on('SIGTERM', () => server.close(() => process.exit(0)));
```

## Routing
```ts
// src/routes/users.ts
import { Router } from 'express';
const r = Router();

r.get('/', async (_req, res) => {
  res.json([{ id: '1', name: 'Alice' }]);
});

r.post('/', async (req, res) => {
  res.status(201).json({ id: '2', ...req.body });
});

export default r;
```

```ts
// src/app.ts (mount routes)
import users from './routes/users';
app.use('/users', users);
```

## Validation
```ts
// src/middleware/validate.ts
import { AnyZodObject } from 'zod';
import { Request, Response, NextFunction } from 'express';

export const validate = (schema: AnyZodObject) =>
  (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse({
      body: req.body,
      query: req.query,
      params: req.params,
    });
    if (!result.success) {
      return res.status(400).json({ errors: result.error.flatten() });
    }
    next();
  };
```

```ts
// src/routes/users.ts (with schema)
import { z } from 'zod';
import { validate } from '../middleware/validate';

const CreateUser = z.object({
  body: z.object({
    name: z.string().min(1),
    email: z.string().email(),
  })
});

r.post('/', validate(CreateUser), async (req, res) => {
  res.status(201).json({ id: '2', ...req.body });
});
```

## Auth (JWT)
```ts
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'missing token' });
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!);
    (req as any).user = payload;
    next();
  } catch {
    return res.status(401).json({ error: 'invalid token' });
  }
}
```

## Error Handling
```ts
// src/middleware/errors.ts
import { Request, Response, NextFunction } from 'express';

export function notFound(_req: Request, res: Response) {
  res.status(404).json({ error: 'not found' });
}

export function onError(err: any, _req: Request, res: Response, _next: NextFunction) {
  const status = err.status || 500;
  res.status(status).json({ error: err.message || 'internal error' });
}
```

```ts
// src/app.ts (mount at end)
import { notFound, onError } from './middleware/errors';
app.use(notFound);
app.use(onError);
```

## Tests
```ts
// test/health.test.ts
import request from 'supertest';
import app from '../src/app';

test('health', async () => {
  const res = await request(app).get('/health');
  expect(res.status).toBe(200);
  expect(res.body.ok).toBe(true);
});
```

## Best Practices
- Smaller routers, layered services, DTO validation.
- Security: `helmet`, rate limit, CORS allowlist.
- Observability: request IDs, structured logs, metrics.
- Performance: pagination, indexes, caching.
- Migrations: `prisma`, `knex`, or `sequelize`.

## Deployment
- Dockerfile + multi-stage build, healthcheck, env-only configs.

## Resources
- Docs: expressjs.com
- Examples: `https://github.com/expressjs/express/tree/master/examples`

