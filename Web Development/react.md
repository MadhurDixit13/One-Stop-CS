# What is React

React is a JavaScript library for building user interfaces. It lets you describe what the UI should look like for any given state using components and JSX, and it efficiently updates the DOM when your data changes. React focuses on the view layer and integrates well with libraries for routing, state management, data fetching, and styling.
# What Problem it solves?
React solves the problem of building complex, dynamic user interfaces (UIs) in a manageable and efficient way. Before React, developers often struggled with keeping the UI consistent with the underlying application state as it changed over time. Here are the core problems React addresses:

1. UI-State Synchronization

Traditional JavaScript or jQuery apps required developers to manually manipulate the DOM whenever data changed.

This led to "UI drift" (the UI getting out of sync with the actual data), especially as apps grew in complexity.

React solves this with its declarative model: you describe what the UI should look like for a given state, and React updates it automatically when the state changes.

2. Complexity of Large Applications

As applications scale, different parts of the UI depend on different pieces of state, and interactions between them get messy.

React introduces a component-based architecture, where the UI is broken down into small, reusable building blocks.

Each component manages its own state (or receives it as props), making the system more modular and maintainable.

3. Performance Issues with DOM Updates

Direct DOM manipulation is expensive, and repeatedly updating it can slow down the application.

React introduced the Virtual DOM, an in-memory representation of the UI. React computes the minimal changes needed and efficiently updates the real DOM, improving performance.

4. Code Reusability and Maintainability

Without a structured framework, UI code (HTML, CSS, JavaScript) often became tangled and hard to reuse.

React’s components encapsulate logic, structure, and styling, making it easier to reuse, test, and reason about different parts of the app.

✅ In short: React’s main contribution is that it makes building interactive, stateful, and large-scale UIs predictable, efficient, and easier to reason about.

Examples

### Example 1: Keeping UI in sync with state (Counter)

Before (imperative DOM manipulation)

```html
<div id="app">
  <span id="value">0</span>
  <button id="inc">+1</button>
</div>
<script>
  let count = 0;
  const valueEl = document.getElementById('value');
  const incBtn = document.getElementById('inc');

  function render() {
    valueEl.textContent = count; // Manually sync UI to state
  }

  incBtn.addEventListener('click', () => {
    count++;
    render(); // You must remember to call this everywhere state changes
  });

  render();
</script>
```

Notes:
- You must remember to update the DOM every time the data changes
- Easy to forget a render call → stale UI

After (React: declarative + Virtual DOM)

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <span>{count}</span>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
    </div>
  );
}
```

Notes:
- You describe what the UI should look like for a given `count`
- React handles updating the DOM efficiently whenever `count` changes

### Example 2: Multiple UI parts depending on shared state (list + filter + selection)

Before (imperative; easy to drift out of sync)

```html
<input id="query" placeholder="Search..." />
<ul id="list"></ul>
<script>
  let items = ["Apple", "Banana", "Cherry"];
  let selected = new Set();
  let query = "";

  const listEl = document.getElementById('list');
  const qEl = document.getElementById('query');

  function render() {
    listEl.innerHTML = "";
    items
      .filter(x => x.toLowerCase().includes(query))
      .forEach(item => {
        const li = document.createElement('li');
        li.textContent = item;
        if (selected.has(item)) li.style.fontWeight = 'bold';
        li.onclick = () => { toggle(item); };
        listEl.appendChild(li);
      });
  }

  function toggle(item) {
    if (selected.has(item)) selected.delete(item);
    else selected.add(item);
    render(); // must remember to re-render here
  }

  qEl.addEventListener('input', (e) => {
    query = e.target.value.toLowerCase();
    render(); // and here
  });

  render();
</script>
```

Notes:
- You must thread re-renders through every event path
- As features grow (sorting, paging, async loads), this becomes brittle

After (React components + state drives everything)

```jsx
import { useMemo, useState } from 'react';

