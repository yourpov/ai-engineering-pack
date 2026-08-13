# Bun / Node.js Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Read project `AGENTS.md` if present. For reviews use `skills/review/audit/SKILL.md` (scope + mode). For naming/comments use `skills/engineering/craft/SKILL.md` (not detector scoring). Prefer extending an existing repo over scaffolding a parallel tree. Discover verify commands from the project; do not invent a toolchain.

Clean structure for server-side JavaScript/TypeScript applications.

---

## Project Structure

```
project/
├── src/
│   ├── routes/         # HTTP route handlers
│   ├── services/       # Business logic
│   ├── storage/        # Database access
│   ├── middleware/     # Request processing
│   └── utils/          # Helpers
├── tests/
├── package.json
└── tsconfig.json
```

---

## Principles

**Async/await everywhere**
No callbacks. Use promises with async/await.

**Single responsibility**
Each module does one thing well.

**Explicit dependencies**
Import what you need. No globals.

**Type safety**
Use TypeScript for production apps.

---

## Server Setup

### Bun HTTP Server

**src/server.ts**
```typescript
import { serve } from 'bun';
import { createRouter } from './routes';
import { Database } from './storage/db';

const db = new Database();
const router = createRouter(db);

serve({
  port: 3000,
  
  fetch(req) {
    return router.handle(req);
  },
});

console.log('Server running on http://localhost:3000');
```

### Express Alternative

**src/server.ts**
```typescript
import express from 'express';
import { createUserRoutes } from './routes/users';
import { Database } from './storage/db';

const app = express();
const db = new Database();

app.use(express.json());
app.use('/users', createUserRoutes(db));

app.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

---

## Routes

**src/routes/users.ts**
```typescript
import { Router } from 'express';
import { UserService } from '../services/user';
import { Database } from '../storage/db';

export function createUserRoutes(db: Database) {
  const router = Router();
  const service = new UserService(db);
  
  router.get('/', async (req, res) => {
    try {
      const users = await service.findAll();
      res.json(users);
    } catch (err) {
      res.status(500).json({ error: 'Failed to fetch users' });
    }
  });
  
  router.post('/', async (req, res) => {
    try {
      const user = await service.create(req.body);
      res.status(201).json(user);
    } catch (err) {
      if (err instanceof ValidationError) {
        res.status(400).json({ error: err.message });
      } else {
        res.status(500).json({ error: 'Failed to create user' });
      }
    }
  });
  
  router.delete('/:id', async (req, res) => {
    try {
      await service.delete(Number(req.params.id));
      res.status(204).send();
    } catch (err) {
      res.status(500).json({ error: 'Failed to delete user' });
    }
  });
  
  return router;
}
```

---

## Services

**src/services/user.ts**
```typescript
import { Database } from '../storage/db';

export interface User {
  id: number;
  name: string;
  age: number;
}

export class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

export class UserService {
  constructor(private db: Database) {}
  
  async findAll(): Promise<User[]> {
    return this.db.query('SELECT * FROM users');
  }
  
  async findById(id: number): Promise<User | null> {
    const users = await this.db.query(
      'SELECT * FROM users WHERE id = ?',
      [id]
    );
    return users[0] || null;
  }
  
  async create(data: { name: string; age: number }): Promise<User> {
    this.validate(data);
    
    const result = await this.db.query(
      'INSERT INTO users (name, age) VALUES (?, ?)',
      [data.name, data.age]
    );
    
    return {
      id: result.insertId,
      name: data.name,
      age: data.age,
    };
  }
  
  async delete(id: number): Promise<void> {
    await this.db.query('DELETE FROM users WHERE id = ?', [id]);
  }
  
  private validate(data: { name: string; age: number }) {
    if (!data.name) {
      throw new ValidationError('Name required');
    }
    
    if (data.age < 0 || data.age > 150) {
      throw new ValidationError('Invalid age');
    }
  }
}
```

---

## Database Layer

**src/storage/db.ts**
```typescript
import Database from 'better-sqlite3';

export class DB {
  private db: Database.Database;
  
  constructor(path = 'data.db') {
    this.db = new Database(path);
    this.init();
  }
  
  private init() {
    this.db.exec(`
      CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER NOT NULL
      )
    `);
  }
  
  query<T = any>(sql: string, params: any[] = []): T[] {
    return this.db.prepare(sql).all(params) as T[];
  }
  
  run(sql: string, params: any[] = []) {
    return this.db.prepare(sql).run(params);
  }
  
  close() {
    this.db.close();
  }
}
```

---

## Middleware

**src/middleware/logger.ts**
```typescript
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.path} ${res.statusCode} ${duration}ms`);
  });
  
  next();
}
```

**src/middleware/auth.ts**
```typescript
import { Request, Response, NextFunction } from 'express';

export function requireAuth(req: Request, res: Response, next: NextFunction) {
  const token = req.headers.authorization;
  
  if (!token) {
    res.status(401).json({ error: 'Unauthorized' });
    return;
  }
  
  try {
    const user = verifyToken(token);
    req.user = user;
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}

function verifyToken(token: string) {
  // Token verification logic
  return { id: 1, name: 'User' };
}
```

---

## Error Handling

**Global error handler**

```typescript
import { Request, Response, NextFunction } from 'express';

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  console.error(err);
  
  if (err instanceof ValidationError) {
    res.status(400).json({ error: err.message });
    return;
  }
  
  res.status(500).json({ error: 'Internal server error' });
}

// Use it
app.use(errorHandler);
```

