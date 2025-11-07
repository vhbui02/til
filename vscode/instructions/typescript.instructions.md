---
description: "Updated: 2025-10-11. TypeScript coding standards and best practices for modern web development. Cre: https://github.com/PatrickJS/awesome-cursorrules/blob/main/rules-new/typescript.mdc "
applyTo: "**/*.ts, **/*.tsx, **/*.d.ts"
---

# TypeScript Best Practices

Instructions for building high-quality TypeScript applications with modern patterns, type safety, and best practices following the official TypeScript documentation at https://www.typescriptlang.org/docs/.

## Module System

- Use pure ES6 modules syntax, DO NOT write `CommonJS` module syntax: `require`, `module.exports`, ... or its helpers.

## Type System

### Type Definitions

- Prefer `type` over `interface`

```js
// Not preferred
interface User {
  id: number;
  name: string;
  email: string;
}

// Good
type User = {
  id: number,
  name: string,
  email: string,
};
```

- Use type for Unions, Intersections, and mapped types.

```js
// Good: Type for union
type Status = "loading" | "success" | "error";

// Good: Type for intersection
type UserWithPermissions = User & { permissions: string[] };

// Good: Type for mapped types
type OptionalUser = {
  [K in keyof User]?: User[K];
};
```

- DO NOT use `any`, prefer `unknown` for unknown types.

```js
// Bad: Using any
function processData(data: any) {
  return data.name; // No type checking
}

// Good: Using unknown with type guards
function processData(data: unknown) {
  if (typeof data === "object" && data !== null && "name" in data) {
    return (data as { name: string }).name;
  }
  throw new Error("Invalid data");
}
```

- Divide TypeScript configurations into two files and use strict TypeScript configurations.

```json
// tsconfig.json
{
  // ...
  "references": [
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.node.json"
    }
  ]
  // ...
}
```

```json
// tsconfig.app.json
{
  // Cre: https://khalilstemmler.com/blogs/typescript/node-starter-project/
  // Cre: https://github.com/alan2207/bulletproof-react/blob/master/apps/react-vite/tsconfig.json
  "compilerOptions": {
    // "rootDir": "",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },
    // "composite": true,
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"], // add ambient types to use features from different ECMAScript ver
    "module": "ESNext", // most stable, for code that will be bundled
    /* ====================================================================== */
    /* Bundler mode */
    /* ====================================================================== */
    "moduleResolution": "bundler", // how TS lookup a file
    "allowImportingTsExtensions": true, // import can include TS file ext
    "isolatedModules": true, // Ensure transpiling process is success when using single-file transpiler
    "moduleDetection": "force", // How TS determine a file is a module
    "noEmit": true, // no JS src code, src maps or declaration, Babel and swc do the jobs that handle convert TS file to a file running in JS env
    "jsx": "react-jsx",
    "allowJs": true, // allow JS file to be part of TypeScript project
    "esModuleInterop": true, // enhance CommonJS and ESM interoperability
    "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
    /* ====================================================================== */
    /* Linting */
    /* ====================================================================== */
    "strict": true,
    "alwaysStrict": true,
    "noImplicitAny": false, // some destructurizing props don't need to be known about
    "noImplicitThis": true,
    "noUnusedLocals": false, // NOTE: devs tend to add future works so it's best to leave this off
    "noUnusedParameters": false, // NOTE: devs tend to add future works so it's best to leave this off
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true, // err detection in switch statement
    /* ====================================================================== */
    /* Misc */
    /* ====================================================================== */
    "skipLibCheck": true, // prevent type checking for 'd.ts', therefore exclude node_modules when type checking whole project
    // "declaration": true, // generate `.d.ts` for every JS/TS file inside projects, provide IntelliSense
    "resolveJsonModule": true, // Enable import JSON directly
    "verbatimModuleSyntax": true, // enforce `import type`
    "forceConsistentCasingInFileNames": true, // enable case-sensitive import
    "useDefineForClassFields": true,
    "types": [
      "vite/client"
      // "vitest/globals"
    ],
    "declaration": true,
    "sourceMap": true,
    "allowSyntheticDefaultImports": true // allow `import x from y even when a module doesn't have `default` export
  },
  // rel path to dir storing implementation files
  // "files": []
  "include": ["src"],
  "exclude": ["node_modules", "tmp", ".tmp", "temp", ".temp"]
}
```

- Leverage built-in utility types for common type transformations and reduce boilerplate code.

```js
// Good: Using utility types
type PartialUser = Partial<User>;
type RequiredUser = Required<User>;
type PickUser = Pick<User, "id" | "name">;
type OmitUser = Omit<User, "password">;
type RecordUser = Record<string, User>;
```

- Use Generics for reusable type patterns, enable type-safe and practice DRY that works with multiple data types.

```js
// Good: Generic function
function identity<T>(arg: T): T {
  return arg;
}

