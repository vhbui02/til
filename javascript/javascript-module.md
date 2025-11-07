# JavaScript Module

## Module system

Module system is a mechanism defining how code and dependencies are imported and exported:

### `CommonJS`

a module system implemented by NodeJS's JS Runtime Environment, but not supported by Browser's JS Runtime Environment

```js
// math.js
function add(a, b) {
  return a + b;
}

module.exports = { add };

// app.js
const math = require("./math"); // sync loading
console.log(math.add(2, 3)); // Output: 5
```

### `ES6`

A module system implemented by both Node.js and Browser's JS Runtime Environment

```json
// Nodejs, package.json
{
  "type": "module"
}
```

```html
<!-- Browser, <script> tag -->
<script src="greet.js" type="module"></script>
<script type="module">
  import { greet } from "./greet.js";
  console.log(greet("Browser"));
</script>
```

#### Importing

- **Import entire module as an object:**

  ```js
  import * as utils from "./utils.js";
  // usage: utils.foo, utils.bar
  ```

- **Import named exports:**

  ```js
  import { foo, bar } from "./utils.js";
  ```

- **Import and rename:**

  ```js
  import { foo as myFoo } from "./utils.js";
  ```

- **Import default export:**

  ```js
  import myDefault from "./utils.js";
  ```

- **Import default and named exports together:**

  ```js
  import myDefault, { foo, bar } from "./utils.js";
  ```

- **Import for side effects only:**

  ```js
  import "./setup.js";
  ```

- **Dynamic import (asynchronously):**
  ```js
  const module = await import("./utils.js");
  // usage: module.foo, module.default
  ```

#### Exporting syntax

- **Inline export:**

  ```js
  export const foo = 42;
  export function bar() {}
  export class Baz {}
  ```

- **Object export:**

  ```js
  const foo = 42;
  function bar() {}
  class Baz {}
  export { foo, bar, Baz };
  ```

- **Renaming export:**

  ```js
  const foo = 42;
  export { foo as myFoo };
  ```

- **Default export (function, class, or value):**
  ```js
  export default function () {}
  // or
  export default class {}
  // or
  const foo = 42;
  export default foo;
  ```

#### Re-exporting syntax

- **Re-export all exports from another module:**

  ```js
  export * from "./utils.js";
  ```

- **Re-export specific exports:**

  ```js
  export { foo, bar } from "./utils.js";
  ```

- **Re-export and rename:**

  ```js
  export { foo as myFoo } from "./utils.js";
  ```

- **Re-export default as named:**

  ```js
  export { default as utilsDefault } from "./utils.js";
  ```

- **Re-export named as default:**
  ```js
  export { foo as default } from "./utils.js";
  ```
