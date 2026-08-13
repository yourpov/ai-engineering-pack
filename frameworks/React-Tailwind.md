# React / Tailwind Architecture

> **Agent load:** Open Project Structure, Principles, Error Handling, Configuration, and Project Prompt / Validation first. Open other sections only when the task needs them. Pair with the matching `languages/*` file and project `AGENTS.md`. Reviews: `skills/review/audit/SKILL.md`. Craft: `skills/engineering/craft/SKILL.md`. Prefer extending an existing repo over scaffolding a parallel tree.

Framework patterns for building web applications. Use alongside `../languages/TypeScript.md` and `../standards/Principles.md`.

---

## Project Structure

```
project/
├── src/
│   ├── components/     # UI components
│   ├── hooks/          # React hooks
│   ├── context/        # Context providers
│   ├── lib/            # Business logic
│   ├── api/            # HTTP clients
│   └── utils/          # Helpers
├── public/             # Static assets
├── tests/
├── package.json
├── tsconfig.json
└── tailwind.config.js
```

---

## Principles

**Component composition over inheritance**
Build complex UIs from simple components.

**Single responsibility**
Each component does one thing well.

**Props, not global state**
Pass data explicitly where possible.

**Tailwind for styles**
Use utility classes, not custom CSS.

**Context sparingly**
Only for truly global state (auth, theme).

---

## Component Structure

### Simple Component

**Button.tsx**
```typescript
interface ButtonProps {
  text: string;
  onClick: () => void;
  disabled?: boolean;
}

export function Button({ text, onClick, disabled = false }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:opacity-50"
    >
      {text}
    </button>
  );
}
```

### Component with State

**Counter.tsx**
```typescript
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  
  function increment() {
    setCount(count + 1);
  }
  
  function reset() {
    setCount(0);
  }
  
  return (
    <div className="p-4 border rounded">
      <p className="text-lg font-medium">Count: {count}</p>
      <div className="flex gap-2 mt-2">
        <button
          onClick={increment}
          className="px-3 py-1 bg-green-600 text-white rounded"
        >
          Increment
        </button>
        <button
          onClick={reset}
          className="px-3 py-1 bg-gray-600 text-white rounded"
        >
          Reset
        </button>
      </div>
    </div>
  );
}
```

### List Component

**UserList.tsx**
```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserListProps {
  users: User[];
  onSelect: (user: User) => void;
}

export function UserList({ users, onSelect }: UserListProps) {
  if (users.length === 0) {
    return <p className="text-gray-500">No users found</p>;
  }
  
  return (
    <ul className="space-y-2">
      {users.map(user => (
        <UserItem
          key={user.id}
          user={user}
          onSelect={onSelect}
        />
      ))}
    </ul>
  );
}

interface UserItemProps {
  user: User;
  onSelect: (user: User) => void;
}

function UserItem({ user, onSelect }: UserItemProps) {
  return (
    <li
      onClick={() => onSelect(user)}
      className="p-3 border rounded hover:bg-gray-50 cursor-pointer"
    >
      <p className="font-medium">{user.name}</p>
      <p className="text-sm text-gray-600">{user.email}</p>
    </li>
  );
}
```

---

## Custom Hooks

**hooks/useUsers.ts**
```typescript
import { useState, useEffect } from 'react';
import { User } from '../lib/user';
import { fetchUsers } from '../api/users';

export function useUsers() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    loadUsers();
  }, []);
  
  async function loadUsers() {
    try {
      setLoading(true);
      const data = await fetchUsers();
      setUsers(data);
      setError(null);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }
  
  return { users, loading, error, reload: loadUsers };
}
```

**hooks/useForm.ts**
```typescript
import { useState } from 'react';

export function useForm<T>(initial: T) {
  const [values, setValues] = useState<T>(initial);
  
  function update(field: keyof T, value: T[keyof T]) {
    setValues(prev => ({ ...prev, [field]: value }));
  }
  
  function reset() {
    setValues(initial);
  }
  
  return { values, update, reset };
}
```

---

## Error Handling

React has two distinct failure paths, handle both:

- **Render errors** → an **error boundary** catches them and shows fallback UI instead
  of unmounting the whole tree.
- **Async / event errors** (fetch, handlers) → boundaries *don't* catch these. Handle
  them in state and render the error inline (see `skills/engineering/errors/SKILL.md` for the message
  rules).

**Error boundary** (catches render-time errors):

```typescript
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
}

export class ErrorBoundary extends Component<Props, State> {
  state = { hasError: false };
  
  static getDerivedStateFromError() {
    return { hasError: true };
  }
  
  componentDidCatch(error: Error) {
    console.error('Error caught:', error);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="p-4 bg-red-50 border border-red-200 rounded">
          <p className="text-red-800">Something went wrong</p>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

**Async errors** (boundaries can't catch these, keep them in state):

```typescript
function Users() {
  const [error, setError] = useState<string | null>(null);
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    fetchUsers()
      .then(setUsers)
      .catch(() => setError('Couldn’t load users. Try again in a moment.'));
  }, []);

  if (error) {
    return <p className="text-red-800" role="alert">{error}</p>;
  }
  return <UserList users={users} />;
}
```

---

## TailwindCSS Patterns

**Component styles**

```typescript
// Bad - inline styles
<div style={{ padding: '16px', backgroundColor: '#fff' }}>