// Good: Generic interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// Good: Generic class
class Container<T> {
  private value: T;

  constructor(value: T) {
    this.value = value;
  }

  getValue(): T {
    return this.value;
  }
}
```

### Type Guards and Narrowing

- Use type guards for runtime type checking, enable safe type narrowing and improve type safety at runtime.

```js
// Good: Type guard function
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === "object" &&
    obj !== null &&
    "id" in obj &&
    "name" in obj &&
    "email" in obj
  );
}

// Good: Using type guard
function processUser(data: unknown) {
  if (isUser(data)) {
    // TypeScript knows data is User here
    console.log(data.name);
  }
}
```

- Implement proper null checking, prevent runtime errors and improves code reliability.

```js
// Good: Null checking
function getUserName(user: User | null): string {
  if (user === null) {
    return "Unknown User";
  }
  return user.name;
}

// Good: Optional chaining
function getUserEmail(user?: User): string | undefined {
  return user?.email;
}

// Good: Nullish coalescing
function getDisplayName(user: User | null): string {
  return user?.name ?? "Anonymous";
}
```

## Naming Conventions

- Use `kebab-case` for file name.

```txt
service
├── user-service.ts
├── data-service.ts
├── ...
```

- Use `PascalCase` for types and interfaces. PascalCase distinguishes types from variables and follows TypeScript conventions.

```js
// Good: PascalCase for types
interface UserProfile {}
type ApiResponse<T> = {};
enum UserRole {}
```

- Use `camelCase` for variables and functions.

```js
// Good: camelCase for variables and functions
const userName = "John";
function getUserData() {}
const apiResponse = {};
```

- Use `UPPER_CASE` for constants.

```js
// Good: UPPER_CASE for constants
const API_BASE_URL = "https://api.example.com";
const MAX_RETRY_ATTEMPTS = 3;
const DEFAULT_TIMEOUT = 5000;
```

- Use descriptive names with auxiliary verbs for boolean variables. Making boolean variables self-documenting is important.

```js
// Bad: Unclear boolean names
const loading = true;
const error = false;

// Good: Descriptive boolean names
const isLoading = true;
const hasError = false;
const isAuthenticated = true;
const canEdit = false;
```

- Add `-Props` suffix to React Component properties.

```js
// Good: Props prefix for React components
type ButtonProps = {
  label: string,
  onClick: () => void,
  disabled?: boolean,
};

function Button({ label, onClick, disabled }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {label}
    </button>
  );
}
```

## Code Organization

- Keep type definitions close to where they are used. If a type is referenced by multiple modules, move it to a dedicated type file inside the `types/` directory.

```js
// Good: Definition near references
type UserCardProps = {
  user: User,
  onEdit: (user: User) => void,
};

function UserCard({ user, onEdit }: UserCardProps) {
  return <div onClick={() => onEdit(user)}>{user.name}</div>;
}
```

or

```js
// src/types/user.ts
export type User = {
  id: string,
  name: string,
};

// src/auth/login.ts
import { User } from "../types/user";

