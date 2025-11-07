---
description: "Updated: 2025-07-10. React best practices and patterns for modern web applications. Cre: https://github.com/PatrickJS/awesome-cursorrules/blob/main/rules-new/react.mdc and https://github.com/github/awesome-copilot/blob/main/instructions/reactjs.instructions.md"
applyTo: "**/*.jsx,**/*.tsx"
---

# React Best Practices

Instructions for building high-quality ReactJS applications with modern patterns, hooks, and best practices following the official React documentation at https://react.dev.

## Project Tech Stack

- Language: `TypeScript` and `TypeScriptReact`
- Core: `React v18`, `React Router DOM v6`, ...
- UI library: `radix-ui`.
- Styling: `TailwindCSS` + plugins
- Internationalization: `i18next` and `react-i18next`. **CRITICAL: Never include `t` or `i18n` object into the dependency array of React Hooks.**
- Icon: `Lucide React`.
- Toast: `Sonner`.
- Programming language: `TypeScript ES2020`.
- Build tool: `Vite v6`.
- Validation: `zod v3`
- Local state management: `zustand`.
- Server state management: `@tanstack/react-query`
- Form: `react-hook-form`
- Utility: `clsx`, `class-variance-authority`, `date-fns`
- Linting and formatting: `ESLint` (React, Hooks, Tailwind CSS plugins)

- Functional components and hooks are the default pattern, zero legacy class component dependencies.
- Modern libraries are used for forms, validation, and state managements.

## Development Standards

### Architecture

- Favor Functional Component over Class Components. Functional components are simpler, easier to test, and leverage React hooks for state and side effects, leading to more maintainable code.

```jsx
// Bad
class UserCard extends React.Component<{ name: string }> {
  render() {
    return <div>{this.props.name}</div>;
  }
}

// Good
function UserCard({ name }: { name: string }) {
  return <div>{name}</div>;
}
```

- Structure components using composition patterns such as children, render props, or hooks. Favor components over utilize inheritance for component logic reuse. Composition is more flexible and predictable than inheritance, allowing for better code reuse and separation of concerns.

```jsx
// Bad: Inheritance
class BaseCard extends React.Component {}
class UserCard extends BaseCard {}

// Good: Composition with children
function Card({ children }: { children: React.ReactNode }) {
  return <div className="card">{children}</div>;
}
```

- Split any components exceeding 500 lines, or, handling more than one responsibility into smaller, focused components. Not only practice SRY which reduce complexity and improve maintainability but smaller components are easier to test, debug, and reuse.

```jsx
// Good: Focused React component
function UserProfile({ user }: { user: User }) {
  return (
    <div>
      <Avatar src={user.avatar} />
      <UserDetails user={user} />
    </div>
  );
}
```

- Separate presentational (UI only) and container (logic/state) components. Separating presentational and container components improves code clarity, reusability, and testability by isolating UI from business logic.

```jsx
// Good: seperated
function UserList({ users }: { users: User[] }) {
  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
function UserListContainer() {
  const users = useFetchUsers();
  return <UserList users={users} />;
}
```

- Implement proper component hierarchies with clear data flow. Utilize `zustand` to avoid prop drilling. Clear hierarchies and data flow prevent confusion and bugs, while context eliminates excessive prop passing and simplifies state sharing.

  Example:

  ```jsx
  import { create } from 'zustand'

  type User = {
    id: number
    name: string
    avatarUrl?: string
  }

  interface UserStoreState {
    user: User | null
    setUser: (user: User | null) => void
  }

  const useUserStore = create<UserStoreState>((set) => ({
    user: null,
    setUser: (user) => set({ user }),
  }))

  function GrandParent({ user }: { user: User }) {
    const setUser = useUserStore((state) => state.setUser)
    React.useEffect(() => {
      setUser(user)
    }, [user, setUser])
    return <Parent />
  }

  function Parent() {
    return <Child />
  }

  function Child() {
    const user = useUserStore((state) => state.user)
    return <div>{user?.name}</div>
  }
  ```

- You MUST organize components by feature or domain for scalability. Organizing by feature or domain improves scalability, maintainability, and makes it easier to locate related code as the application grows.

### TypeScript Integration

- Utilize TypeScript interfaces for props, state, and component definitions. Interfaces provide strong typing, improve code safety, and make components easier to understand and maintain.

```jsx
interface ButtonProps {
  label: string;
  onClick: () => void;
}
function Button({ label, onClick }: ButtonProps) {
  return <button onClick={onClick}>{label}</button>;
}
```

- Define proper types for event handlers and refs. Proper typing for events and refs prevents runtime errors and improves code reliability and developer experience.

```jsx
function Input({
  onChange,
}: {
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void,
}) {
  return <input onChange={onChange} />;
}
const inputRef = React.useRef < HTMLInputElement > null;
```

- Implement generic components where appropriate.
  Reasoning: Generics enable reusable, type-safe components that work with multiple data types, improving flexibility and reducing duplication.
  Example:

  ```jsx
  interface ListProps<T> {
    items: T[];
    renderItem: (item: T) => React.ReactNode;
  }
  function List<T>({ items, renderItem }: ListProps<T>) {
    return <ul>{items.map(renderItem)}</ul>;
  }
  ```

- Utilize TypeScript strict mode type safety. Strict mode enforces type safety, catches potential bugs early, and ensures robust code quality.

```json
{
  "compilerOptions": {
    "strict": true
  }
}
```

- Leverage React's built-in types (React.FC, React.ComponentProps, etc.). Using built-in types ensures compatibility with React's ecosystem and improves type safety and code clarity.

```jsx
const MyComponent: React.FC<{ name: string }> = ({ name }) => <div>{name}</div>;
type ButtonProps = React.ComponentProps<"button">;
```

- Create union types for component variants and states. Union types make component APIs more expressive and type-safe, allowing for clear definition of allowed values and states.

```jsx
type ButtonVariant = "primary" | "secondary" | "danger";
type ButtonProps = {
  variant: ButtonVariant,
  label: string,
};

function Button({ variant, label }: ButtonProps) {
  return <button className={variant}>{label}</button>;
}
```