// Good - Tailwind utilities
<div className="p-4 bg-white">
```

**Conditional classes**

```typescript
import clsx from 'clsx';

interface ButtonProps {
  variant: 'primary' | 'secondary';
  disabled?: boolean;
}

export function Button({ variant, disabled }: ButtonProps) {
  return (
    <button
      className={clsx(
        'px-4 py-2 rounded',
        variant === 'primary' && 'bg-blue-600 text-white',
        variant === 'secondary' && 'bg-gray-200 text-gray-800',
        disabled && 'opacity-50 cursor-not-allowed'
      )}
      disabled={disabled}
    >
      Click me
    </button>
  );
}
```

**Reusable class groups**

```typescript
// Bad - repeated classes
<div className="p-4 bg-white border rounded shadow">
<div className="p-4 bg-white border rounded shadow">

// Good - extract to component
function Card({ children }) {
  return (
    <div className="p-4 bg-white border rounded shadow">
      {children}
    </div>
  );
}
```

**Responsive design**

```typescript
<div className="
  flex flex-col
  md:flex-row
  gap-4
  p-4
  md:p-6
  lg:p-8
">
  <div className="w-full md:w-1/2">Left</div>
  <div className="w-full md:w-1/2">Right</div>
</div>
```

---

## State Management

**Local state for simple cases**

```typescript
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  function add(text: string) {
    setTodos([...todos, { id: Date.now(), text }]);
  }
  
  function remove(id: number) {
    setTodos(todos.filter(t => t.id !== id));
  }
  
  return <div>{/* ... */}</div>;
}
```

**Context for shared state**

```typescript
import { createContext, useContext, useState, ReactNode } from 'react';

interface User {
  name: string;
  email: string;
}

interface AuthContextValue {
  user: User | null;
  login: (user: User) => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  function login(newUser: User) {
    setUser(newUser);
  }
  
  function logout() {
    setUser(null);
  }
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

---

## Testing

**Component tests**

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from './Button';

test('renders button text', () => {
  render(<Button text="Click me" onClick={() => {}} />);
  expect(screen.getByText('Click me')).toBeInTheDocument();
});

test('calls onClick when clicked', () => {
  const onClick = jest.fn();
  render(<Button text="Click" onClick={onClick} />);
  
  fireEvent.click(screen.getByText('Click'));
  expect(onClick).toHaveBeenCalledTimes(1);
});

test('disabled button does not trigger onClick', () => {
  const onClick = jest.fn();
  render(<Button text="Click" onClick={onClick} disabled />);
  
  fireEvent.click(screen.getByText('Click'));
  expect(onClick).not.toHaveBeenCalled();
});
```

**Hook tests**

```typescript
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('increments counter', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});
```

---

## Configuration

**tailwind.config.js**
```javascript
export default {
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

---

## Summary

Build small, focused components.
Use Tailwind utilities, not custom CSS.
Custom hooks for reusable logic.
Context sparingly, only for global state.
Test components and hooks.
Error boundaries for graceful failure.

---

## Project Prompt

Build a React and Tailwind frontend against the structure and rules above. Where they
disagree with your defaults, this file wins.

Read `../languages/TypeScript.md` and `../standards/Principles.md` alongside this file before starting.

**Styling**
- Tailwind CSS 4.0 only, no custom CSS unless necessary
- Utility-first approach
- Mobile-first responsive design
- Consistent spacing scale

**Components**
- Composition over inheritance
- Props for data flow, context only for global state
- Components stay focused; split when a file mixes unrelated concerns
- Error boundaries for graceful failure

**Performance**
- Memoize expensive calculations
- Lazy load routes and components
- Virtualize long lists
- Debounce event handlers

### Setup

```bash
npm create vite@latest project-name -- --template react-ts
cd project-name
npm install
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### Deliverables

1. Complete project following architecture structure above
2. Type-safe React components
3. Custom hooks for reusable logic
4. Context providers for state
5. Tailwind styling throughout
6. Routing with React Router
7. Form handling with validation
8. Error boundaries
9. README with setup instructions
10. Basic test suite

### Validation Checklist

- [ ] Verify commands from project AGENTS.md / README run (or honest manual checks listed)
- [ ] No secrets committed; env examples use placeholders only

- [ ] Components stay focused; split when UI, data fetching, and business rules share one file without need
- [ ] No `any` types
- [ ] Type all props and state
- [ ] No inline styles (use Tailwind)
- [ ] Custom hooks for logic
- [ ] Error boundaries implemented
- [ ] Loading states handled
- [ ] Names match domain and local convention (skills/engineering/craft/SKILL.md)
- [ ] TypeScript compiles with no errors

### Pre-Delivery

```bash
npm run build
npm run type-check
npm test
```