// src/auth/register.ts
import { User } from "../types/user";
```

- Export Type and Interface from dedicated type files when shared. Centralized type files improve reusability and reduce duplication across the codebase.

```js
// src/types/user.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

export type UserRole = "admin" | "user" | "guest";

// src/components/UserProfile.tsx
import { User, UserRole } from "../types/user";
```

- Use barrel exports (`index.ts`) for organizing exports. Barrel exports provide clean import paths and better organization of related types.

```js
// src/types/index.ts
export * from "./user";
export * from "./api";
export * from "./common";

// Usage
import { User, ApiResponse, Status } from "../types";
```

- Place shared types in a `types` directory.

```
src/
├── types/
│   ├── user.ts
│   ├── api.ts
│   ├── common.ts
│   └── index.ts
├── components/
├── utils/
├── ...
```

- Co-locate React Component props with their components, keep related code together and improves maintainability.

```js
// components/Button.tsx
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
}

export function Button({ label, onClick, variant = "primary" }: ButtonProps) {
  return (
    <button className={variant} onClick={onClick}>
      {label}
    </button>
  );
}
```

## Functions

- Use explicit return types for public functions. Explicit return types improve code clarity, enable better tooling support, and catch type errors early.

```js
// Good: Explicit return types
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}
```

- Use arrow functions for callbacks and methods. Arrow functions provide lexical `this` binding and are more concise for callbacks.

```js
// Good: Arrow functions for callbacks
const users = data.map((user) => ({
  id: user.id,
  name: user.name,
}));

// Good: Arrow functions for methods
class UserService {
  private users: User[] = [];

  addUser = (user: User): void => {
    this.users.push(user);
  };
}
```

- Implement proper error handling with custom error types. Custom error types provide better error context and enable type-safe error handling.

```js
// Good: Custom error types
class ValidationError extends Error {
  constructor(message: string, public field: string) {
    super(message);
    this.name = "ValidationError";
  }
}

class ApiError extends Error {
  constructor(message: string, public statusCode: number) {
    super(message);
    this.name = "ApiError";
  }
}

// Usage
function validateUser(user: unknown): User {
  if (!isUser(user)) {
    throw new ValidationError("Invalid user data", "user");
  }
  return user;
}
```

- Use function overloads for complex type scenarios. Function overloads provide precise typing for functions with different parameter combinations.

```js
// Good: Function overloads
function createUser(name: string): User;
function createUser(name: string, email: string): User;
function createUser(name: string, email?: string): User {
  return {
    id: Date.now(),
    name,
    email: email ?? `${name.toLowerCase()}@example.com`,
  };
}
```

- Prefer `async/await` over `Promises`. Wrap `awaits` inside try/catch with structured errors.

```js
// Not preferred: Promise chains
function fetchUserData(id: number): Promise<User> {
  return fetch(`/api/users/${id}`)
    .then((response) => {
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      return response.json();
    })
    .catch((error) => {
      console.error("Failed to fetch user:", error);
      throw error;
    });
}

// Good: async/await
async function fetchUserData(id: number): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return await response.json();
  } catch (e: any) {
    console.error("Failed to fetch user:", e);
    const errorCode = e.data?.code;
    const i18nKey = `errorCodes.${errorCode}`;
    if (errorCode && t(i18nKey) !== i18nKey) {
      console.error(t(i18nKey));
    } else {
      console.error(`Failed to fetch user. Error: ${errorCode || "SERVER_CONNECTION_ERROR"}`);
    }
  }
}
```

## Advanced Type Patterns

- Use `readonly` for immutable properties, prevent accidental mutations and makes intent clear.

```js
// Good: readonly properties
interface Config {
  readonly apiUrl: string;
  readonly timeout: number;
}

// Good: readonly arrays
function processItems(items: readonly Item[]): void {
  // items.push(newItem); // Error: Cannot assign to 'push' because it is a read-only property
}
```

- Leverage Discriminated Union to handle type-safe of different states or variants.

```js
// Good: Discriminated union
type ApiState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success", data: T }
  | { status: "error", error: string };

