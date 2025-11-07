# Nunjucks

Mozilla has created a wonderful template engine that it's my go-to choice when working with 11ty.

## Installation

```sh
# Node
npm install nunjucks
```

## API usage

```html
<!-- Browser -->
<head>
  <script src="nunjucks.min.js"></script>
  <script src="nunjucks-slim.min.js"></script>
</head>
```

```js
nunjucks.configure({ autoescape: true });
nunjucks.configure("views", { autoescape: true }); // relative path
nunjucks.configure("/views", { autoescape: true, web: { async: true } }); // absolute path, recommended

nunjucks.render("index.html", { foo: "bar" });
nunjucks.render("async.html", function (err, res) {});

nunjucks.renderString("Hello {{ username}}", { username: "World" });

env.addGlobal("greeting", "Hello World");
env.getGlobal("greeting");

const env = new nunjucks.Environment();
env.addFilter("syncFilter", function (val, cb) {
  // ...
});
// async, yet must be known at compile-time
env.addFilter(
  "asyncFilter", // name
  function (val, cb) {
    // ...
  },
  true // async
);
env.getFilter("syncFilter");

// must be known at compile-time
env.addExtension("MyExtension", new MyExtension());
env.getExtension("MyExtension");
```

## Precompiled Template

CLI:

```sh
# precompile a whole directory
nunjucks-precompile "./src/_includes/partials" > ./js/partials.js

# precompile individual template
nunjucks-precompile "./src/_includes/partials/partial1.html" > ./js/partial-1.js

# cre: https://stackoverflow.com/a/31418455/9122512
# CAUTION: render() API can't render precompiled templates from macros
# CAUTION: only renderString() API can, but it's not included in slim version.
```

```html
<head>
  <script src="/js/nunjucks-slim.min.js"></script>
  <script src="/js/partials.js"></script>
  <script src="/js/partial-1.js"></script>
</head>
```

```js
// JS API
nunjucks.precompile();
```

> [!CAUTION]
>
> Compiling templates inside browser results in slow page render! It's recommended to always precompile your template using CLI.

There is a [GitHub Gist from 2022](https://gist.github.com/little-brother/c06114cf73cddfca1372c04e3e138867) teaching advanced techniques on how to use Nunjucks in the browser/client-side that populated data remotely.

## Security

### Syntax

```njk
{# This is a comment #}

{{ variableName }}                // expression
{{ variableName | filterName }}   // expression with filter

{% set variableName %}            // control-flow statement
{% set variableName = "value" %}  // sync operation
{% setAsync asyncVariableName %}  // async operation

// NOTE: NOT PLURAL
{% include 'includes.njk' %}     // abs path
{% include './includes.njk' %}   // rel path

// NOTE: THE ONLY PLURAL
{% extends 'base.njk' %}          // abs path
{% extends './base.njk' %}        // rel path

// NOTE: NOT PLURAL
{% import 'macros.njk' as macro %}         // abs path
{% import './macros.njk' as macro %}       // rel path
```

**Q: How to choose between Nunjucks `{% include %}`, Nunjucks `{% import %}` and 11ty `addShortcode()` ?**

A: According to [Darth Mall's "Includes and Macros"](https://darthmall.net/2021/includes-and-macros/):

- Nunjucks `{% include ... %}`: for components that doesn't need parameters passed to it. When an engineer is reading a partial, it's assumed that the context are provided in global inheritance context. Finding where these context came from in the cascading stack requires a lot of context switching, therefore decrease maintainablity. Use cases: `<header>`, `<footer>`, `<nav>`

  **TIPS:** If you need to use `{% set ... %}` to configure the context of a partial `{% include ... %}`. your environment is being polluted with these contexts and engineers tend to forget to unset them. However, that is the only cons. You should use `{% import %}`

- Nunjucks `{% import ... %}` and `{% macro ... %}`: for components that need to take parameters. It's much more maintainable. Use cases: components such as accordion, carousel, table, pagination, hero, card, ...

  **TIPS:** It's best to **put all macros** that are related inside one single file inside a separated directory (e.g. `src/_includes/macros/components.njk` instead of `src/_includes/partials/foobar.njk`), then **import it at the base layout** so that every layouts that `{% extends %}` it can use the macros without having to re `{% import %}`.

- 11ty `addShortcode()`: According to [eleventy#1894 discussion](https://github.com/11ty/eleventy/discussions/1894), **Nunjucks's Macro and 11ty's Shortcodes closely resemble each other.** Use cases: same as Nunjucks's Macro with additional advanced use cases requiring JS logic for processing, or simply you want it to work across multiple template engines.

## [Nunjucks in 11ty](https://www.11ty.dev/docs/languages/nunjucks/)
