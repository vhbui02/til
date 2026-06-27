# TypeScript Declaration File

To consume declaration files, there are a few methods:

1. **Direct Module Import:** When importing the JavaScript implementation, TypeScript locates the adjacent `.d.ts` file automatically. NOTE that the declarations must be adjacent to the JavaScript modules.

```ts
import { WHIPClient } from "./whip.js";
import { WHEPClient } from "./whep.js";
```

2. **Triple-Slash Reference Directive:** Explicitly associates declaration files in environments where module resolution is not automatic.

```ts
/// <reference path="./whep.d.ts" />
/// <reference path="./whip.d.ts" />
```

3. **TypeScript Configuration Inclusion:** Configure `tsconfig.json` to add declrations to `include:` array

```ts
{
  "compilerOptions": {
    "moduleResolution": "node",
    "allowJs": true,
    "checkJs": true
  },
  "include": [
    "src/**/*",
    "whep.d.ts", // here
    "whip.d.ts", // here
  ]
}
```

> [!TIPS]
>
> Run `npx tsc --noEmit` to validate type resolution and checking.