export default function FruitPicker() {
  const [query, setQuery] = useState('');
  const [selected, setSelected] = useState(new Set());
  const items = ['Apple', 'Banana', 'Cherry'];

  const filtered = useMemo(
    () => items.filter(x => x.toLowerCase().includes(query.toLowerCase())),
    [items, query]
  );

  function toggle(item) {
    setSelected(prev => {
      const next = new Set(prev);
      next.has(item) ? next.delete(item) : next.add(item);
      return next;
    });
  }

  return (
    <div className="space-y-2">
      <input
        placeholder="Search..."
        value={query}
        onChange={e => setQuery(e.target.value)}
      />
      <ul>
        {filtered.map(item => (
          <li
            key={item}
            style={{ fontWeight: selected.has(item) ? '700' : '400', cursor: 'pointer' }}
            onClick={() => toggle(item)}
          >
            {item}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

What this shows:
- Declarative UI: Describe what the UI should be; React figures out how to update it
- Component model: Encapsulate logic + view; reuse and test easily
- Predictable updates: State changes → React re-renders → DOM updates are minimal and correct

---

## Core Concepts

- **Components**: Reusable building blocks that render UI. Can be functions or classes (modern React uses function components).
- **JSX**: Syntax extension that lets you write HTML-like code in JavaScript. It compiles to `React.createElement` calls.
- **Props**: Read-only inputs to a component. Passed from parent to child.
- **State**: Internal, mutable data managed by a component via hooks like `useState`.
- **Render Cycle**: State/props changes trigger re-renders; React reconciles changes to the DOM.
- **Hooks**: Functions that let you use state and other React features in function components (`useState`, `useEffect`, `useMemo`, etc.).

### JSX Example
```jsx
function Hello({ name }) {
  return <h1>Hello, {name}!</h1>;
}
```

### Props vs State
```jsx
import { useState } from 'react';

function Counter({ step = 1 }) { // step is a prop (read-only)
  const [count, setCount] = useState(0); // count is state (mutable via setter)
  return (
    <button onClick={() => setCount(c => c + step)}>
      Count: {count}
    </button>
  );
}
```

### Essential Hooks
- **useState**: Local component state
  ```jsx
  const [value, setValue] = useState('');
  ```
- **useEffect**: Side effects (data fetching, subscriptions)
  ```jsx
  useEffect(() => {
    document.title = `Count ${count}`;
    return () => { /* cleanup if needed */ };
  }, [count]);
  ```
- **useMemo**: Memoize expensive computations
  ```jsx
  const filtered = useMemo(() => items.filter(f), [items, f]);
  ```
- **useCallback**: Memoize function references to avoid unnecessary renders
  ```jsx
  const onClick = useCallback(() => doSomething(id), [id]);
  ```

---

## Project Setup

Modern, fast setup using Vite:
```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
```

For TypeScript:
```bash
npm create vite@latest my-app -- --template react-ts
```

Alternative toolchains: Next.js for full-stack React apps, Remix for web standards-first routing, CRA is legacy.

### Suggested Folder Structure
```text
src/
  assets/
  components/
  hooks/
  pages/
  routes/
  services/      # API clients, data fetching helpers
  store/         # state management (if needed)
  styles/
  App.jsx/tsx
  main.jsx/tsx
```

---

## Common Commands (NPM)

- `npm run dev` – start local dev server
- `npm run build` – production build
- `npm run preview` – preview production build locally
- `npm test` – run tests (if configured)
- `npm run lint` – run linter
- `npm run format` – format with Prettier (if configured)

Yarn/PNPM equivalents: `yarn dev`, `pnpm dev`, etc.

---

## Routing (react-router)

Install and basic usage:
```bash
npm install react-router-dom
```
```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import Home from './pages/Home';
import About from './pages/About';

const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/about', element: <About /> },
]);

export default function App() {
  return <RouterProvider router={router} />;
}
```

---

## Forms (Controlled Inputs)
```jsx
import { useState } from 'react';

function SignupForm() {
  const [form, setForm] = useState({ email: '', password: '' });
  const onChange = e => setForm({ ...form, [e.target.name]: e.target.value });
  const onSubmit = e => { e.preventDefault(); /* submit */ };

  return (
    <form onSubmit={onSubmit}>
      <input name="email" value={form.email} onChange={onChange} />
      <input name="password" type="password" value={form.password} onChange={onChange} />
      <button type="submit">Create account</button>
    </form>
  );
}
```

---

## Data Fetching

Basic fetch with `useEffect`:
```jsx
import { useEffect, useState } from 'react';

function Users() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;
    async function load() {
      try {
        const res = await fetch('https://jsonplaceholder.typicode.com/users');
        if (!res.ok) throw new Error('Network error');
        const data = await res.json();
        if (!cancelled) setUsers(data);
      } catch (e) {
        if (!cancelled) setError(e);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }
    load();
    return () => { cancelled = true; };
  }, []);

  if (loading) return <p>Loading…</p>;
  if (error) return <p>Error: {error.message}</p>;
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

Consider libraries for caching and mutations:
- TanStack Query (React Query)
- SWR

---

## Performance Essentials

- Split components to minimize re-renders; lift state only when necessary.
- Memoize expensive computations with `useMemo`.
- Stabilize callbacks passed to children with `useCallback`.
- Wrap pure child components with `React.memo`.
- Use key props correctly when rendering lists.

```jsx
const ListItem = React.memo(function ListItem({ item, onSelect }) {
  return <li onClick={() => onSelect(item.id)}>{item.label}</li>;
});
```

---

## Testing (Vitest + React Testing Library)

Install:
```bash
npm install -D vitest @testing-library/react @testing-library/user-event jsdom
```

Example test:
```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Counter from './Counter';

test('increments', async () => {
  render(<Counter />);
  await userEvent.click(screen.getByRole('button'));
  expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
});
```

---

## TypeScript Tips

```tsx
type ButtonProps = {
  onClick: () => void;
  children: React.ReactNode;
};

export function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}
```

---

## Styling Options

- CSS Modules, Tailwind CSS, Vanilla Extract, styled-components, Emotion.
- Prefer co-locating styles with components for maintainability.

---

## Common Pitfalls

- Mutating state instead of creating new objects/arrays.
- Forgetting dependency arrays in `useEffect` (or adding unstable references).
- Overlifting state; keep state as local as possible.
- Using indexes as keys for dynamic lists.

---

## Further Reading

- Official React docs: https://react.dev/learn
- React Router: https://reactrouter.com/
- TanStack Query: https://tanstack.com/query/latest
- Testing Library: https://testing-library.com/docs/react-testing-library/intro/