function renderApiState<T>(state: ApiState<T>): React.ReactNode {
  switch (state.status) {
    case "idle":
      return <div>Ready to fetch</div>;
    case "loading":
      return <div>Loading...</div>;
    case "success":
      return <div>Data: {JSON.stringify(state.data)}</div>;
    case "error":
      return <div>Error: {state.error}</div>;
  }
}
```

- DO NOT USE type assertions, unless necessary. Type assertions bypass type checking and can lead to runtime errors.

```js
// Bad: Unnecessary type assertion
const user = {} as User; // This creates an empty object, not a valid User

// Good: Proper type checking
function createUser(data: unknown): User {
  if (isUser(data)) {
    return data;
  }
  throw new Error("Invalid user data");
}

// Good: Necessary type assertion with validation
const element = document.getElementById("my-element") as HTMLInputElement;
if (element) {
  element.value = "new value";
}
```

## Error Handling

- Create custom error types for domain-specific errors. Custom error types provide better error context and enable type-safe error handling.

```js
// Good: Domain-specific error types
class UserNotFoundError extends Error {
  constructor(public userId: number) {
    super(`User with ID ${userId} not found`);
    this.name = "UserNotFoundError";
  }
}

class InsufficientPermissionsError extends Error {
  constructor(public requiredPermission: string) {
    super(`Insufficient permissions. Required: ${requiredPermission}`);
    this.name = "InsufficientPermissionsError";
  }
}

// Usage
async function getUser(id: number): Promise<User> {
  const user = await fetchUserFromApi(id);
  if (!user) {
    throw new UserNotFoundError(id);
  }
  return user;
}
```

- Use Result types for operations that can fail. Result types provide type-safe error handling without exceptions.

```js
// Good: Result type pattern
type Result<T, E = Error> =
  | { success: true, data: T }
  | { success: false, error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) {
    return { success: false, error: "Division by zero" };
  }
  return { success: true, data: a / b };
}

// Usage
const result = divide(10, 2);
if (result.success) {
  console.log("Result:", result.data);
} else {
  console.error("Error:", result.error);
}
```

- Implement proper error boundaries in React. Error boundaries catch JavaScript errors anywhere in the component tree and display fallback UI.

```js
// Good: Error boundary component
interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends React.Component<
  React.PropsWithChildren<{}>,
  ErrorBoundaryState
> {
  constructor(props: React.PropsWithChildren<{}>) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("Error caught by boundary:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <div>Something went wrong. Please try again.</div>;
    }
    return this.props.children;
  }
}
```

- Use `try-catch` blocks with typed catch clauses. Typed catch clauses provide better error handling and type safety.

```js
// Good: Typed catch clauses
async function handleApiCall(): Promise<void> {
  try {
    const data = await fetchData();
    processData(data);
  } catch (error) {
    if (error instanceof ApiError) {
      console.error("API Error:", error.statusCode, error.message);
    } else if (error instanceof ValidationError) {
      console.error("Validation Error:", error.field, error.message);
    } else {
      console.error("Unknown error:", error);
    }
  }
}
```

- Handle `Promise` rejections properly. Unhandled `Promise` rejections can cause application crashes and poor user experience.

```js
// Good: Proper Promise rejection handling
async function fetchUserData(id: number): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch user data:", error);
    throw error; // Re-throw to let caller handle
  }
}

// Usage with proper error handling
fetchUserData(123)
  .then((user) => console.log("User:", user))
  .catch((error) => console.error("Error fetching user:", error));
```

## Design Patterns

- Use the Builder pattern for complex object creation.

```js
// Good: Builder pattern
class UserBuilder {
  private user: Partial<User> = {};

  setId(id: number): UserBuilder {
    this.user.id = id;
    return this;
  }

  setName(name: string): UserBuilder {
    this.user.name = name;
    return this;
  }

  setEmail(email: string): UserBuilder {
    this.user.email = email;
    return this;
  }

