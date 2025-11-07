# `window.location.href` versus `window.open()`

<!-- tl;dr starts -->

There are two ways to switch the current opened page onto another page: `window.location.href` property and `window.open()` method.

<!-- tl;dr ends -->

According to [Somnath Muluk's answer on SOF](https://stackoverflow.com/a/39397877/9122512)

- Get the current information about the current page: there are numerous information that can get from `window.location` object:
  - url
  - protocol
  - hostname
  - hashstring
  - ...
- Redirect the page to another page: use `window.location.href`
- Open link in the new or specific window: use `window.open(url, windowName, windowFeatures)`. Read more at [JavaScript Coder "Using the window.open method"](https://www.javascript-coder.com/window-popup/javascript-window-open/)