### Component Design

- Follow the SRY (Single Responsibility Principle).

```jsx
function Avatar({ src }: { src: string }) {
  return <img src={src} alt="User avatar" />;
}
function UserDetails({ user }: { user: User }) {
  return <div>{user.name}</div>;
}
// Each component does one thing only.
```

- Utilize descriptive and consistent naming conventions.

```jsx
// Bad
function Up({ u }: { u: User }) {
  return <div>{u.name}</div>;
}
// Good
function UserProfileCard({ user }: { user: User }) {
  return <div>{user.name}</div>;
}
```

- Implement proper prop validation with TypeScript or PropTypes. Prop validation ensures components receive correct data, preventing runtime errors and improving reliability.

```jsx
interface ButtonProps {
  label: string;
  onClick: () => void;
}
function Button({ label, onClick }: ButtonProps) {
  return <button onClick={onClick}>{label}</button>;
}
```

- Design components to be testable and reusable. Testable and reusable components reduce duplication, improve reliability, and make it easier to build scalable applications.

```jsx
function Input({
  value,
  onChange,
}: {
  value: string,
  onChange: (v: string) => void,
}) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}
// Can be reused in multiple forms.
```

- Keep components small and focused on a single concern.

```jsx
function LoadingSpinner() {
  return <div>Loading...</div>;
}
```

- Utilize Composition patterns (render props, children as functions). Composition patterns enable flexible component reuse and separation of concerns, improving maintainability and scalability.

```jsx
function List<T>({
  items,
  renderItem,
}: {
  items: T[],
  renderItem: (item: T) => React.ReactNode,
}) {
  return <ul>{items.map(renderItem)}</ul>;
}

// Usage
<List items={[1, 2, 3]} renderItem={(item) => <li key={item}>{item}</li>} />;
```

### State Management

- Utilize the `useState()` hook for all local component state. `useState` is simple, efficient, and designed for managing local state in functional components.

```jsx
const [value, setValue] = React.useState("");
```

- You MUST utilize the `useReducer()` hook for state logic involving multiple sub-values or complex state transitions. It centralizes complex state logic, making it easier to manage, test, and debug.

```jsx
const [state, dispatch] = React.useReducer(reducer, initialState);
```

- Leverage `zustand` for sharing state across component trees. You MUST NOT utilize Zustand for local state. Zustand is intended for global/shared state; using it for local state adds unnecessary complexity and reduces performance.

```jsx
import React from "react"
import { create } from "zustand"

type CounterState = {
  count: number
  increment: () => void
  decrement: () => void
  reset: () => void
}

const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
  decrement: () => set((s) => ({ count: s.count - 1 })),
  reset: () => set({ count: 0 }),
}))

function CounterDisplay() {
  const count = useCounterStore((s) => s.count)
  return <div>Shared count: {count}</div>
}

function CounterControls() {
  const increment = useCounterStore((s) => s.increment)
  const decrement = useCounterStore((s) => s.decrement)
  const reset = useCounterStore((s) => s.reset)

  return (
    <div style={{ display: "flex", gap: 8 }}>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
      <button onClick={reset}>Reset</button>
    </div>
  )
}

export default function App() {
  return (
    <div style={{ fontFamily: "sans-serif", padding: 16 }}>
      <h3>Zustand shared state example</h3>
      <CounterDisplay />
      <CounterControls />
    </div>
  )
}
```

- Keep state as close as possible to the components that utilize it. You MUST NOT lift state unnecessarily. Keeping state local reduces coupling, improves performance, and makes components easier to understand and maintain.

```jsx
// Bad: State lifted to parent when not needed
function Parent() {
  const [query, setQuery] = React.useState("");
  return <SearchInput query={query} setQuery={setQuery} />;
}

// Good: Local state in child component
function SearchInput() {
  const [query, setQuery] = React.useState("");
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

- Utilize `@tanstack/react-query` for server state management. Server state management library includes robust and reliable data fetching, caching, synchronization, and updating data from external sources like APIs

```jsx
// Good: Utilize React Query for server state
import { useQuery } from "@tanstack/react-query";
function UserList() {
  const { data, error, isLoading } = useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then((res) => res.json()),
  });
  // ...
}
```

- DO NOT utilize prop drilling when passing data through more than two component levels. Prop drilling makes code harder to maintain and refactor.

```jsx
// Bad: Prop drilling through multiple layers
function GrandParent({ user }) {
  return <Parent user={user} />;
}
function Parent({ user }) {
  return <Child user={user} />;
}
function Child({ user }) {
  return <div>{user.name}</div>;
}

// Good: Utilize Zustand for shared/global state
import { create } from "zustand";

type User = {
  id: number,
  name: string,
};

interface UserStoreState {
  user: User | null;
  setUser: (user: User | null) => void;
}

const useUserStore =
  create <
  UserStoreState >
  ((set) => ({
    user: null,
    setUser: (user) => set({ user }),
  }));

function GrandParent({ user }: { user: User }) {
  const setUser = useUserStore((state) => state.setUser);
  React.useEffect(() => {
    setUser(user);
  }, [user, setUser]);
  return <Parent />;
}

function Parent() {
  return <Child />;
}

function Child() {
  const user = useUserStore((state) => state.user);
  return <div>{user?.name}</div>;
}
```

- Implement state normalization and data structures. Normalized state prevents duplication, simplifies updates, and improves performance, especially for collections and relational data.

```jsx
// Bad: Nested, denormalized state
const [users, setUsers] = React.useState([
  { id: 1, name: "Alice", posts: [{ id: 101, title: "Hello" }] },
  { id: 2, name: "Bob", posts: [{ id: 102, title: "World" }] },
]);