  build(): User {
    if (!this.user.id || !this.user.name || !this.user.email) {
      throw new Error("Missing required fields");
    }
    return this.user as User;
  }
}

// Usage
const user = new UserBuilder()
  .setId(1)
  .setName("John Doe")
  .setEmail("john@example.com")
  .build();
```

- Implement the Repository pattern for data access. The Repository pattern abstracts data access logic and provides a consistent interface.

```js
// Good: Repository pattern
interface UserRepository {
  findById(id: number): Promise<User | null>;
  findAll(): Promise<User[]>;
  save(user: User): Promise<User>;
  delete(id: number): Promise<void>;
}

class ApiUserRepository implements UserRepository {
  async findById(id: number): Promise<User | null> {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) {
      return null;
    }
    return response.json();
  }

  async findAll(): Promise<User[]> {
    const response = await fetch("/api/users");
    return response.json();
  }

  async save(user: User): Promise<User> {
    const response = await fetch("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(user),
    });
    return response.json();
  }

  async delete(id: number): Promise<void> {
    await fetch(`/api/users/${id}`, { method: "DELETE" });
  }
}
```

- Use the Factory pattern for object creation. The Factory pattern centralizes object creation logic and provides flexibility.

```js
// Good: Factory pattern
interface Notification {
  send(message: string): void;
}

class EmailNotification implements Notification {
  send(message: string): void {
    console.log(`Sending email: ${message}`);
  }
}

class SmsNotification implements Notification {
  send(message: string): void {
    console.log(`Sending SMS: ${message}`);
  }
}

