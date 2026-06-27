# TanStack Query Error Boundary

- `useErrorBoundary: false` (default): error object is captured inside the hook and returned directly to component state (`isError`, `error`, ...) => UI error states are handled manually inside each component.
- `useErrorBoundary: true`: errors are escalated, the hook throws the error during React render phase, crashing the specific component tree and pass the responsibility up to the nearest `<ErrorBoundary>` component.

## Examples

With `useErrorBoundary: false`

```tsx
import { useQuery } from '@tanstack/react-query';

function ProfileComponent() {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ['profile'],
    queryFn: fetchProfile,
    throwOnError: false, // Default behavior
  });

  if (isPending) return <div>Loading...</div>;
  if (isError) return <div>Failed to load profile: {error.message}</div>;

  return <div>Welcome, {data.name}!</div>;
}
```

With `useErrorBoundary: true`

```tsx
import { useQuery } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

function ProfileComponent() {
  const { data } = useQuery({
    queryKey: ['profile'],
    queryFn: fetchProfile,
    throwOnError: true, // Will throw to parent boundary on failure
  });

  // No 'if (isError)' checking required here
  return <div>Welcome, {data.name}!</div>;
}

// Wrapper component
function App() {
  return (
    <ErrorBoundary fallback={<div>Something went wrong at a higher level!</div>}>
      <ProfileComponent />
    </ErrorBoundary>
  );
}
```
