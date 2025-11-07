# `window.onload()` API Versus `DOMContentLoaded` event

During the page loading process:

- **`DOMContentLoaded` event**: fires with initial HTML document (DOM tree) is loaded, without waiting for stylesheets, images, iframes, ... Ideal for executing JavaScript code that manipulates the DOM structure as soon as it is ready

  ```js
  window.addEventListener("DOMContentLoaded", function () {
    // DOM is loaded
  });
  ```

- **`window.onload()` API**: fires after the entire page, including images, stylesheets, iframes, ... has finished loading. Use this when all resources must be available before executing code.

  ```js
  window.onload = function () {
    // Everything is fully loaded (DOM, images, styles, etc.)
  };
  ```

Use `DOMContentLoaded` for most DOM-related scripts to improve performance.