// Good: Normalized state using objects and IDs
const [users, setUsers] = React.useState<{ [id: number]: User }>({
  1: { id: 1, name: "Alice" },
  2: { id: 2, name: "Bob" },
});
const [posts, setPosts] = React.useState<{ [id: number]: Post }>({
  101: { id: 101, userId: 1, title: "Hello" },
  102: { id: 102, userId: 2, title: "World" },
});
const [userPostIds, setUserPostIds] = React.useState<{
  [userId: number]: number[];
}>({
  1: [101],
  2: [102],
});
```

### Hooks

- Always follow the official Rules of Hooks:

  - Utilize Hooks at the top level of React functions, before any early returns
  - Call Hooks while React is rendering a function component, more specifically, at the top level in the body of a function component or at the top level in the body of a custom Hook.
  - DO NOT call Hooks inside loops, conditional statements, nested functions, after a conditional `return` statement, event handlers, inside functions passed to `useMemo()`, `useReducer()`, `useEffect()`.

  Violating the Rules of Hooks can lead to unpredictable behavior and bugs, as React relies on consistent hook order for state management.

```jsx
function Bad({ cond }) {
  if (cond) {
    // Bad: inside a condition (to fix, move it outside!)
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  for (let i = 0; i < 10; i++) {
    // Bad: inside a loop (to fix, move it outside!)
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad({ cond }) {
  if (cond) {
    return;
  }
  // Bad: after a conditional return (to fix, move it before the return!)
  const theme = useContext(ThemeContext);
  // ...
}

function Bad() {
  function handleClick() {
    // Bad: inside an event handler (to fix, move it outside!)
    const theme = useContext(ThemeContext);
  }
  // ...
}

function Bad() {
  const style = useMemo(() => {
    // Bad: inside useMemo (to fix, move it outside!)
    const theme = useContext(ThemeContext);
    return createStyle(theme);
  });
  // ...
}

class Bad extends React.Component {
  render() {
    // Bad: inside a class component (to fix, write a function component instead of a class!)
    useEffect(() => {});
    // ...
  }
}

function Counter() {
  // Good: top-level in a function component
  const [count, setCount] = useState(0);
  // ...
}

function useWindowWidth() {
  // Good: top-level in a custom Hook
  const [width, setWidth] = useState(window.innerWidth);
  // ...
}
```

- Extract reusable logic into custom hooks. You MUST NOT duplicate logic across components. Custom hooks promote DRY principles, improve testability, and centralize logic for easier updates and bug fixes.

  Example:

  ```jsx
  // Good: Custom hook
  function useWindowWidth() {
    const [width, setWidth] = React.useState(window.innerWidth);
    React.useEffect(() => {
      const handleResize = () => setWidth(window.innerWidth);
      window.addEventListener("resize", handleResize);
      return () => window.removeEventListener("resize", handleResize);
    }, []);
    return width;
  }

  // Usage
  function Sidebar() {
    const width = useWindowWidth();
    // ...
  }
  ```

- Always include all external dependencies in the dependency array of `useEffect` and other hooks that accept dependencies. Specifying dependencies ensures effects run correctly and prevents bugs related to stale closures or missed updates or infinity loops.

```jsx
React.useEffect(() => {
  // ...
}, [dependency1, dependency2]);
```

- implement cleanup functions in `useEffect()` when the effect creates subscriptions, timers, or side effects. Cleanup functions prevent memory leaks and unintended side effects, ensuring resources are released when components unmount.

```jsx
React.useEffect(() => {
  const id = setInterval(doSomething, 1000);
  return () => clearInterval(id);
}, []);
```

- Utilize `useMemo()` and `useCallback()` to memoize expensive computations and functions. You MUST only memoize when there is a measurable performance benefit. These hooks optimize performance by memoizing expensive computations and stable function references, preventing unnecessary recalculations and re-renders when dependencies have not changed.

```jsx
// Good:
// useMemo for expensive computation
const sortedList = React.useMemo(() => {
  return items.sort((a, b) => a.value - b.value);
}, [items]);

// useCallback for stable function reference
const handleClick = React.useCallback(() => {
  doSomething();
}, [doSomething]);
```

- Utilize `useRef()` for accessing DOM elements and storing mutable values that do not trigger re-renders. It lets you persist values across renders without causing updates, making it ideal for referencing DOM nodes or storing mutable data.

  Example:

  ```jsx
  // Accessing DOM element
  function FocusInput() {
    const inputRef = React.useRef < HTMLInputElement > null;
    React.useEffect(() => {
      inputRef.current?.focus();
    }, []);
    return <input ref={inputRef} />;
  }

  // Storing mutable value
  function Counter() {
    const countRef = React.useRef(0);
    function increment() {
      countRef.current += 1;
    }
    return <button onClick={increment}>Increment</button>;
  }
  ```

- DO NOT nest hooks inside other hooks or functions. Nesting hooks violates the Rules of Hooks, leading to unpredictable state and rendering issues.

```jsx
// Bad
function useBadHook() {
  function inner() {
    React.useState(0);
  }
}
```

### Performance Optimization

- Wrap expensive or frequently re-rendered components with `React.memo`. `React.memo` prevents unnecessary re-renders, improving performance for components with stable props.

```jsx
const MemoizedList = React.memo(List);
```

- Utilize `useCallback()` and `useMemo()` to avoid unnecessary re-renders by ensuring stable props and memoizing callbacks. Stable props and memoized callbacks prevent child components from re-rendering when not needed, improving efficiency.

```jsx
// useCallback()
// Good: Stable props and memoized callback
const handleClick = React.useCallback(() => {
  doSomething();
}, [doSomething]);

<ChildComponent onClick={handleClick} />

// Bad: Inline function causes new reference each render
<ChildComponent onClick={() => doSomething()} />
```

```jsx
// useMemo()
// Good: Stable object prop using useMemo
const config = React.useMemo(() => ({ theme: 'dark' }), []);

<SettingsPanel config={config} />

// Bad: New object reference every render
<SettingsPanel config={{ theme: 'dark' }} />
```

- Implement lazy loading for components that are not needed immediately using `React.lazy` and `Suspense`. Lazy loading reduces initial bundle size and speeds up page load by loading components only when needed.

```jsx
const LazyComponent = React.lazy(() => import("./LazyComponent"));
```

- Optimize bundle size with tree shaking and dynamic imports. Tree shaking removes unused code from the final bundle, and dynamic imports load code only when needed, reducing initial load time and improving performance.

```jsx
// Tree shaking: Only import what you utilize
import { Button } from "@radix-ui/react-button"; // Good: Only imports Button, not the whole library

// Dynamic import: Load component only when needed
const LazySettingsPanel = React.lazy(() => import("./SettingsPanel"));

function App() {
  const [showSettings, setShowSettings] = React.useState(false);

  return (
    <div>
      <button onClick={() => setShowSettings(true)}>Show Settings</button>
      {showSettings && (
        <React.Suspense fallback={<div>Loading...</div>}>
          <LazySettingsPanel />
        </React.Suspense>
      )}
    </div>
  );
}
```

- Provide a unique and stable `key` prop for each item in a rendered list.

```jsx
{
  items.map((item) => <li key={item.id}>{item.name}</li>);
}
```

- Profile components with React DevTools to identify performance bottlenecks. Profiling with React DevTools helps detect inefficient renders and resource usage, enabling targeted performance optimizations.

### Data Fetching with `@tanstack/react-query`

```jsx
import { useQuery } from "@tanstack/react-query";

function UserList() {
  const { data, error, isLoading } = useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then((res) => res.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div role="alert">Failed to load users.</div>;
  return (
    <ul>
      {data.map((user: User) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

- Implement proper loading, error, and success states for all data fetching operations.

```jsx
const { data, error, isLoading } = useQuery({ ... });
if (isLoading) return <div>Loading...</div>;
if (error) return <div role="alert">Error loading data.</div>;
return <div>Success! {JSON.stringify(data)}</div>;
```

- Handle race conditions and request cancellation to prevent outdated data from overwriting newer results.

```jsx
import { useQuery } from "@tanstack/react-query";

function Search({ query }: { query: string }) {
  const { data, isFetching } = useQuery({
    queryKey: ["search", query],
    queryFn: () => fetch(`/api/search?q=${query}`).then((res) => res.json()),
    enabled: !!query,
  });

  return (
    <div>
      {isFetching && <span>Searching...</span>}
      {data && (
        <ul>
          {data.results.map((r: Result) => (
            <li key={r.id}>{r.title}</li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

- Utilize optimistic updates for better user experience when mutating data.

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function AddTodo() {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: (newTodo: Todo) =>
      fetch("/api/todos", { method: "POST", body: JSON.stringify(newTodo) }),
    onMutate: async (newTodo) => {
      await queryClient.cancelQueries(["todos"]);
      const previousTodos = queryClient.getQueryData(["todos"]);
      queryClient.setQueryData(["todos"], (old: Todo[] = []) => [
        ...old,
        newTodo,
      ]);
      return { previousTodos };
    },
    onError: (_err, _newTodo, context) => {
      queryClient.setQueryData(["todos"], context?.previousTodos);
    },
    onSettled: () => {
      queryClient.invalidateQueries(["todos"]);
    },
  });

  // ...
}
```

- Implement proper caching strategies to avoid unnecessary network requests and improve performance.

```jsx
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
  staleTime: 1000 * 60 * 5, // cache for 5 minutes
  cacheTime: 1000 * 60 * 10, // keep cache for 10 minutes
});
```

- Handle offline scenarios and network errors gracefully, providing fallback UI and retry options.

```jsx
const { error, isFetching } = useQuery({ ... });

if (error) {
  return (
    <div role="alert">
      Network error. <button onClick={() => refetch()}>Retry</button>
    </div>
  );
}
if (isFetching) return <div>Loading...</div>;
```

### Error Handling

- Introduce `ErrorBoundary` for component-level error handling. Error boundaries prevent the entire app from crashing and provide fallback UI, improving reliability and user experience.

```jsx
import { ErrorBoundary } from "react-error-boundary";

function ErrorFallback() {
  return <div>Unable to load component. Please refresh the page.</div>;
}

<ErrorBoundary FallbackComponent={ErrorFallback}>
  <LazyComponent />
</ErrorBoundary>;
```

- Utilize proper error states in data fetching.

```jsx
import { useQuery } from "@tanstack/react-query";

function UserList() {
  const { data, error, isLoading } = useQuery({
    queryKey: ["users"],
    queryFn: () => fetch("/api/users").then((res) => res.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div role="alert">Failed to load users.</div>;
  return (
    <ul>
      {data.map((user: User) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

- Handle errors from asynchronous operations using `try/catch` blocks or error states. Proper error handling prevents silent failures and enables user-friendly error reporting and recovery.

```jsx
async function fetchData() {
  try {
    const response = await fetch("/api/data");
    const data = await response.json();
    setData(data);
  } catch (error) {
    setError("Failed to fetch data");
  }
}
```

- Provide fallback UI for error scenarios. Fallback UI ensures the app remains usable even when some components fail, reducing frustration and downtime.

```jsx
function ErrorFallback() {
  return <div>Unable to load component. Please refresh the page.</div>;
}

<React.Suspense fallback={<ErrorFallback />}>
  <LazyComponent />
</React.Suspense>;
```

- Log errors to console DevTool for debugging and provide meaningful error messages. Logging errors enables efficient debugging and monitoring, while meaningful error messages inform users about issues and guide them to possible solutions, improving reliability and user experience.

```jsx
try {
  // some operation...
} catch (e: any) {
  console.error("Error occurred in fetchUser:", error);
}
```

### Forms and Validation

- Utilize controlled components for all form inputs. Controlled components provide explicit control over form state, enabling validation, error handling, and predictable behavior.

```jsx
import { Input } from "@radix-ui/react-input";
import { useState } from "react";
const [value, setValue] = useState("");
<Input value={value} onChange={(e) => setValue(e.target.value)} />;
```

- Implement schema-based form validation using `zod`. Consistent validation ensures reliable user input handling and reduces bugs and security risks.

```jsx
import { useForm } from "react-hook-form";
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";
import { Input } from "@radix-ui/react-input";

const schema = z.object({
  email: z.string().email(),
});

type FormValues = z.infer<typeof schema>;

export function EmailForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm <
  FormValues >
  {
    resolver: zodResolver(schema),
  };
  const onSubmit = async (data: FormValues) => {
    // submit logic
  };
  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <label htmlFor="email" className="block">
        Email
      </label>
      <Input
        id="email"
        {...register("email")}
        type="email"
        aria-required="true"
        aria-invalid={!!errors.email}
        aria-describedby="email-error"
      />
      {errors.email && (
        <span id="email-error" role="alert" className="text-red-500">
          {errors.email.message}
        </span>
      )}
      <button type="submit" disabled={isSubmitting} className="btn-primary">
        Submit
      </button>
      {isSubmitting && <span>Submitting...</span>}
    </form>
  );
}
```

- Utilize debounced validation for better user experience, integrate debounce logic with `react-hook-form` and `useCallback/useRef` for stable debounce. Debounced validation prevents excessive validation calls, reduces UI lag, and improves responsiveness for users typing in form fields.

```jsx
import { useForm } from "react-hook-form";
import { useRef, useCallback } from "react";
import { Input } from "@radix-ui/react-input";

function useDebouncedCallback(
  callback: (...args: any[]) => void,
  delay: number
) {
  const timeoutRef = useRef<number | undefined>();
  return useCallback(
    (...args: any[]) => {
      if (timeoutRef.current) window.clearTimeout(timeoutRef.current);
      timeoutRef.current = window.setTimeout(() => callback(...args), delay);
    },
    [callback, delay]
  );
}

export function DebouncedEmailForm() {
  const {
    register,
    trigger,
    formState: { errors },
  } = useForm();
  const debouncedValidate = useDebouncedCallback(() => trigger("email"), 400);
  return (
    <form>
      <Input
        {...register("email")}
        onChange={debouncedValidate}
        aria-invalid={!!errors.email}
      />
      {errors.email && <span role="alert">{errors.email.message}</span>}
    </form>
  );
}
```

- Handle file uploads and complex form scenarios using controlled components and consistent validation. Controlled components and consistent validation ensure robust handling of file uploads and complex forms, improving reliability, accessibility, and user experience.

```jsx
import { useForm } from "react-hook-form";
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";
import { Input } from "@radix-ui/react-input";

const schema = z.object({
  file: z.any(),
  description: z.string().min(1, "Description is required"),
});
type FormData = z.infer<typeof schema>;

export function FileUploadForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm <
  FormData >
  {
    resolver: zodResolver(schema),
  };
  const onSubmit = (data: FormData) => {
    const file = data.file[0];
    // handle file upload logic
  };
  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <label htmlFor="file" className="block">
        Upload File
      </label>
      <Input
        id="file"
        type="file"
        {...register("file", { required: "File is required" })}
        aria-describedby="file-error"
      />
      {errors.file && (
        <span id="file-error" role="alert" className="text-red-500">
          {errors.file.message}
        </span>
      )}
      <label htmlFor="description" className="block">
        Description
      </label>
      <Input
        id="description"
        {...register("description", { required: "Description is required" })}
        aria-describedby="desc-error"
      />
      {errors.description && (
        <span id="desc-error" role="alert" className="text-red-500">
          {errors.description.message}
        </span>
      )}
      <button type="submit" disabled={isSubmitting} className="btn-primary">
        Submit
      </button>
      {isSubmitting && <span>Uploading...</span>}
    </form>
  );
}
```

- Manage form submission state explicitly, including loading and error states. Explicit submission state improves user experience and enables robust error handling and feedback.

```jsx
import { useState } from "react";
const [loading, setLoading] = useState(false);
const [error, setError] = (useState < string) | (null > null);
async function handleSubmit(e: React.FormEvent) {
  e.preventDefault();
  setLoading(true);
  setError(null);
  try {
    await submitForm(values);
  } catch (err) {
    setError("Submission failed");
  } finally {
    setLoading(false);
  }
}
```

- Display loading indicators and error messages during form submission. Feedback during submission keeps users informed and improves accessibility and usability.

```jsx
<form onSubmit={handleSubmit}>
  <Input
    value={values.email}
    onChange={(e) => setValues({ ...values, email: e.target.value })}
  />
  {loading && <span>Submitting...</span>}
  {error && <span role="alert">{error}</span>}
  <button type="submit" disabled={loading}>
    Submit
  </button>
</form>
```

- Ensure all forms are accessible, including proper labeling, keyboard navigation, and ARIA attributes. Accessible forms are usable by all users, including those with disabilities, and comply with legal requirements.

```jsx
<form>
  <label htmlFor="email" className="block">
    Email
  </label>
  <Input
    id="email"
    {...register("email")}
    type="email"
    aria-required="true"
    aria-invalid={!!errors.email}
    aria-describedby="email-error"
    className="focus:ring-2 focus:ring-blue-500"
  />
  {errors.email && (
    <span id="email-error" role="alert" className="text-red-500">
      {errors.email.message}
    </span>
  )}
  <button type="submit" className="btn-primary">
    Submit
  </button>
</form>
```

### Routing

- Utilize `react-router-dom` library for client-side routing.

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
      </Routes>
    </BrowserRouter>
  );
}
```

- Implement nested routes and route protection. Nested routes and protection allow for modular navigation structures and secure access control, improving user experience and security.

```jsx
import { Routes, Route, Outlet, Navigate } from "react-router-dom";

function ProtectedRoute({ isAuthenticated }: { isAuthenticated: boolean }) {
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" />;
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route
          path="dashboard"
          element={<ProtectedRoute isAuthenticated={isLoggedIn} />}
        >
          <Route index element={<Dashboard />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Route>
      <Route path="/login" element={<Login />} />
    </Routes>
  );
}
```

- Implement proper handling of route parameters and query strings.

```jsx
import { useParams, useSearchParams } from "react-router-dom";

function UserProfile() {
  const { userId } = useParams();
  const [searchParams] = useSearchParams();
  const tab = searchParams.get("tab");
  return (
    <div>
      User ID: {userId}, Tab: {tab}
    </div>
  );
}
```

- Implement lazy loading for route-based code splitting. Lazy loading reduces initial bundle size and improves load times by only loading code when needed.

```jsx
import { Suspense, lazy } from "react";

const Settings = lazy(() => import("./Settings"));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Routes>
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

- Implement proper navigation patterns and back button handling. Proper navigation and back button handling ensure intuitive user experience and prevent navigation errors.

```jsx
import { useNavigate } from "react-router-dom";

function BackButton() {
  const navigate = useNavigate();
  return <button onClick={() => navigate(-1)}>Go Back</button>;
}
```

- Implement breadcrumbs and navigation state management. Breadcrumbs and navigation state management improve usability and help users understand their location within the app.

```jsx
import { Link, useLocation } from "react-router-dom";

function Breadcrumbs() {
  const location = useLocation();
  const paths = location.pathname.split("/").filter(Boolean);
  return (
    <nav aria-label="breadcrumb">
      <ol>
        <li>
          <Link to="/">Home</Link>
        </li>
        {paths.map((segment, idx) => {
          const url = "/" + paths.slice(0, idx + 1).join("/");
          return (
            <li key={url}>
              <Link to={url}>{segment}</Link>
            </li>
          );
        })}
      </ol>
    </nav>
  );
}
```

### Testing

- Always write unit tests for every component.

```jsx
import { render, screen } from "@testing-library/react";
import Button from "./Button";

test("renders button with label", () => {
  render(<Button label="Click me" onClick={() => {}} />);
  expect(screen.getByText("Click me")).toBeInTheDocument();
});
```

- Always test component behavior, not implementation details. Testing behavior ensures components work as intended for users, making tests robust against refactoring.

```jsx
import { render, screen, fireEvent } from "@testing-library/react";
import Toggle from "./Toggle";

test("toggles state on button click", () => {
  render(<Toggle />);
  const button = screen.getByRole("button");
  fireEvent.click(button);
  expect(button).toHaveTextContent("On");
});
```

- Utilize Jest for test runner and assertion library.

```ts
// jest.config.js
module.exports = {
  testEnvironment: "jsdom",
  setupFilesAfterEnv: ["@testing-library/jest-dom/extend-expect"],
};

// Example test file
import { render, screen } from "@testing-library/react";
import Header from "./Header";

test("renders header title", () => {
  render(<Header title="Dashboard" />);
  expect(screen.getByText("Dashboard")).toBeInTheDocument();
});
```

- Implement integration tests for complex flows involving multiple components. Integration tests verify that components work together as intended, catching issues missed by unit tests.

```jsx
import { render, screen, fireEvent } from "@testing-library/react";
import LoginForm from "./LoginForm";

test("submits login form and displays welcome message", async () => {
  render(<LoginForm />);
  fireEvent.change(screen.getByLabelText(/email/i), {
    target: { value: "user@example.com" },
  });
  fireEvent.change(screen.getByLabelText(/password/i), {
    target: { value: "password123" },
  });
  fireEvent.click(screen.getByRole("button", { name: /login/i }));
  expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
});
```

- Utilize Playwright to write end-to-end test cases for critical user journeys. End-to-end tests validate complete workflows, ensuring the application functions correctly from the user's perspective.

```jsx
import { test, expect } from "@playwright/react";
import App from "./App";

test("user can sign up, log in, and access dashboard", async ({ page }) => {
  await page.goto("/");

  // Sign up
  await page.fill('input[name="email"]', "newuser@example.com");
  await page.fill('input[name="password"]', "password123");
  await page.click('button[type="submit"]');
  await expect(page.getByText("Welcome, newuser@example.com")).toBeVisible();

  // Log out
  await page.click("button#logout");
  await expect(page.getByText("You have been logged out")).toBeVisible();

  // Log in
  await page.fill('input[name="email"]', "newuser@example.com");
  await page.fill('input[name="password"]', "password123");
  await page.click('button[type="submit"]');
  await expect(
    page.getByText("Welcome back, newuser@example.com")
  ).toBeVisible();

  // Access dashboard
  await page.click('a[href="/dashboard"]');
  await expect(page.getByText("Dashboard")).toBeVisible();
});
```

- Mock external dependencies and API calls. Mocking isolates tests from external systems, ensuring reliability and speed, and allowing for edge case testing.

```jsx
import { render, screen, waitFor } from "@testing-library/react";
import UserList from "./UserList";

global.fetch = jest.fn(() =>
  Promise.resolve({
    json: () => Promise.resolve([{ id: 1, name: "Alice" }]),
  })
) as jest.Mock;

test("renders user list from API", async () => {
  render(<UserList />);
  expect(await screen.findByText("Alice")).toBeInTheDocument();
});
```

- Test all user interactions, including clicks, form submissions, and keyboard events. Testing interactions ensures the UI behaves as expected and is accessible to all users.

```jsx
import { render, screen, fireEvent } from "@testing-library/react";
import Counter from "./Counter";

test("increments counter on button click", () => {
  render(<Counter />);
  fireEvent.click(screen.getByText("Increment"));
  expect(screen.getByText("Count: 1")).toBeInTheDocument();
});
```

- Test error scenarios and edge cases. Testing error scenarios ensures the app handles failures gracefully and remains robust.

```jsx
import { render, screen } from "@testing-library/react";
import UserProfile from "./UserProfile";

test("shows fallback when user is null", () => {
  render(<UserProfile user={null} />);
  expect(screen.getByText("No user data available.")).toBeInTheDocument();
});
```

- Test accessibility features and keyboard navigation. Testing accessibility and keyboard navigation ensures the app is usable by all users, including those with disabilities.

```jsx
import { render, screen, fireEvent } from "@testing-library/react";
import Modal from "./Modal";

test("focuses input when modal opens", () => {
  render(<Modal open={true} />);
  const input = screen.getByPlaceholderText("Enter name");
  expect(document.activeElement).toBe(input);
});

test("closes modal on Escape key", () => {
  render(<Modal open={true} onClose={jest.fn()} />);
  fireEvent.keyDown(screen.getByRole("dialog"), { key: "Escape" });
  expect(screen.queryByRole("dialog")).not.toBeInTheDocument();
});
```

### Security

- Always validate user inputs and also sanitize them with schema validation library `zod` or type guards to prevent XSS attacks. Sanitizing user inputs prevents malicious code injection, protecting users and application data.

```jsx
import DOMPurify from "dompurify";

function SafeHtml({ html }: { html: string }) {
  const cleanHtml = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: cleanHtml }} />;
}

// manual escape data
function UserName({ name }: { name: string }) {
  const safeName = name.replace(/[<>&"'`]/g, "");
  return <span>{safeName}</span>;
}
```

- Avoid dynamic code execution and untrusted template rendering. Encode untrusted content before rendering HTML, use framework escaping or trusted types.

```jsx
// Unsafe (DO NOT USE) - demonstrates what to avoid:
function UnsafeRenderer({ tpl, ctx }: { tpl: string; ctx: Record<string, any> }) {
  // BAD: constructing and executing code from user input (template)
  // This is vulnerable to XSS / remote code execution.
  // DO NOT use eval / new Function on untrusted strings.
  // eslint-disable-next-line no-new-func
  const render = new Function(
    "ctx",
    `with (ctx) { return \`${tpl}\`; }` // template literal executed as code
  ) as (c: Record<string, any>) => string;

  const html = render(ctx); // attacker-controlled tpl => RCE/XSS
  return <div dangerouslySetInnerHTML={{ __html: html }} />; // unsafe
}

// Safe 1: Render untrusted text using React's automatic escaping
function SafeText({ text }: { text: string }) {
  // React will escape strings when rendered in JSX, preventing HTML injection.
  return <div>{text}</div>;
}

// Safe 2: Render sanitized HTML using DOMPurify + Trusted Types (if available)
import DOMPurify from "dompurify";

type TrustedHTMLLike = string | TrustedHTML;

function SafeHtml({ html }: { html: string }) {
  // Always sanitize incoming HTML
  // Prefer a Trusted Types policy when running in browsers that support it.
  let trusted: TrustedHTMLLike;

  if (typeof window !== "undefined" && (window as any).trustedTypes) {
    // Create or reuse a trusted types policy that sanitizes via DOMPurify
    // The policy must be created once; safe to guard with a symbol on window.
    const POL_NAME = "__app_sanitize_policy__";
    if (!(window as any)[POL_NAME]) {
      (window as any)[POL_NAME] = (window as any).trustedTypes.createPolicy("app-sanitize", {
        createHTML: (s: string) =>
          // DOMPurify can return a TrustedHTML when RETURN_TRUSTED_TYPE is used.
          DOMPurify.sanitize(s, { RETURN_TRUSTED_TYPE: true }) as TrustedHTML,
      });
    }
    trusted = (window as any)[POL_NAME].createHTML(html) as TrustedHTML;
  } else {
    // Fallback: sanitize to a plain string
    trusted = DOMPurify.sanitize(html);
  }

  // Only use sanitized/trusted content with dangerouslySetInnerHTML
  return <div dangerouslySetInnerHTML={{ __html: trusted as string }} />;
}

// Safe 3: Convert markup to safe React elements instead of using innerHTML
import { parse } from "html-react-parser"; // lightweight parser that outputs React nodes

function SafeHtmlToReact({ html }: { html: string }) {
  // sanitize first, then parse into React nodes
  const clean = DOMPurify.sanitize(html);
  return <div>{parse(clean)}</div>;
}


<SafeText text={'<img src=x onerror=alert(1)> raw text will be escaped'} />
<SafeHtml html={userProvidedHtml} />
<SafeHtmlToReact html={userProvidedHtml} />

<!-- Never:
- use UnsafeRenderer or any eval/new Function on user templates
- mark untrusted strings as HTML without sanitizing and/or using Trusted Types -->
```

- Utilize HTTPS for all external API calls. HTTPS encrypts data in transit, protecting sensitive information and ensuring secure communication.

```jsx
fetch("https://api.example.com/data");
```

- Implement proper authentication and authorization patterns. Proper authentication and authorization restrict access to sensitive resources, ensuring user privacy and application security.

```jsx
// Protect routes using authentication context
import { Navigate, Outlet } from "react-router-dom";

function ProtectedRoute({ isAuthenticated }: { isAuthenticated: boolean }) {
  return isAuthenticated ? <Outlet /> : <Navigate to="/login" />;
}

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route
          path="dashboard"
          element={<ProtectedRoute isAuthenticated={isLoggedIn} />}
        >
          <Route index element={<Dashboard />} />
          <Route path="settings" element={<Settings />} />
        </Route>
      </Route>
      <Route path="/login" element={<Login />} />
    </Routes>
  );
}
```

- DO NOT store sensitive data in `localStorage` or `sessionStorage`. They are vulnerable to XSS attacks and should be used to store non-sentitive data.

```jsx
// Bad: Store passwords or JWTs

// Good: Store only UI preference setting, ...
localStorage.setItem("theme", "dark");
```

### Accessibility

- Utilize semantic HTML elements for all UI components. Semantic HTML improves accessibility, SEO, and maintainability by conveying meaning to browsers and assistive technologies.

```jsx
// Good: Semantic button for form submission
<button type="submit">Submit</button>

// Good: Using <nav> for navigation
<nav>
  <ul>
  <li><a href="/dashboard">Dashboard</a></li>
  <li><a href="/settings">Settings</a></li>
  </ul>
</nav>

// Bad: Using <div> for interactive elements
<div onClick={handleClick}>Submit</div>
```

- Implement appropriate ARIA attributes for custom components and widgets. ARIA attributes make custom components accessible to users with disabilities, ensuring compliance and usability.

```jsx
// Good: Custom toggle switch with ARIA
<button
  role="switch"
  aria-checked={isOn}
  aria-label="Enable notifications"
  onClick={toggle}
>
  {isOn ? "On" : "Off"}
</button>;

// Good: Accessible dialog using @radix-ui/react-dialog
import { Dialog, DialogTrigger, DialogContent } from "@radix-ui/react-dialog";

<Dialog>
  <DialogTrigger asChild>
    <button>Open Dialog</button>
  </DialogTrigger>
  <DialogContent aria-label="User Settings">
    <h2>User Settings</h2>
    {/* ... */}
  </DialogContent>
</Dialog>;
```

- Ensure all interactive elements are accessible via keyboard navigation. Keyboard accessibility is essential for users who cannot utilize a mouse, improving inclusivity and usability.

```jsx
// Good: All buttons and links are focusable and actionable via keyboard
<button onClick={handleSubmit}>Submit</button>
<a href="/profile">Profile</a>

// Good: Custom menu using @radix-ui/react-dropdown-menu
import { DropdownMenu, DropdownMenuTrigger, DropdownMenuContent, DropdownMenuItem } from '@radix-ui/react-dropdown-menu';

<DropdownMenu>
  <DropdownMenuTrigger asChild>
  <button>Options</button>
  </DropdownMenuTrigger>
  <DropdownMenuContent>
  <DropdownMenuItem onSelect={handleEdit}>Edit</DropdownMenuItem>
  <DropdownMenuItem onSelect={handleDelete}>Delete</DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>
```

- Test components with screen readers to verify accessibility. Screen reader testing ensures that visually impaired users can interact with the application effectively.

```jsx
// Good: Utilize role and aria attributes for screen reader support
<div role="alert" aria-live="assertive">
  {errorMessage}
</div>
```

- Manage focus explicitly for modal dialogs, popups, and dynamic content. Explicit focus management prevents users from getting lost in the UI and improves accessibility for dynamic content.

```jsx
// Good: Focus management with @radix-ui/react-dialog
import { Dialog, DialogTrigger, DialogContent } from "@radix-ui/react-dialog";

<Dialog>
  <DialogTrigger asChild>
    <button>Open Modal</button>
  </DialogTrigger>
  <DialogContent>
    <input autoFocus type="text" placeholder="Enter name" />
  </DialogContent>
</Dialog>;
```

- Provide descriptive alt text for images and descriptive text for icons. Descriptive alt text ensures images are accessible to screen readers and improves SEO and user experience.

```jsx
// Good: Descriptive alt text for company logo
<img src="/assets/logo.png" alt="ApplyEdge company logo" />

// Good: User avatar with alt text
<img src={user.avatarUrl} alt={`Avatar of ${user.name}`} />
```

- Implement proper color contrast ratios for all text and interactive elements to ensure readability and accessibility. Adequate color contrast ensures text and UI elements are readable by all users, including those with visual impairments.

```jsx
// Good: Sufficient contrast between text and background
<div style={{ background: "#ffffff", color: "#222222" }}>Readable Text</div>

// Bad: Insufficient contrast (light gray on white)
<div style={{ background: "#ffffff", color: "#cccccc" }}>Hard to read</div>

// Good: Utilize accessible color palettes and test with tools like WebAIM Contrast Checker or axe DevTools.
// WCAG AA requires a contrast ratio of at least 4.5:1 for normal text and 3:1 for large text.
```

- Test components with screen readers and accessibility tools.

```jsx
// Good: Utilize role and aria attributes for screen reader support
<div role="alert" aria-live="assertive">
  {errorMessage}
</div>

// Test with screen readers (NVDA, VoiceOver, JAWS) and tools like axe DevTools or Lighthouse to verify that alerts, dialogs, and navigation are announced correctly and interactive elements are accessible.
```

### Code Organization

- Group related components, hooks, and utilities together in the same directory.
  Reasoning: Keeping related code improves maintainability, discoverability, and reduces cognitive load for developers.

- Utilize consistent and descriptive file naming conventions for all files.
  Reasoning: Consistent naming makes it easier to locate files, understand their purpose, and reduces onboarding time for new team members.

- Organize directories by feature or domain, not by type.
  Reasoning: Feature-based organization aligns code structure with business logic, making scaling and refactoring easier as the application grows.

- Keep styles (CSS, SCSS, styled-components, etc.) in the same directory as their related components.
  Reasoning: Co-locating styles with components ensures that UI logic and presentation are tightly coupled, reducing the risk of orphaned or unused styles.

- Utilize named/explicit imports and exports for all modules.
  Reasoning: Named imports and exports improve code clarity, enable better tooling support, and prevent accidental import of unused code.

- Document complex component logic in a dedicated README or code documentation file.
  Reasoning: Documentation helps future maintainers understand design decisions, edge cases, and integration points, reducing technical debt.

## Implementation Process

1. Plan component architecture and data flow.
2. Set up project structure with proper folder organization.
3. Define TypeScript interfaces and types.
4. Implement core components with proper styling.
5. Add state management and data fetching logic.
6. Implement routing and navigation.
7. Add form handling and validation.
8. Implement error handling and loading states.
9. Add testing coverage for components and functionality.
10. Optimize performance and bundle size.
11. Ensure accessibility compliance.
12. Add documentation and code comments.

## Additional Guidelines

- Follow React's naming conventions (`PascalCase` for components, `camelCase` for functions)
- Utilize meaningful commit messages and maintain clean git history
- Implement proper code splitting and lazy loading strategies
- Document complex components and custom hooks with JSDoc
- Utilize ESLint and Prettier for consistent code formatting
- Keep dependencies up to date and audit for security vulnerabilities
- Implement proper environment configuration for different deployment stages
- Utilize React Developer Tools for debugging and performance analysis

## Patterns

- Higher-Order Components (HOCs) for cross-cutting concerns
- Render props pattern for component composition
- Compound components for related functionality
- Provider pattern for context-based state sharing
- Container/Presentational component separation
- Custom hooks for reusable logic extraction

---

<!-- End of React Best Practices instructions -->
