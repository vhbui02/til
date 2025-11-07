# JavaScript Runtime Environment Versus JavaScript Engine

## Overview

A **JavaScript Engine** is the program that parses, compiles (often using a Just-In-Time (JIT) compiler), and executes JavaScript code; it handles the core language features defined by the ECMAScript specification.

A **JavaScript Runtime Environment** provides the engine with additional APIs and objects (such as the DOM in browsers or file system access in Node.js) so that JavaScript code can interact with the outside world and underlying system.

Chrome Browser and Node.js both use the same **JavaScript Engine** called **V8**, but they provide different **JavaScript Runtime Environments**:

- Chrome Browser's JavaScript Runtime Environment provides: the DOM (`Node`, `Element`, `HTMLElement`, etc.), global objects (`window`, ...), and various Web APIs (`fetch`, `localStorage`, `setTimeout()`, ...). CommonJS are not supported.

- Node.js's JavaScript Runtime Environment provides: CommonJS module system (`module`, `require`, ...), ES modules (`import/export`), global objects (`global`, `process`, `Buffer`, ...) and built-in modules such as `fs`, `http`, and more.

## Browser

Modern browsers support JS modules in 2 ways: global objects (most commonly via `window`) and ES modules.

### HTMLScriptElement `defer` property

A boolean value controling the order of script execution: code are executed **AFTER** parsing DOM tree but **BEFORE** `DOMContentLoaded` event fired.

- Use `defer` for scripts that aren't blocking the DOM parsing process or manipulating DOM tree, `defer` can be used.
- DO NOT use `defer` for scripts requiring immediate running, before the DOM tree is finished parsing (e.g. `<script>console.log("Hello, World");</script>`).

Engineers tend to write `window.addEventListener("DOMContentLoaded", () => {})` inside deferred script.

Only use `defer` on classic scripts, such as `window` global objects exported modules.

Modules that are exported using ES module syntax (`import/export`) are called "module scripts" and had `defer` true by default.

```html
<!-- using global objects `window` to export module -->
<script src="/global.js" defer></script>
```

```js
// global.js
window.exportedFunction = function () {
  console.log("This is an exported function");
};
```

### HTMLScriptElement `type="module"` property

Many modern browsers support ES modules:

```html
<!-- using ES module syntax -->
<script src="/esm.js" type="module"></script>
```

## References

- [georg's SOF answer](https://stackoverflow.com/a/29027933/9122512)
