# TypeScript Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Read project `AGENTS.md` if present. For reviews use `skills/review/audit/SKILL.md` (scope + mode). For naming/comments use `skills/engineering/craft/SKILL.md` (not detector scoring). Prefer extending an existing repo over scaffolding a parallel tree. Discover verify commands from the project; do not invent a toolchain.

Clean structure for TypeScript applications.

---

## Project Structure

```
project/
├── src/
│   ├── lib/            # Business logic (domain types + rules)
│   ├── api/            # HTTP clients (I/O at the boundary)
│   ├── config.ts      # Typed, validated environment config
│   └── utils/          # Helpers
├── tests/
├── .env.example       # Documented env vars (committed; real .env is gitignored)
├── package.json
└── tsconfig.json
```

Organize `src/` by **domain**, not by type, once it grows, a `users/` folder holding
its own logic, API calls, and types beats parallel `lib/` / `api/` trees you have to
jump between. Keep it flat until that hurts.

---

## Principles

**Type everything**
Use TypeScript's type system fully.

**Single responsibility**
Each module does one thing well.

**Explicit dependencies**
Import what you need. No globals.

**Strict mode**
Enable all strict checks. No shortcuts.

---

## Business Logic

**lib/user.ts**
```typescript
export interface User {
  id: string;
  name: string;
  age: number;
}

export class ValidationError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'ValidationError';
  }
}

export function createUser(name: string, age: number): User {
  if (!name) {
    throw new ValidationError('Name required');
  }

  if (!isValidAge(age)) {
    throw new ValidationError('Invalid age');
  }

  return {
    id: crypto.randomUUID(),
    name,
    age,
  };
}

function isValidAge(age: number): boolean {
  return age >= 0 && age <= 150;
}
```

> `crypto.randomUUID()` is collision-free and available in Node 19+, Bun, Deno, and
> browsers. Never mint ids with `Date.now()`, two calls in the same millisecond
> collide. In a real app the database (or `ulid`/`uuid`) usually owns id generation.

---

## API Layer