class NotificationFactory {
  static create(type: "email" | "sms"): Notification {
    switch (type) {
      case "email":
        return new EmailNotification();
      case "sms":
        return new SmsNotification();
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Usage
const emailNotifier = NotificationFactory.create("email");
emailNotifier.send("Hello World");
```

- Follow the repository's dependency injection.
  Reasoning: Dependency injection improves testability, flexibility, and separation of concerns.

  Example:

  ```js
  // Good: Dependency injection
  interface UserService {
    getUser(id: number): Promise<User>;
  }

  class ApiUserService implements UserService {
    async getUser(id: number): Promise<User> {
      const response = await fetch(`/api/users/${id}`);
      return response.json();
    }
  }

  class UserComponent {
    constructor(private userService: UserService) {}

    async loadUser(id: number): Promise<User> {
      return this.userService.getUser(id);
    }
  }

  // Usage
  const userService = new ApiUserService();
  const userComponent = new UserComponent(userService);
  ```

- Use the Module pattern for encapsulation. The Module pattern provides encapsulation and prevents global namespace pollution.

```js
// Good: Module pattern/IIFE
const UserModule = (() => {
  // Private variables
  let users: User[] = [];

  // Private methods
  function validateUser(user: User): boolean {
    return user.name.length > 0 && user.email.includes("@");
  }

  // Public API
  return {
    addUser(user: User): boolean {
      if (validateUser(user)) {
        users.push(user);
        return true;
      }
      return false;
    },

    getUsers(): User[] {
      return [...users]; // Return copy to prevent external modification
    },

    findUser(id: number): User | undefined {
      return users.find((user) => user.id === id);
    },
  };
})();
```

## Performance and Optimization

- Use `const` assertions for immutable data and should be used most of the time.It provides better type inference and prevent accidental mutations.

```js
// Good: const assertions
const colors = ["red", "green", "blue"] as const;
type Color = (typeof colors)[number]; // 'red' | 'green' | 'blue'

const config = {
  apiUrl: "https://api.example.com",
  timeout: 5000,
} as const;
```

- Use template literal types for string manipulation. Template literal types provide type-safe string operations and better IntelliSense.

```js
// Good: Template literal types
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type ApiEndpoint = `/api/${string}`;
type FullUrl = `https://api.example.com${ApiEndpoint}`;

function makeRequest(method: HttpMethod, endpoint: ApiEndpoint): void {
  const url: FullUrl = `https://api.example.com${endpoint}`;
  // ...
}
```

- Use conditional types for complex type logic. Conditional types enable sophisticated type transformations and better type safety.

```js
// Good: Conditional types
type NonNullable<T> = T extends null | undefined ? never : T;

type ApiResponse<T> = T extends User
  ? { data: User; type: "user" }
  : T extends Product
  ? { data: Product; type: "product" }
  : { data: T; type: "unknown" };

function processResponse<T>(response: ApiResponse<T>): void {
  if (response.type === "user") {
    console.log("User:", response.data.name);
  } else if (response.type === "product") {
    console.log("Product:", response.data.title);
  }
}
```

## Testing and Type Safety

- Write type-safe tests using TypeScript. Type-safe tests catch type errors at compile time and provide better IntelliSense.

```js
// Good: Type-safe tests
import { describe, it, expect } from "vitest";

describe("User validation", () => {
  it("should validate a correct user", () => {
    const user: User = {
      id: 1,
      name: "John Doe",
      email: "john@example.com",
    };

    expect(isUser(user)).toBe(true);
  });

  it("should reject invalid user data", () => {
    const invalidData = { name: "John" }; // Missing required fields

    expect(isUser(invalidData)).toBe(false);
  });
});
```

- Use type predicates in test utilities. Type predicates provide type-safe testing utilities and improve test reliability.

```js
// Good: Type predicates in tests
function assertIsUser(value: unknown): asserts value is User {
  if (!isUser(value)) {
    throw new Error(`Expected User, got ${typeof value}`);
  }
}

it("should process user data correctly", () => {
  const data = fetchUserData();
  assertIsUser(data);

  // TypeScript knows data is User here
  expect(data.name).toBeDefined();
  expect(data.email).toBeDefined();
});
```

## Code Quality and Maintenance

- Always use ESLint with TypeScript-specific rules since it catches common mistakes and enforces consistent code style.

```json
// .eslintrc.json
{
  "extends": [
    "@typescript-eslint/recommended",
    "@typescript-eslint/recommended-requiring-type-checking"
  ],
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn"
  }
}
```

- Always configure Prettier for consistent code formatting, ensures consistent code formatting across the team and reduces formatting-related merge conflicts.

```json
// .prettierrc.json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2
}
```

- Document complex types with `JSDoc` comments. `JSDoc` comments provide better documentation and IntelliSense for complex types.

```js
/**
 * Represents a user in the system
 * @interface User
 */
interface User {
  /** Unique identifier for the user */
  id: number;
  /** Full name of the user */
  name: string;
  /** Email address of the user */
  email: string;
}

/**
 * Creates a new user with the given data
 * @param userData - The user data to create
 * @returns A promise that resolves to the created user
 * @throws {ValidationError} When user data is invalid
 */
async function createUser(userData: Omit<User, "id">): Promise<User> {
  // Implementation
}
```

## Implementation Process

1. Set up TypeScript configuration with strict mode enabled
2. Define core types and interfaces for the domain
3. Implement type-safe utility functions and helpers
4. Create type guards for runtime type checking
5. Implement error handling with custom error types
6. Add comprehensive type definitions for external APIs
7. Write type-safe tests for all functions and components
8. Configure ESLint and Prettier for code quality
9. Document complex types and functions with JSDoc
10. Review and refactor types for better reusability

## General Guidelines

- Prefer readable, explicit solutions over clever shortcuts.
- Extend current abstractions before inventing new ones.
- Use meaningful commit messages and maintain clean git history
- Keep dependencies up to date and audit for security vulnerabilities
- Write comprehensive type definitions for API responses

## Common Patterns

- Type guards for runtime type checking
- Discriminated unions for state management
- Builder pattern for complex object creation
- Repository pattern for data access abstraction
- Factory pattern for object creation
- Dependency injection for better testability
- Module pattern for encapsulation
- Result types for error handling
- Template literal types for string manipulation
- Conditional types for complex type logic

---

<!-- End of TypeScript Best Practices instructions -->
