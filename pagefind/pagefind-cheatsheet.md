# Pagefind Cheatsheet

## Overview

Capabilities:

- Fully static client-side search library.
- Low-bandwidth footprint.
- Scalable, up to tens of thousands of pages.
- SSG-agnostic supports (11ty, Hugo, Jekyll, Next, Astro, ...)
- Zero hosting infrastructure.
- [Index library](https://www.npmjs.com/package/pagefind) have zero dependencies.

Core features:

- Zero-config support for localization.
- Rich filter engine.
- Custom sort attributes.
- Custom metadata tracking.
- Custom content weighting.
- Cross-domain searching.
- Index anything (PDFs, JSON, subtitles, ...) using separate NodeJS indexing library.

> Pagefind claims to be running an FTS on a 10000 page site with a total payload < 300kB (including the Pagefind library itself). For most site, this will be ~100kB.

## Mechanism

- Step 1: Analyze the build directory containing static files.
- Step 2: Create index. The search index is split in chunks, the chunks are lazy-loaded.
- Step 3: Add a static search JS bundle to build files which exposes JS API which can be used anywhere on the site. A pre-built UI is provided but you can create custom UI.

## Installation

```sh
npm install -g pagefind
pagefind --site _site

# or
npx pagefind --site _site
```

Output: `_site/pagefind`. It contains Pagefind's browser dependencies, as well as the index files.

## Indexing

By default, all elements inside `<body>` tag are indexed (whitelist all, blacklist some approach), except the following:

- organizational elements (`<nav>`, `<footer>`, etc.)
- programmatic elements (`<script>`, `<form>`, etc.)

Of course, that is undesired outcome. To narrow it down, use built-in `data-pagefind-body` tag. If this tag existed, only data: 1. inside the HTML page that has this tag and 2. inside this tag are indexed.

**NOTE:** At this point, I don't even know why I need `data-pagefind-*` beside `data-pagefind-body` ?

```html
<body>
  <!-- data-pagefind-* and data-pagefind-ignore -->
  <main data-pagefind-body>
    <h1>I'm indexed</h1>
    <p>This content will be indexed</p>
    <p data-pagefind-ignore>This content will NOT be indexed</p>
  </main>
  <aside>This content will not be indexed</aside>
  <article data-pagefind-body>
    <p>This content will be also indexed.</p>
  </article>

  <!-- Ignore tag -->
  <aside data-pagefind-ignore>
    <h1>This might still be detected as the page title</h1>
    <p data-pagefind-meta="a">
      This metadata will still appear in search results.
    </p>
  </aside>
  <aside data-pagefind-ignore="all">
    <h1>This cannot be detected as the page title</h1>
    <p data-pagefind-meta="b">This metadata will not be picked up.</p>
  </aside>

  <!-- Capture value from attributes -->
  <div data-pagefind-card>
    <h1>Condimentum Nullam</h1>
    <img
      src="/hero.png"
      title="Image Title"
      alt="Image Alt"
      data-pagefind-index-attrs="title,alt"
    />
    <p>Nullam id dolor id nibh ultricies.</p>
  </div>
  <!-- Index content: "Condimentum Nullam. Image Title. Image Alt. Nullam id dolor id nibh ultricies." -->
</body>
```

### Indexing ranking

Content in Pagefind's index is ranked. The higher the rank, the more likely it will be shown up on search results.

Custom ranking using `data-pagefind-weight="{{NUM}}"` where `0.0 <= NUM <= 10.0`

```html
<body>
  <div>
    <h1>I have a ranking of 7.0</h1>
    <h2>I have a ranking of 6.0</h2>
    <h3>I have a ranking of 5.0</h3>
    <h4>I have a ranking of 4.0</h4>
    <h5>I have a ranking of 3.0</h5>
    <h6>I have a ranking of 2.0</h6>
    <p>I have a ranking of 1.0, and not just me!</p>
    <p data-pagefind-weight="2">
      I have a custom ranking, it's higher than normal. If the search term
      matches this section, this page will be boosted higher in the search
      results ranking.
    </p>
    <p data-pagefind-weight="0.5">
      I have a custom ranking, it's smaller than normal. My text is unimportant
      and matching words in this block are only worth half a normal word.
    </p>
  </div>
</body>
```

## Searching

```html
<!-- `pagefind` directory is generated when `pagefind` CLI is running -->
<link rel="stylesheet" href="/pagefind/pagefind-ui.css" />
<script src="/pagefind/pagefind-ui.js" defer></script>

<!-- NOTE: you can put this script inside a separate JS file -->
<script>
  window.addEventListener("DOMContentLoaded", (e) => {
    const search = new PagefindUI({
      element: "#search",
      baseUrl: "/docs/", // def="/"
      bundlePath: "/subpath/pagefind/", // there is a chance of search not working and console log warning message that a path could not be detected
      pageSize: 5, // how many results are put inside one page
      showSubResults: true, // show nested results for each heading within a page
      showEmptyFilters: false, // def=true, hide filters with no results along the count
      showImages: false, // show image metadata
      excerptLength: 15, // def to 30, 12 for sub results
      openFilters: ["Tags", "Type"], // def=filter display shows value only there is one filter with <=6 values. Any filter name specified in `openFilters` will be displayed by default
      resetStyles: false, // def=true => apply a CSS to reset itself. Use `false` to inherit from site styles
      debounceTimeoutMs: 500, // wait interval between the moment user stops typing and performing a search automatically. Set to 0 to trigger a search EVERY TIME there is a change
      highlightParam: "highlight", // add search term as a query parameter under the same name
      translations: {
        placeholder: "Search my website",
        zero_results: "Couldn't find [SEARCH_TERM]",
        // ...
        // docs: https://github.com/CloudCannon/pagefind/blob/main/pagefind_ui/translations/en.json
      },
      autofocus: true, // def=false
      sort: {
        date: "desc", // arbitrary sort options, override all ranking. Sort key must be registered using JS API.
      },
      processTerm: function (term) {
        // call before performing a search
        // use cases: normalize search term, ...
        return term.replace(/aa/g, "ā");
      },
      processResult: function (result) {
        // call before displaying each result
        // use cases: fix relative URL, rewrite titles, ...
        console.log(result);
      },
    });

    // search + filter + update UI
    search.triggerSearch("preload search term");
    search.triggerFilters({ Category: ["Article", "Documentation"] });

    // destroy UI
    search.destroy();
    const search = new PagefindUI({
      element: "#search",
      showSubResults: true,
    });
  });
</script>
```

```css
/* CSS simple customization */
body {
  --pagefind-ui-*: ...;
}

body.dark {
  --pagefind-ui-*: ...;
}

/* 
  CSS advanced customization 
  Don't import pagefind-ui.css, use F12 to identify class names and add CSS properties yourself
*/
```

JS API:

```js
const pagefind = await import("/pagefind/pagefind.js");

// best to called BEFORE init()
await pagefind.options({
  baseUrl: "/docs", // def="/"
  bundlePath: "/subpath/pagefind/", // def="/pagefind"
  excerptLength: 15, // def=30
  highlightParam: "highlight", // add search team as query parameter
  ranking: {
    // ...
  },
  // index weight... use for search across multiple site
  // merge filter... use for search across multiple site
});

// load Pagefind's deps and metadata/
// best practice: call init() when search bar is focused
pagefind.init();
```

## Metadata

Auxiliary data displayed inside search results.

Built-in metadata keys: `title`, `image` and `image_alt`, they are inferred from:

- `title` === first `<h1>` content
- `image` === `src` of the first `<img>` follows `<h1>`
- `image_alt` === `alt` of the first `<img>` follows `<h1>`.

Override them using `data-pagefind-meta="title|image[src]|image_alt[alt]"`. Values are NOT combined from multiple tag with the same name inside a page

```html
<!-- capture metadata from element's content -->
<h1 data-pagefind-meta="title">Hello World</h1>

<!-- capture metadata from element's attributes -->
<img
  data-pagefind-meta="image[src], image_alt[alt]"
  src="/john-wick.png"
  alt="John Wick Avatar"
/>

<!-- capture metadata inline -->
<!-- NOTE: value captured regardless of element's location in DOM tree -->
<h1 data-pagefind-meta="date:2025-07-20">Hello World</h1>

<!-- custom metadata key: link_text, link_title, other -->
<!-- capture metadata from content, element's attribute and inline -->
<a
  href="/"
  title="Homepage"
  data-pagefind-meta="link_text, link_title[title], other:Freeform text"
  >Hello World
</a>

<!-- defining fallback metadata -->
<!-- `image` doesn't have to be taken from `src` of some `<img>` -->
<head>
  <meta
    data-pagefind-default-meta="image[content]"
    content="/social.png"
    property="og:image"
  />
</head>
```

## Filtering

Syntax:

```
data-pagefind-filter="<filter_name>"
```

and

```
data-pagefind-filter="<filter_name>[<html_attr_name>]"
```

```html
<!-- capture value from element's content -->
<section>
  <h3>Special thanks to:</h1>
  <p data-pagefind-filter="author">Silverbullet069</p>
  <p data-pagefind-filter="author">vhbui02</p>
</section>

<!--
{
  author: ["Silverbullet069"]
  author: ["Silverbullet069", "vhbui02"]
}
-->

<!-- capture value from element's attribute -->
<!-- NOTE: `data-pagefind-filter` doesn't need to be within `<body>` tag nor inside an element tagged with `data-pagefind-body` -->
<head>
  <meta
    data-pagefind-filter="author[content]"
    content="Pagefind"
    property="og:site_name"
  />
</head>

<!--
{
  "author": ["Pagefind"]
}
-->

<!-- Capture inline value -->
<h1 data-pagefind-filter="author:CloudCannon">Hello World</h1>

<!--
{
  "author": ["CloudCannon"]
}
-->

<!-- Capture all three at once -->
<h1
  data-section="Documentation"
  data-category="Article"
  data-pagefind-filter="heading, tag[data-section], tag[data-category], inline:Freeform text, capture to the end."
>
  Hello World
</h1>

<!--
{
  "heading": ["Hello World"],
  "tag": ["Documentation", "Article"],
  "inline": ["Freeform text, capture to the end."]
}
-->
```

### Filter results behavior in Default UI

- Filter values will be shown in a sidebar.
- A count beside each filter is shown, depicting the number of results available within that filter (**NOTE:** search and current toggled filter results, not total results)
- Filter without any results will still be shown, unless `showEmptyFilters` option is set to `false`.
- Filters are combined using `AND` operation.

### Customize filter results behavior with JS API

```js
// load all values that are registered to be filtered
const filters = await pagefind.filters();

/*
{
  "tag": {
    "Documentation": 4,
    "Article": 12
  },
  "author": {
    "CloudCannon": 6,
    "Liam Bigelow": 12,
    "Pagefind": 1
  }
}
*/

// filtering as part of a search or as part of a standalone action
// 1st param: search phrase or null
// 2nd param: an object with "filters" property
const search = await pagefind.search("insert search phrase", {
  filters: {
    author: "CloudCannon",
  },
});

const search2 = await pagefind.search(null, {
  filters: {
    // Compound filters
    author: ["CloudCannon", "Pagefind"],
    tag: "Article",
  },
});

// 'all', 'any', 'not', 'none'
const search3 = await pagefind.search("insert search phrase", {
  // "tag" includes "Article" && ("author" includes "CloudCannon" || "author" includes "Pagefind")
  filters: {
    author: {
      any: ["CloudCannon", "Pagefind"],
    },
    tag: "Article",
  },
  // "tag" includes "Article" || "author" includes ["CloudCannon", "Pagefind"]
  filters: {
    any: {
      author: ["CloudCannon", "Pagefind"],
      tag: "Article",
    },
  },
  // ...
  filters: {
    any: [
      {
        author: "Pagefind",
        tag: "Article",
      },
      {
        author: "CloudCannon",
        tag: "Documentation",
      },
    ],
    not: {
      year: "2018",
    },
  },
});
```

## Sorting

Syntax: `data-pagefind-sort="<sort_key>"`

`<sort_key>` is registered when calling JS API.

Values that are parsed as numbers will sort numerically, otherwise sort alphabetically.

```html
<!-- sort criterion: element's content-->
<p data-pagefind-sort="date">2025-07-20</p>

<!-- sort criterion: element's attribute -->
<h1 data-pagefind-sort="weight[data-weight]" data-weight="10">Hello World</h1>

<!-- sort criterion: inline value -->
<h1 data-pagefind-sort="date:2025-07-20">Hello World</h1>

<!-- sort criteria: all three at once -->
<h1
  data-weight="10"
  data-date="2025-07-20"
  data-pagefind-sort="heading, weight[data-weight], date[data-date], author:Freeform text, captured to the end"
>
  Hello World
</h1>
```

### Customize sorting behavior with JS API

> [!IMPORTANT]
>
> All ranking are overriden.

```js
const search = await pagefind.search("insert search phrase", {
  sort: {
    // syntax: <sort_key>: "asc" | "desc"
    date: "asc",
  },
});
```

## Reference

- [Pagefind's Docs](https://pagefind.app/docs/)