The base URL comes from typed config (see [Configuration](#configuration)), never
hard-code a host. Errors carry the operation and status so a 3 a.m. log is useful
(`../standards/Principles.md` §5.3).

**api/users.ts**
```typescript
import { User } from '../lib/user';
import { config } from '../config';

export class ApiError extends Error {
  constructor(message: string, readonly status: number) {
    super(message);
    this.name = 'ApiError';
  }
}

async function request<T>(path: string, init?: RequestInit): Promise<T> {
  const response = await fetch(`${config.apiUrl}${path}`, init);
  if (!response.ok) {
    throw new ApiError(`${init?.method ?? 'GET'} ${path} failed`, response.status);
  }
  return response.json() as Promise<T>;
}

export function fetchUsers(): Promise<User[]> {
  return request<User[]>('/users');
}

export function saveUser(user: User): Promise<User> {
  return request<User>('/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(user),
  });
}

export async function deleteUser(id: string): Promise<void> {
  await request<void>(`/users/${id}`, { method: 'DELETE' });
}
```

One `request` helper wraps `fetch` so the URL, error shape, and JSON parsing live in
one place, adding auth headers or a retry later means editing a single function, not
three.

---

## Error Handling

**Try-catch for async operations**

Branch on your own error types, the typed `ApiError` and `ValidationError` carry
enough to react differently. Translate them into a user-facing message at the boundary
(see `skills/engineering/errors/SKILL.md` for the what/why/action rules).

```typescript
import { ApiError } from '../api/users';
import { ValidationError } from '../lib/user';

async function loadData() {
  try {
    const data = await fetchUsers();
    processData(data);
  } catch (err) {
    if (err instanceof ValidationError) {
      showError('Please check the form and try again.');
    } else if (err instanceof ApiError && err.status === 404) {
      showError('We couldn’t find those users.');
    } else {
      showError('Something went wrong loading your data. Try again in a moment.');
    }
  }
}
```

---

## Type Safety

**Use strict types**

```typescript
// Bad - any
function process(data: any) {
  return data.value;
}

// Good - specific type
interface Data {
  value: string;
}

function process(data: Data) {
  return data.value;
}
```

**Union types for variants**

```typescript
type Status = 'idle' | 'loading' | 'success' | 'error';

interface State {
  status: Status;
  data: User[] | null;
  error: string | null;
}
```

**Discriminated unions**

```typescript
type Result =
  | { success: true; data: User[] }
  | { success: false; error: string };

function handle(result: Result) {
  if (result.success) {
    console.log(result.data);  // TypeScript knows data exists
  } else {
    console.error(result.error);  // TypeScript knows error exists
  }
}
```

---

## Naming

Conventions that match the wider ecosystem and `../standards/Principles.md` §1.

```typescript
// Types & interfaces, PascalCase nouns
interface User {}
type Status = 'idle' | 'loading' | 'ready';

// Functions, camelCase verbs
function saveUser() {}
function parseConfig() {}

// Booleans, read like questions
const isActive = true;
const hasPermission = false;

// Constants, SCREAMING_SNAKE_CASE for true constants
const MAX_RETRIES = 3;

// Files, kebab-case, matching the concept
// user-service.ts, api-client.ts
```

- **No `I` prefix on interfaces** (`User`, not `IUser`), the ecosystem dropped it.
- **No Hungarian notation** (`strName`, `bActive`). The type system already knows.
- **Domain words over generic ones**, `order`, not `data`; `notify`, not `handler`.
  See `skills/engineering/craft/SKILL.md` for naming and comment quality.

---

## Testing

**Unit tests**

```typescript
import { createUser, ValidationError } from './lib/user';

test('creates user with valid data', () => {
  const user = createUser('Alice', 30);
  
  expect(user.name).toBe('Alice');
  expect(user.age).toBe(30);
});

test('throws on empty name', () => {
  expect(() => createUser('', 30)).toThrow(ValidationError);
});

test('throws on invalid age', () => {
  expect(() => createUser('Bob', -5)).toThrow(ValidationError);
});
```

---

## Configuration

**Environment config, parse once, at the boundary**

Read every env var in one place, validate it, and export a typed object. The rest of
the app imports `config` and trusts it, no `process.env.FOO` scattered through the
codebase, no `string | undefined` to guard at every use (`../standards/Principles.md` §15.1).

**src/config.ts**
```typescript
function required(name: string): string {
  const value = process.env[name];
  if (!value) {
    throw new Error(`Missing required env var: ${name}`);
  }
  return value;
}

export const config = {
  apiUrl: required('API_URL'),
  port: Number(process.env.PORT ?? 3000),
  debug: process.env.DEBUG === 'true',
} as const;
```

**.env.example** (commit this; the real `.env` is gitignored)
```
API_URL=http://localhost:3000
PORT=3000
DEBUG=false
```

Fail fast on startup if a required var is missing, a clear boot error beats a
`fetch(undefined)` deep in a request handler.

**tsconfig.json**
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true
  },
  "include": ["src"]
}
```

---

## Summary

Type everything with TypeScript.
Keep modules small and focused.
Keep business logic separate from presentation.
Use strict mode, no shortcuts.
Handle errors explicitly.

---

## Project Prompt

Write TypeScript against the structure and rules above. Where they disagree with your
defaults, this file wins.

Read `../standards/Principles.md` alongside this file before starting.

**Type Safety**
- Type everything, no `any`
- Use interfaces for contracts
- Strict mode enabled
- Discriminated unions for state

**Error Handling**
- Custom error classes for domain errors (carry context: operation, status, ids)
- Try-catch for async operations
- Never swallow errors silently

**Configuration**
- All env vars parsed and validated once in `config.ts`, exported typed
- Fail fast on a missing required var; no `process.env` reads elsewhere
- `.env.example` committed, real `.env` gitignored

**Testing**
- Unit tests for all business logic
- Mock external dependencies
- Test all error paths

### Setup

```bash
npm init -y
npm install typescript
npm install -D @types/node      # process, crypto, etc.
npx tsc --init
```

### Deliverables

1. Complete project following architecture structure above
2. Type-safe modules with interfaces
3. Business logic separated from I/O
4. Typed, validated `config.ts` + `.env.example`
5. Custom error classes that carry operation, status, and ids
6. README with setup instructions
7. Basic test suite

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Functions are small and single-purpose; extract when a second concern appears (see Principles / skills/engineering/craft/SKILL.md)
- [ ] No `any` types
- [ ] Type all function signatures
- [ ] Custom error classes for domain errors
- [ ] No hard-coded hosts/URLs, all config via `config.ts`
- [ ] Ids generated with `crypto.randomUUID()` / DB, never `Date.now()`
- [ ] Names match domain and local convention (skills/engineering/craft/SKILL.md)
- [ ] TypeScript compiles with no errors

### Pre-Delivery

```bash
npm run build
npm run type-check
npm test
```