**Async error wrapper**

```typescript
type AsyncHandler = (
  req: Request,
  res: Response,
  next: NextFunction
) => Promise<any>;

export function asyncHandler(fn: AsyncHandler) {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// Use it
router.get('/', asyncHandler(async (req, res) => {
  const users = await service.findAll();
  res.json(users);
}));
```

---

## Configuration

Parse and validate every env var once with Zod, export a typed `config`, and import
that everywhere, never read `process.env` deep in the app (`../standards/Principles.md` §15.1).
A missing or malformed var fails loudly at boot, not mid-request.

**src/config.ts**
```typescript
import { z } from 'zod';

const schema = z.object({
  PORT: z.string().default('3000'),
  DATABASE_URL: z.string(),
  JWT_SECRET: z.string(),
});

export const config = schema.parse(process.env);
```

**.env**
```
PORT=3000
DATABASE_URL=postgresql://localhost/mydb
JWT_SECRET=your-secret-key
```

---

## File Operations

**Read file**

```typescript
// Bun
const text = await Bun.file('data.txt').text();
const json = await Bun.file('data.json').json();

// Node.js
import { readFile } from 'fs/promises';

const text = await readFile('data.txt', 'utf-8');
const json = JSON.parse(await readFile('data.json', 'utf-8'));
```

**Write file**

```typescript
// Bun
await Bun.write('output.txt', 'Hello, world!');
await Bun.write('data.json', JSON.stringify({ name: 'Alice' }));

// Node.js
import { writeFile } from 'fs/promises';

await writeFile('output.txt', 'Hello, world!');
await writeFile('data.json', JSON.stringify({ name: 'Alice' }));
```

---

## HTTP Clients

**Fetch API (built-in)**

```typescript
async function fetchUsers() {
  const response = await fetch('https://api.example.com/users');
  
  if (!response.ok) {
    throw new Error('Failed to fetch users');
  }
  
  return response.json();
}

async function createUser(user: User) {
  const response = await fetch('https://api.example.com/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(user),
  });
  
  if (!response.ok) {
    throw new Error('Failed to create user');
  }
  
  return response.json();
}
```

---

## Testing

**Bun test**

```typescript
import { test, expect } from 'bun:test';
import { UserService } from './services/user';

test('create user with valid data', async () => {
  const db = createTestDB();
  const service = new UserService(db);
  
  const user = await service.create({ name: 'Alice', age: 30 });
  
  expect(user.name).toBe('Alice');
  expect(user.age).toBe(30);
});

test('create user with invalid age throws error', async () => {
  const db = createTestDB();
  const service = new UserService(db);
  
  expect(() => 
    service.create({ name: 'Bob', age: -5 })
  ).toThrow('Invalid age');
});
```

**Jest (Node.js)**

```typescript
import { UserService } from './services/user';

describe('UserService', () => {
  let service: UserService;
  
  beforeEach(() => {
    const db = createTestDB();
    service = new UserService(db);
  });
  
  it('creates user with valid data', async () => {
    const user = await service.create({ name: 'Alice', age: 30 });
    
    expect(user.name).toBe('Alice');
    expect(user.age).toBe(30);
  });
  
  it('throws error for invalid age', async () => {
    await expect(
      service.create({ name: 'Bob', age: -5 })
    ).rejects.toThrow('Invalid age');
  });
});
```

---

## Package Scripts

**package.json**
```json
{
  "scripts": {
    "dev": "bun --watch src/server.ts",
    "build": "bun build src/server.ts --outdir dist",
    "start": "bun dist/server.js",
    "test": "bun test"
  }
}
```

---

## Summary

Use async/await for all async operations.
Keep routes thin, logic in services.
Handle errors explicitly.
Use TypeScript for type safety.
Test services and routes.
Keep functions small and focused.

---

## Project Prompt

Build an HTTP API on Bun or Node against the structure and rules above. Where they
disagree with your defaults, this file wins.

Read `../standards/Principles.md` alongside this file before starting.

**Architecture**
- Follow the exact structure defined above
- Strict separation: routes → services → data access
- No business logic in routes
- All dependencies injected explicitly

**Security**
- Validate all input in middleware
- Never trust client data
- Environment variables for secrets
- Rate limiting
- Don't leak internals in error responses

**Error Handling**
- Custom error classes for different scenarios
- Central error middleware
- Proper HTTP status codes

**Testing**
- Unit tests for all services
- Integration tests for routes
- Mock external dependencies

### Setup

```bash
bun init
bun add express zod
bun add -d @types/express
```

### Deliverables

1. Complete project following architecture structure above
2. All routes, services, repositories
3. Database connection with proper pooling
4. Authentication middleware
5. Input validation using Zod
6. Error handling middleware
7. Environment configuration
8. README with setup instructions
9. Basic test suite

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Functions are small and single-purpose; extract when a second concern appears (see Principles / skills/engineering/craft/SKILL.md)
- [ ] No nested conditionals beyond one level
- [ ] All dependencies injected
- [ ] No comments explaining what code does
- [ ] Names match domain language and local convention (skills/engineering/craft/SKILL.md); no empty Manager/Handler/Processor stacks without a reason
- [ ] All errors handled properly
- [ ] Input validation on all routes
- [ ] SOLID principles followed
- [ ] Code reads like prose

### Pre-Delivery

```bash
bun run typecheck     # tsc --noEmit, zero errors
bun test              # all tests pass
bun run lint          # no lint errors
bun run start         # boots clean; fails fast if env is missing
```
