# 11ty Cheatsheet

<!-- tl;dr starts -->

11ty is my go-to Static Site Generator for my web projects.

<!-- tl;dr ends -->

Eleventy is a coordinator, between the templates and the data processing through these templates. The template can be Markdown, Nunjucks, HTML, ... and the data can be inside dedicated JSON and JS snippets, the template file's frontmatter, ...

## Directory structure

> Always start by designing your directory structure.

```txt
_site/                                # files that will be served on production
admin/                                # Decap CMS admin directory, no `npm` module
│   |-- index.html
│   |-- config.yml
functions/                            # file-based routing Cloudflare Pages Functions
|   |-- [foo].js
|   |-- bar.js
|   |-- baz/
src/
│   |-- _11ty/                        # Configuration files and logic for Eleventy
│   │   |-- collections/              # Collection definitions (e.g., for multilingual content)
│   │   │   |-- foo_en.js
│   │   │   |-- foo_vi.js
│   │   |-- filters/                  # Pure functions that modify A to B
|   |   |   |-- date.js
│   │   |-- utils/                    # General pure functions
|   |   |   |-- sort.js
│   │   |-- shortcodes/               # Template Language Custom Tags wrapper, rarely used in small project
│   |-- _data/                        # Global data files (JSON/JS)
│   │   |-- foo.json
│   │   |-- bar.js
│   │   |-- ...
│   |-- _includes/
│   │   |-- layouts/                  # Layout templates (Nunjucks)
│   │   │   |-- feed.html
│   │   │   |-- ...
│   │   |-- macros/                   # Nunjucks macros
│   │   |-- partials/                 # Reusable components (e.g., header, footer)
│   |-- assets/
│   │   |-- fonts/                    # .ttf
│   │   |-- imgs/
│   │   │   |-- foo/                  # images for src/content/{{locale}}/foo/
│   │   │   |-- bar/                  # images for src/content/{{locale}}/bar/
│   │   │   |-- favicon.ico
|   |   |   |-- icon.svg
│   │   |-- css/                      # stylesheets (.css, .scss, .sass, ...)
│   │   |-- js/                       # external JavaScript files (local CDN, ...)
│   │   |-- robots.txt
│   |-- content/                      # content that managed by a Git-based CMS
│   |   |-- vi/                       # multilingual projects
│   |   |-- en/
|   |   |   |-- pages/                #
|   |   |   |-- links/
|   |   |   |-- foo/
|   |   |   |   |-- foo.11tydata.js   # export `layout:` and `permalink:`
|   |   |   |-- bar/                  # same as "foo/"
|   |   |   |-- en.11tydata.js        # export locale: "en"
|   |   |-- index.njk
|   |   |-- sitemap.njk               #
│   |-- rss.html                      # RSS feed template
build_tasks/
|
eleventy.config.mjs
jsconfig.json
```

## Configuration API

```js
// plugins
import eleventyRssPlugin from "@11ty/eleventy-plugin-rss";
import { I18nPlugin } from "@11ty/eleventy";

// internal modules
// CAUTION: not recommended
// CAUTION: config file should be SSOT
import { sortByDisplayOrder } from "./src/utils/sort.js";
import dateFilter from "./src/filters/date-filter.js";
import w3DateFilter from "./src/filters/w3-date-filter.js";

// Add TypeScript Type Definitions, which enable some extra autocomplete features in IDE
/** @param {import("@11ty/eleventy").UserConfig} eleventyConfig */
export default function (eleventyConfig) {
  // CAUTION: Order matters!

  // `permalink:` is the only data key that can be processed by template engine
  // NOTE: improve build time significantly
  eleventyConfig.setDynamicPermalinks(false);

  // Plugins
  eleventyConfig.addPlugin(eleventyRssPlugin);
  eleventyConfig.addPlugin(I18nPlugin, {
    // BCP 47-compatible language tag
    defaultLanguage: "en",

    // Rename the default universal filter names
    // optional
    filters: {
      url: "locale_url", // transform a URL with the current page’s locale code
      links: "locale_links", // find the other localized content for a specific input file
    },

    // When to throw errors for missing localized content files
    errorMode: "strict", // throw an error if content is missing at /en/slug
    errorMode: "allow-fallback", // only throw an error when the content is missing at both /en/slug and /slug
    errorMode: "never", // don’t throw errors for missing content
  });

  // per-engine env option
  eleventyConfig.setNunjucksEnvironmentOptions({
    throwOnUndefined: true,
  });

  eleventyConfig.setInputDirectory("src"); // def: .
  // eleventyConfig.setIncludesDirectory("my_includes"); // def: _includes // rel to input dir
  // eleventyConfig.setLayoutsDirectory("_layouts"); // def: _includes // if set, the layouts will live outside of the Includes directory
  // eleventyConfig.setDataDirectory("lore"); // def: _data // rel to input dir
  // eleventyConfig.setOutputDirectory("dist"); // def: _site // rel to root dir

  // eleventyConfig.setQuietMode(true); // 11ty will not show each file it processes and the output file anymore // IMO, it's better use ad-hoc CLI option `--quiet`

  // eleventyConfig.setDataFileBaseName("index"); // by default, 11ty will look for Directory Data Files that match the current folder name. Use this setting if you want to name your files differently

  // eleventyConfig.setDataFileSuffixes([".11tydata", ""]); // file suffixes for Template and Directory Specific Data Files
  // *.11tydata.json, *.11tydata.js, *.json

  // eleventyConfig.setFrontMatterParsingOptions({ language: "js" }); // set the syntax that would be used in your front matter // def is YAML

  /* ===================================================================== */
  /* Passthrough                                                           */
  /* ===================================================================== */
  // Specify files or directories to be copied into output directory
  eleventyConfig.addPassthroughCopy("admin"); // Decap CMS directory
  eleventyConfig.addPassthroughCopy("src/images/"); // Static assets // NOTE: rel to root dir
  // eleventyConfig.addPassthroughCopy("**/*.jpg"); // glob pattern, maintain directory structure
  // eleventyConfig.setTemplateFormats(["md", "css"]); // def=html,liquid,ejs,md,hbs,mustache,haml,pug,njk,11ty.js // css is not yet a recognized template extension in Eleventy
  eleventyConfig.setServerPassthroughCopyBehavior("passthrough"); // def="copy", files are referenced directly and will not be copied to your output directory. Changes to passthrough file copies WILL NOT trigger an 11ty build but will live reload appropriately in the dev server

  // structured content at a collection level
  // CAUTION: inner callback order of execution is unknown, collections creation order is unpredictable
  // TIPS: consider using filter against a collection created by `tags:` with memoization
  eleventyConfig.addCollection("work", (collection) =>
    sortByDisplayOrder(collection.getFilteredByGlob("./src/work/*.md"))
  );
  eleventyConfig.addCollection("featuredWork", (collection) =>
    sortByDisplayOrder(collection.getFilteredByGlob("./src/work/*.md")).filter(
      (x) => x.data.featured
    )
  );
  // why reverse it?
  // 11ty has sorted them in chronological date order
  // we want them to be sorted in reverse chronological, which means newest first
  eleventyConfig.addCollection(
    "blog",
    (collection) =>
      // create a copy of the orginal array and mutate it instead
      // in case we want to use our blog collection somewhere else in project and didn't want to order reversed
      [...collection.getFilteredByGlob("src/posts/*.md")].reverse()
    // NOTE: you can use Nunjucks filter `reverse` instead of using JS method
  );
  eleventyConfig.addCollection("people", (collection) =>
    collection
      .getFilteredByGlob("./src/people/*.md")
      // Markdown files are numbered files instead of named files
      // If stuff changes, content can be switched out without breaking URLs
      .sort((a, b) => (Number(a.fileSlug) > Number(b.fileSlug) ? 1 : -1))
  );

  eleventyConfig.addFilter("dateFilter", dateFilter);
  eleventyConfig.addFilter("w3DateFilter", w3DateFilter);

  // per-engine filter
  eleventyConfig.addNunjucksFilter(
    "concatThreeStrings",
    function (arg1, arg2, arg3) {
      return arg1 + arg2 + arg3;
    }
  );

  return {
    // NOTE; order doesn't matter
    markdownTemplateEngine: "njk", // Markdown files run through this template engine before translating to HTML
    dataTemplateEngine: "njk", // *.11tydata.*, *.json, ...
    htmlTemplateEngine: "njk", // HTML files run through this template engine before translating to (better) HTML
  };
}
```

## [SIX sources of data](https://www.11ty.dev/docs/data/#sources-of-data)

Templates can retrieve data ("variable references") from **SIX** different sources at build time. When similar data exists in multiple sources, higher priority sources override lower priority sources. 11ty calls this behavior the [Eleventy Data Cascade](https://www.11ty.dev/docs/data-cascade/).

I have listed these sources of data in the order of **lowest to highest** priority:

IMO, this Data Cascade is hardest part to grasp in 11ty. One could render their own projects unmaintainable if the template's behavior is hard to understand due to the poor data sources' design decisions.

### 1. [Global Data Files](https://www.11ty.dev/docs/data-global/)

- Store static data can be **globally** dynamically retrived during build (any Layouts and Pages can access them)
- File-based locator: use file/directory name with dot notation.
- Appropriate for small arrays pulling from remote CMS..

**Examples:**

1. Simple static JSON files

```sh
src/_data/goals.json          # JSON file
src/_data/users/goals.json    # JSON file in "users" directory
src/_data/data.json           # JSON file
```

`goals.json`

```json
[
  "Make personal website",
  "Draft blog post",
  "Rebuild personal website",
  "Write blog post",
  "Don't rebuild personal website"
]
```

`data.json`

```json
{
  "myData": ["item1", "item2", "item3", "item4"]
}
```

```html
<ul>
  {% for goal in goals %}
  <li>{{ goal }}</li>
  {% endfor %}
</ul>

<ul>
  {% for goal in users.goals %}
  <li>{{ goal }}</li>
  {% endfor %}
</ul>

<ul>
  {% for item in data.myData %}
  <li>{{ item }}</li>
  {% endfor %}
</ul>
```

2. Dynamic data from JS snippets

- Using `@11ty/eleventy-fetch` official plugin, data can be fetched remote once, and made available to all of the templates that referenced the JS data file's name (e.g. the filename becomes global variables `{{ studioList }}`). JS data files refer to each other using `this` keyword (e.g `this.studioList`)
- `eleventyFetch()` function caches the API responses for 1 day, improve build performance.
- During build process, 11ty auto calls exported function.
- Return empty array on failure, prevent template crash.
- Parameter `configData`: access 11ty's `eleventy` data.

`src/_data/studioList.js`:

```js
import eleventyFetch from "@11ty/eleventy-fetch";

/**
 * Grabs the remote data for studio images and returns back
 * an array of objects
 *
 * @returns {Array} Empty or array of objects
 */
// I will ESM syntax
export default async function (configData) {
  // access configData.eleventy global variable
  // ...

  try {
    let url = "https://11ty-from-scratch-content-feeds.piccalil.li/media.json";
    const { items } = await eleventyFetch(url, {
      // set cache duration to 1 day
      duration: "1d",
      // Eleventy Fetch parse JSON
      type: "json",
    });

    // Pay attention to the data structure of this function's output
    return items;
  } catch (err) {
    console.log(err);
    return [];
  }
}

// you can use GraphQL, expose env var, cache remote images, CSS fonts, ...
```

### 2. [Global Data from the Configuration API](https://www.11ty.dev/docs/data-global-custom/)

> [!TIP]
>
> This is useful plugins, not recommended for app's content.

- Specify data inside `eleventy.config.mjs`
- Useful for plugins.
- Generic Global Data: `addGlobalData()`
- Per-engine Global Data: `addNunjucksGlobal()`

```js
// literal value
eleventyConfig.addGlobalData("myString", "myValue");

// evaluated before setting the value to the data cascade
// myDate is Date instance
eleventyConfig.addGlobalData("myDate", () => new Date());

// myDate is a `function` that returns a Date instance
eleventyConfig.addGlobalData("myFunction", () => {
  return () => new Date();
});

// literal value, complex paths
eleventyConfig.addGlobalData("myNestedObject.myString", "myValue");

// Computed Data - the highest priority source of data
// myString’s value will be "This is a string!"
eleventyConfig.addGlobalData("eleventyComputed.myString", () => {
  return (data) => "This is a string!";
});

// Promise
eleventyConfig.addGlobalData("myFunctionPromise", () => {
  return new Promise((resolve) => {
    setTimeout(resolve, 100, "foo");
  });
});

// async
eleventyConfig.addGlobalData("myAsyncFunction", async () => {
  return Promise.resolve("hi");
});
```

### 3. [Front Matter Data in Layout Templates](https://www.11ty.dev/docs/layouts/#front-matter-data-in-layouts)

- Anything duplicated among the Pages can be put inside Layout frontmatter.
- There are many level of Layouts (11ty called it [Layout Chaining](https://www.11ty.dev/docs/layout-chaining/)): duplicates among low-level Layout frontmatter can be put inside high-level Layout frontmatter.
- **By default**, the Layout extension is `.html` and placed inside `src/_includes/layouts/*.html`.
- Layout frontmatter can be merged with Page frontmatter, Page frontmatter has higher precedence.
- Layout frontmatter can include some [special data keys](#special-data-keys) (e.g. `tags`, `permalink`, `layout:`...) for Pages to use.

> [!NOTE]
>
> The closer to the content, the higher the priority the data.

> [!TIP]
>
> Layout frontmatter should be avoided for maintainability. The fewer data sources there are, the simpler the application will be.

**Examples:** Layout Template: `src/_includes/layout/base.html`

```html
---
title: My Awesome Blog
---

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{ title }}</title>
  </head>
  <body>
    <!-- 'content' is replaced with the content of lower-level templates whose layout is this file -->
    <!-- avoid double-escape the output. 'content' from lower-level templates has already been properly escaped -->
    {{ content | safe }}
  </body>
</html>
```

### 4. [Template and Directory Data Files](https://www.11ty.dev/docs/data-template-dir/)

- Pages are Markdown with frontmatter.
- JavaScript data files have the highest priority.
- The name of the data files must match either the name of the Page (hence the Template Data File), or the name of the directory (hence the Directory Data File).
  => This name matching behavior can be changed with `eleventyConfig.setDataFileBaseName("index");` API. But 99% of the time you will leave it as-is.

**Example:** list available data in Page `src/posts/subdir/my-first-blog-post.md`

- Highest: the Page's front matter data
- High: Data Template Files, applies to only `src/posts/subdir/my-first-blog-post.md`
  - `src/posts/subdir/my-first-blog-post.11tydata.js` (highest, among the files)
  - `src/posts/subdir/my-first-blog-post.11tydata.json`
  - `src/posts/subdir/my-first-blog-post.json` (best practice)
- Normal: Data Directory Files, applies to all Pages in `src/posts/subdir/*`
  - `src/posts/subdir/subdir.11tydata.js` (highest, among the files)
  - `src/posts/subdir/subdir.11tydata.json`
  - `src/posts/subdir/subdir.json` (best practice)
- Low: Data Parent Directory Files, applies to all Pages in `src/posts/**/*`

  - `src/posts/posts.11tydata.js` (highest, among the files)
  - `src/posts/posts.11tydata.json`
  - `src/posts/posts.json` (best practice)

    ```jsonc
    {
      // Apply a default layout and permalink to multiple Pages inside `src/posts/**/*`
      // NOTE: "layout" shouldn't be managed by CMS.
      "layout": "layouts/post.njk",

      // if you're creating a custom permalink
      // make sure to end it with `/index.html` to prevent 11ty creating a plaintext
      // file whose name is the title after being slugified in output directory
      "permalink": "/post/{{ title | slugify }}/index.html",

      // sometimes, source files shouldn't be mapped to an output file
      // i.e. when you want to render a list of contents from multiple files
      // inside a SINGLE page
      "permalink": false // prevent 11ty from generating output file for each Page file
    }
    ```

- Lowest: Global Data Files in `_data/*`

### 5. [Front Matter Data in Page File](https://www.11ty.dev/docs/data-frontmatter/)

- Page's frontmatter fields override things further up the Data Cascade.
- Using `gray-matter` package, three types of front matter can be processed:
  - `---json` JSON frontmatter
  - `---js` JS frontmatter
  - `---` YAML frontmatter (the most common)
- There are some [special data keys](#special-data-keys) that can be specified inside Page's frontmatter.

> [!IMPORTANT]
>
> Nowadays bobody creates Page from scratch. They are created by Content Management System with arbitrary set of frontmatter fields. Therefore, developers should build their Page around these sets.

```yml
---
title: My page title
# TWO special frontmatter keys
# this change where the file goes on the file system
permalink: # can use variable and shortcodes
eleventyComputed: # can use and set variable and shortcodes for other front matter
---
<!DOCTYPE html>
<html></html>
```

### 6. [Computed Data](https://www.11ty.dev/docs/data-computed/)

Calculate data keys from other data keys, under a special data key called `eleventyComputed:`

`eleventyComputed` is best specified inside **JS frontmatter** and/or **JS Data Global/Template/Directory File**

```js
export default {
  eleventyComputed: {
    myTemplateString: "This is a template string.",
    myString: (data) => "This is a string!",
    myFunction: (data) => `This is a string using ${data.someValue}`
    myAsyncFunction: async (data) => await someAsyncThing(),
    myPromise: (data) => {
      return new Promise((resolve) => {
        setTimeout(() => resolve("Delayed 100ms"), 100)
      })
    }
  }
}
```

**Examples:**

1. Create a navigation menu for your site using [Navigation plugin](https://www.11ty.dev/docs/plugins/navigation/)

- Prerequiiste: Navigation plugin relies on special data key `eleventyNavigation` that must be set inside EVERY individual Page file.
- Problem:
  - Pages is created by CMS has arbitrary set of frontmatter fields.
  - JSON Data Directory Files? It can only set default values. `eleventyNavigation` is vary between Page files.
- Solution:

Page file: `src/posts/my-page-title.md` (and other `src/posts/*.md` files)

```yml
---
title: My Page Title
parent: My Parent Key
---
```

---

**Option 1:** JS Global Data File - `src/_data/eleventyComputed.js` (**NOTE: the file name is already eleventyComputed**)

```js
export default {
  eleventyNavigation: {
    // the `data` parameter holding all data that has been cascaded from the start, even `permalink:` (exception)
    key: (data) => data.title,
    parent: (data) => data.parent,
  },
};
```

---

**Option 2:** JS Data Directory File - `src/posts/posts.11tydata.js`

```js
export default {
  eleventyComputed: {
    eleventyNavigation: {
      // the `data` parameter holding all data that has been cascaded from the start, even `permalink:` (exception)
      key: (data) => data.title,
      parent: (data) => data.parent,
    },
  },
};
```

---

**Option 3:** JSON Data Directory File `src/posts/posts.json`

```json
{
  "eleventyNavigation": {
    // template string syntax
    "key": "{{ title }}",
    "parent": "{{ parent }}"
  }
}
```

**Option 4:** Page frontmatter `src/posts/my-page-title.(md|njk)`

```yml
---
title: My Page Title
parent: My Parent Key
eleventyComputed:
  eleventyNavigation:
    # Template string
    key: "{{ title }}"
    parent: "{{ parent }}"
    # reust + overwrite
    title: "This is my new {{title}}"
---
```

---

The following data is automatically provied to Page files

```json
{
  // From Page's frontmatter, or lower-priority data sources in Data Cascade stack
  "title": "My Page Title",
  "parent": "My Parent Key",

  // From JS Data Directory File
  "eleventyNavigation": {
    "key": "My Page Title",
    "parent": "My Parent Key"
  }
}
```

If you don't want to use JavaScript, and the Page was manually created:

- Write YAML frontmatter fields directly for simplicity.
- Write JSON Data Directory files, remember to use the same template syntax.

**NOTE:** template syntax parsing is slower from using JavaScript. Just use JavaScript all the time.

## [Special data keys](https://www.11ty.dev/docs/data-configuration/)

Among the data keys that are specified inside [SIX sources of data](#six-sources-of-data) (except computed data), there are a few built-in special data keys controlling the behavior of the Layout/Page.

**Basic data keys:**

```json
{
  // change the output target of the current template
  // `permalink:` is allowed to use template engine syntax, for `layout:` you can't
  "permalink": "",

  // wrap the current Page with a Layout found inside `src/_includes` directory
  // can't use template engine syntax
  // Don't add prefix `src/_includes`
  "layout": "",

  // enable iterating over data
  // output multiple HTML files from a single fiie.
  // use a technique called "Content as Data"
  "pagination": "",

  // a String or an Array of String
  // identify a content is part of a Collection
  "tags": "foo",
  "tags": ["foo", "bar"],

  // customize date behavior, allows you to control how a content is sorted inside
  // a Collection
  // The order in which datetime data sources are applied:
  // 1. File creation data in OS
  // 2. Datetime format found in file's name
  // 3. `date:` frontmatter key. Beside standard datetime format, 11ty supports a set of options
  "date": 2025-01-01,           // no double quotes
  "date": "2025-01-01",         // double quotes
  "date": "Last Modified",      // stat -c "%y"
  "date": "Created",            // stat -c "%W"
  "date": "git Last Modified",  // latest Git commit
  "date": "git Created",        // first Git commit
}
```

**Advanced data keys:**

```json
{
  // Per-file template engine customization
  // By default, Markdown files are processed with `markdownTemplateEngine` configuration option
  // If this option is used, explicitly listed every engines you would like to use
  "templateEngineOverride": "md", // only Markdown, nothing else, not markdownTemplateEngine`
  "templateEngineOverride": "njk, md", // processed via Nunjucks first, its output will be feeded to Markdown
  "templateEngineOverride": false, // copy the template, without doing any transformation.

  // Per-file Collection exclusion
  "eleventyExcludeFromCollections": false,

  // The 6th data sources
  // Layman's term - you can create data keys whole value is dependent on other data keys, including ones created by 11ty like `page`
  // Resembles dynamic Data Directory File, which is something only `permalink:` is exceptionally supported
  // NOTE: refer to them directly instead of `page.eleventyComputed.*`
  "eleventyComputed": {
    "key": "{{ title }}",
    "parent": "{{ parent }}"
  }
}
```

- `eleventyDataSchema`: validate data in the Data Cascade stack.
- `eleventyNavigation`: object used by Navigation plugins.
- `eleventyImport.collections`: ...
- `dynamicPermalink`: ...
- `permalinkBypassOutputDir`: ...

### Permalink

`permalink` data key allows remapping the template's output path to a different path than the default. By default, this is the outputing behavior:

<!-- prettier-ignore -->
| Input | Output | `<href="...">` |
| --- | --- | --- |
| `src/index.md` | `src/_site/index.html` | `/` |
| `src/about.md` | `src/_site/about/index.html` | `/about/` |
| - `subdir/template.md`<br/>- `subdir/template/template.md`<br/>- `subdir/template/index.md` | `_site/subdir/template/index.html` | `/subdir/template/` |

**Static `permalink:`**

```yml
---
title: "Static permalink"
permalink: "dir/subdir/unexisted/"
permalink: "dir/subdir/unexisted/index.html" # always end with /index.html at the end
permalink: false # prevent writing file to output directory

# Unexisted output nested directory are created automatically
# Output:
# _site/new-path/subdir/unexisted/index.html
```

**Dynamic `permalink:`**

```yml
---
title: "Dynamic permalink"
permalink: "/{{ page.fileSlug }}/" # remove directory prefix
permalink: "/{{ page.date }}/{{ page.filePathStem }}" # prepend date with existing structure
permalink: "/{{ title | slug }}/" # create permalink from title
permalink: "subdir/{{ title | slugify }}/index.html"

# YAML parse everything wrapped inside `{}` as obj if there is no double-quotes
# Enclose every text data with double-quotes to prevent unexpected behavior
# Output:
# _site/subdir/new-path/index.html
```

**`permalink:` utilize `page:` variable:**

```yml
---
date: "2025-05-01"
permalink: "/blog/{{ page.date | date: '%Y/%m/%d' }}/index.html"
permalink: "/posts/{{ page.date | date: '%Y/%m/%d' }}-{{ title | slugify }}/index.html"
# Refresh memory: `page.date` comes from `date` frontmatter, date in filename or file creation date in OS filesystem as fallback
# Output target: _site/2025/06/29/index.html
# Clean URL without .html: https://examples.com/2025/06/29/
---
```

2. A directory contains multiple high-level Layout Templates (e.g. `recipes/cookies.md`, `recipes/soup.md`, ... etc 50 more).

To set permalinks to all of them, manually set a `permalink:` frontmatter data is a bad idea, we need to populate `permalink:` data from **outside** of these files. We can do so by creating a JS Data Directory File:

`recipes/recipes.11tydata.js`:

```js
export default {
  // NOTE: `permalink` is an exception of implied Computed Data - the highest priority data source
  // NOTE: that explains why `permalink` have access to `title`
  permalink: function ({ title }) {
    return `/recipes/${this.slugify(title)}`;
  },
};
```

3. Map one URL to multiple files for Internationalization (a.k.a i18n)

<!-- TODO: come back to this after you have i18n use case -->

### Layouts

<!-- TODO: finish -->

### Pagination

Iterate over a data set and create multiple files from a single template. Pagination can only be specified inside a Layout Template's frontmatter.

Pagination can be made against an Array (most common), an Object. Data source can come from frontmatter data, local or global files.

---

Data structure of `pagination` object:

```json
// prettier-ignore
{
  "data": "...",        // original string key to the dataset (i.e. `data:` value)
  "size": 69,           // page chunk sizes

  "items": [],          // array of current pags's chunk of data
  "pageNumber": 0,      // current page number, zero-based indexed

  "hrefs": [],          // array of all page's `<a href="...">`
  "href": {
    "next": "url",      // the URL to put inside <a href="...">Next Page</a>
    "previous": "url",  // the URL to put inside <a href="...">Previous Page</a>
    "first": "url",     // the URL to put inside <a href="...">First Page</a> 
    "last": "url"       // the URL to put inside <a href="...">Last Page</a>
  },

  "pages": [],          // array of all chunks of paginated data (in order)
  "page": {
    "next": {},         // Data object for the next page
    "previous": {},     // Data object for the previous page
    "first": {},        // Data object for the first page
    "last": {}          // Data object for the last page
  }
}
```

---

1. `src/paged.njk`:

```yml
---
tags:
  - myCollection
pagination:
  data: testdata  # a variable whose value is an array, a collection, ...

  size: 1 # control the number of each chunk
  size: 2

  # if size=1, alias == Scalar `pagination.items[0]`
  # if size>=2, alias == Array `pagination.items`
  alias: wonder

  # Reverse the data, output: `["item 4", "item 3"]` and `["item 2", "item 1"]`.
  # NOTE: Collection API can do this also
  reverse: true

  # remove values from paginated data, output: `["item 1", "item 2"]` and `["item 4"]`
  filter:
    - item3

  # if the layout has a `tags:` by default, each page generated by the Pagination will be added to the same collection
  addAllPagesToCollections: true

  # Force generate one pagination output with empty chunk of items
  generatePageOnEmptyData: true

  # CAUTION: this property can only be defined inside JS frontmatter ---js
  # Callback functions that modify, filter, change the pagination data in general
  #
  # Order of execution:
  # 1. before:
  # 2. reverse: true
  # 3. filter:
  before: |
    function(paginationData, fullData) {
      let slug = this.slugify(fullData.title)
      return paginationData.map(item => `${slug}-${item} with a suffix.`)
    }

testdata:
  - item1
  - item2
  - item3
  - item4
permalink: "different/{{ pagination.items[0] | slugify }}/index.html"
permalink: "different/{{ wonder | slugify }}/index.html"  # better
permalink: "different/{{ wonder[0] | slugify }}/index.html" # size >=2
---

You can use the alias in your content too {{ wonder[0] }}.

# for size: 1
# output: _site/different/item1/index.html, _site/different/item2/index.html

# for size: 2
# output: _site/different/item1/index.html, _site/different/item3/index.html
```

An example of `fullData`:

```
fullData: {
  huveco: {
    siteUrl: 'https://huveco.com',
    name: 'Huu Viet Manufacturing and Trading Company Limited (HUVECO)',
    shortName: 'HUVECO',
    emails: [ 'sales@huveco.com', 'huveco@gmail.com' ],
    address: 'No 4, lane 10, group 80, Khuong Trung, Thanh Xuan, Hanoi, Vietnam.',
    telephone: '(84-24) 3565 0861',
    copyrightYears: '2007, 2025',
    metaDesc: 'Vietnam bamboo lacquer bowl dish plate tray box vase silk bag handbag . Huu Viet manufacturing and trading company Ltd ( HUVECO ) is one of the leading Vietnamese handicraft manufacturer in manufacturing and exporting handmade home and garden furniture, decorative products. We are also specialized in manufacturing unique and special embroidery and household textile, handbags, souvenirs, gifts and crafts, holiday gift and decoration',
    metaKeywords: 'Vietnam bamboo lacquer bowl dish plate tray box vase silk bag handbag . Huu Viet Manufacturing and Trading Company Ltd , Huveco , Vietnam handicrafts , Vietnam bamboo bowl , bamboo bowls , bamboo dish , bamboo dishes , bamboo plate , bamboo plates , bamboo tray , bamboo trays , bamboo box , bamboo boxes , bamboo vase , bamboo vases , bamboo pot , bamboo pots , Vietnam lacquer bowl , lacquer bowls , lacquer dish , lacquer dishes , lacquer plate , lacquer plates , lacquer tray , lacquer trays , lacquer box , lacquer boxes , lacquer vase , lacquer vases , lacquer pot , lacquer pots , Vietnam bamboo handbag , bamboo handbags , seagrass handbag , sea grass handbag , seagrass handbags , rattan handbag , rattan handbags , silk handbag , silk handbags , suede handbags , canvas handbag , canvas handbags , purse , purses , wallet , wallets , wooden jewelry & gift boxes',
    socialImage: '/images/templates/huveco.png'
  },
  helpers: { readMarkdownFiles: [AsyncFunction: readMarkdownFiles] },
  eleventy: {
    version: '3.1.2',
    generator: 'Eleventy v3.1.2',
    env: {
      source: 'cli',
      runMode: 'serve',
      config: '/home/silverbullet069/LocalRepository/huveco-static/eleventy.config.mjs',
      root: '/home/silverbullet069/LocalRepository/huveco-static'
    },
    directories: {
      input: './src/',
      inputFile: undefined,
      inputGlob: undefined,
      data: './src/_data/',
      includes: './src/_includes/',
      layouts: undefined,
      output: './_site/'
    }
  },
  pkg: {
    type: 'module',
    devDependencies: {
      '@11ty/eleventy': '^3.1.2',
      '@11ty/gray-matter': '^2.0.0',
      'decap-server': '^3.3.0',
      nunjucks: '^3.2.4',
      pagefind: '^1.3.0',
      wrangler: '^4.26.0'
    }
  },
  tags: [ 'products', 'Y8NmQEnKD6fIbDgTiaapC' ],
  layout: 'layouts/product.html',
  eleventyImport: { collections: [ 'all' ] },
  permalink: '{{ page.filePathStem }}/{{ pagination.pageNumber + 1 }}.html',
  pagination: {
    data: 'collections',
    size: 20,
    alias: 'productList',
    generatePageOnEmptyData: true,
    addAllPagesToCollections: false,
    before: [Function: before]
  },
  id: 'L-jRnKv2inVybDcbH67WC',
  title: 'BC003',
  description: 'Coiled and pressed bamboo vase.',
  size: [ 'D30 x H50 cm' ],
  date: '2006-11-05',
  material: 'Spun bamboo',
  image: '/images/products/BC003849514.jpg',
  imageAlt: 'BC003849514.jpg',
  trending: false,
  new: false,
  slideshow: true,
  page: {
    inputPath: './src/products/bc003.md',
    fileSlug: 'bc003',
    filePathStem: '/products/bc003',
    outputFileExtension: 'html',
    templateSyntax: 'njk,md',
    date: 2006-11-05T00:00:00.000Z,
    rawInput: ''
  },
  collections: {
    channels: [
      [Object], [Object],
      [Object], [Object],
      [Object], [Object],
      [Object], [Object],
      [Object], [Object]
    ],
    nav: [ [Object], [Object], [Object], [Object] ]
  }
}
```

---

2. `src/_data/globalDataSet.json` and `src/paged.njk`

```json
{
  "myData": ["item1", "item2", "item3", "item4"]
}
```

```md
---
pagination:
  data: globalDataSet.myData
  size: 1
---

<ol>
<!-- beside `items` array, there are many more property inside `pagination` object -->
{% for item in pagination.items %}
  <li>{{ item }}</li>
{% endfor %}
</ol>

<!-- Default output behavior -->
<!-- _site/paged/index.html -->
<!-- _site/paged/1/index.html -->
```

3. Paging a Collection

11ty has a special data key called `tags` to group templates into a Collection data structure. 11ty can make pagination data out of Collection data.

`src/blog.njk`:

```md
---
pagination:
  data: collections.posts
  size: 6
  alias: posts
---

<ol>
{% for post in posts %}
  <!-- there are so much more properties inside `posts` and `post` -->
  <li><a href="{{ post.url }}">{{ post.data.title }}</a></li>
{% endfor %}
</ol>
```

### Dates

<!-- TODO: finish this -->

## [Supplied Data](https://www.11ty.dev/docs/data-eleventy-supplied/#page-variable-contents)

A list of data keys that're either built-in or computed based on [special data keys](#special-data-keys). They can be referred to inside any Layout Template or Page.

You can called them ""

> [!TIP]
>
> The above keys are reserved keywords, don't create new data key with the same name

There are **FIVE** major built-in Global Variables:

1. [`pagination`](#pagination): divide data into chunks for multiple output pages.
1. [`collections`](#collections): lists of all of your content, grouped by tags. Use dot notation (i.e. `collections.featuredWork`).
1. `page`: has information about the current page

   - `page.url`: `false` if `permalink` set to `false`, else `/path/to/template/` (trailing slash!)
   - `page.inputPath`: path to original source file for the template (e.g. `./path/to/template/file.md`)
   - `page.fileSlug` : `inputPath` filename without file ext (e.g. `file`)
   - `page.filePathStem`: `inputPath` without file ext (e.g. `/path/to/template`)
   - `page.date`: JS Date() object, can be used to sort collections.
   - `page.outputFileExtension`: use as suffix to `filePathSteam` for custom file extensions (e.g. `html`)
   - `page.outputPath`: path to output file in output directory (e.g. `./_site/path/to/output/file.html`)
   - `page.templateSyntax`: which type of files are processed (e.g. `liquid, md`)
   - `page.rawInput`: the unparsed/unrendered plaintaxt content of current template (e.g. `<!DOCTYPE...`)
   - `page.lang`: only needed with i18n plugin.

1. `eleventy`: contains 11ty-specific data from env vars.

   - `eleventy.version`: 11ty version
   - `eleventy.generator`: for use with `<meta name="generator"`
   - `eleventy.env`:
     - `eleventy.env.root`: abs path to the dir in which you run 11ty CLI command
     - `eleventy.env.config`: abs path to config files
     - `eleventy.env.source`: either `cli` or `script`
     - `eleventy.env.runMode`: either `build`, `serve` or `watch`.
   - `eleventy.directories`: root-relative normalized path
     - `eleventy.directories.input`: `./`
     - `eleventy.directories.includes`: `./_includes/` (default)
     - `eleventy.directories.data`: `./_data` (default)
     - `eleventy.directories.output`: `./_site` (default)

   ```json
   {
     "eleventy": {
       "version": "3.1.2",
       "generator": "Eleventy v3.1.2",
       "env": {
         "source": "cli",
         "runMode": "serve",
         "config": "/home/silverbullet069/LocalRepository/huveco-static/eleventy.config.mjs",
         "root": "/home/silverbullet069/LocalRepository/huveco-static"
       },
       "directories": {
         "input": "./src/",
         "inputFile": "...",
         "inputGlob": "...",
         "data": "./src/_data/",
         "includes": "./src/_includes/",
         "layouts": "...",
         "output": "./_site/"
       }
     }
   }
   ```

1. `pkg`: the local project's `package.json` data.
   - `pkg.name`
   - `pkg.description`
   - `pkg.version`
   - `pkg.main`
   - `pkg.scripts`
   - `pkg.type`
   - `pkg.keywords`
   - `pkg.author`
   - `pkg.license`
   - `pkg.dependencies`
   - `pkg.devDependencies`

## Collections

### [Using Configuration API `addCollection()`](https://www.11ty.dev/docs/collections-api/)

This is the most dynamic method to create collections:

```js
export default function (eleventyConfig) {
  // sync
  // local data
  // no I/O
  // build speed is critical

  // CAUTION: non-deterministic order of execution
  // async-friendly
  eleventyConfig.addCollection("myCollectionName", async (collectionsApi) => {
    // get unsorted items
    return collectionsApi.getAll();
  });

  // remote API calls
  eleventyConfig.addCollection("externalProducts", async (collectionsApi) => {
    const response = await fetch("https://api.example.com/products");
    const products = await response.json();
    return products; // add sort() or filter() for further modification
  });

  // database queries
  eleventyConfig.addCollection("dbProducts", async (collectionsApi) => {
    const db = await connect("insert connection string...");
    const products = await db.query("SELECT * FROM products;");
    await db.close();
    return products;
  });

  // file system operations...
  // image processing...
}
```

### Using `tags`

- A single Page can belong to multiple collections by specifying multiple `tags` values.
  - As the data cascades, `tags` are combined into an array. Therefore, any operations regarding `tags:` must viewed as an indexed array.
- A single Collection can holds many Pages.
- By default, Collections are sorted ascending by default, apply Nunjucks `{{ foo | reverse }}` filter to avoid in-array modification.

---

The data structure of `collections:` built-in variable:

```json
//
{
  "post": [], // array
  "post-with-dash": [] // array
  // ...
}
```

Each property inside `collections:` object is an array. Each item inside a `collections.<property>` has the following data structure:

```json
{
  // NOTE: You can omit `page.` dot notation due to backward compatibility
  // NOTE: However, it's recommonded to use page.* for better readability

  // Everything inside `page` built-in global variables
  "page": {
    "url": "/current/page/test/", // false if "permalink" set to `false`. Can be appended with `index.html`
    "inputPath": "./src/current/page/test.md", // include input directory path (i.e. `src`)
    "fileSlug": "test", // basename of "inputPath", extensionless.
    "filePathStem": "/current/page/test", // "inputPath", extensionless
    "date": "new Date()", // JS Date obj, used to sort Collections
    "outputFileExtension": "html", // custom extension for output file, useful for "filePathStem"
    "outputPath": "./_site/path/to/output/file.html", // path to output file in output directory
    "templateSyntax": "liquid, md, html", // >=v2.0, which type of files are processed
    "rawInput": "!<doctype html>", // unparsed/unrendered content of current Page
    "lang": "" // only needed with i18n plugin.
  },
  "data": {
    "title": "",
    "tags": [],
    "permalink": ""
    // ...
    // all data that can be accessed inside this piece of content
  },

  // NOTE: avoid wrapping "templateContent" (or "content") within non-container HTML
  // elements (i.e. not `div`, `span`) because it has already compiled and included
  // HTML elements (which we don't know to know) as appropriate
  "content": "Markdown body, escaped",
  "templateContent": "Markdown body, escaped", // alias for "content", backward-compatibility

  "rawInput": "Markdown body, unescaped"
}
```

`src/myPost.md`

```md
---
title: My Title
# single tag
tags: post
# multi tags, single line
tags: ["post", "post-with-dash"]
# multi tags, multi line
tags:
  - post
  - "post-with-dash"
# exclude content from being added to EVERY collection
eleventyExcludeFromCollections: true
# exclude content from being added to `post` collection
eleventyExcludeFromCollections: ["post"]
---

This will not be available in `collections.all` or `collections.post`.
```

Inside any templates:

```html
---
# declare Collection dependency, inform relationship for smarter incremental builds
eleventyImport:
  collections: ["post"]
---

<ul>
  {% for post in collections.post %}
  <li>{{ post.data.title }}</li>
  {% endfor %}
  <!--  -->
  {% for post in collections['post-with-dash'] %}
  <li>{{ post.data.title }}</li>
  {% endfor %}
  <!--  -->
  {% for post in collections.all %}
  <li><a href="{{ post.url }}">{{ post.url }}</a></li>
  {% endfor %}
</ul>
```

Automatically generate Tag Pages, use pagination to automatically generate a template for each tag:

```md
---
pagination:
  data: collections
  size: 1
  # tag is an array holding the list of keys of the `collections` object
  # or, the list of tag names, the list of string
  alias: tag
  # decentrailized tag list + black list design
  filter:
    - foo
    - bar
# for size=1, tag is collections.items[0] or the tag name
permalink: "/tags/{{ tag | slugify }}"w
---

<h1>Tagged "{{ tag }}"</h1>

<ol>
<!-- accessing the tag  -->
{% set tagList = collections[ tag ] %}
{% for post in tagList | reverse %}
  <li><a href="{{ post.url }}">{{ post.data.title }}</a></li>
{% endfor %}
</ol>
```

Each Page introduce a new tag results in a new pagination page being created: `_dist/tags/new-tag/index.html`. This design allows tag to be decentralized and don't have to be maintained manually.

## [Events](https://www.11ty.dev/docs/events)

11ty supports the ability to run a JS function before/after the building process:

```js
// export function config(eleventyConfig) { ... }
// this is a named export that helps to know where should `directories` be injected
// since there might be more than 1 config is exported
export default function (eleventyConfig) {
  eleventyConfig.on(
    "eleventy.before",
    async ({ directories, runMode, outputMode }) => {
      const { input, inputFile, inputGlob, data, includes, layouts, output } =
        directories;

      // outputMode: a string represents `--to` value
      // valid values: "fs" (default), "json", "ndjson"

      // runMode: a string represents `--serve` or `--watch` on CLI
      // valid values: "build" (default), "watch", "serve"
    }
  );

  eleventyConfig.on(
    "eleventy.after",
    async ({ directories, results, runMode, outputMode }) => {
      // directories, runMode, outputMode structure is the same as the one being injected in `eleventy.before`
      //
      // results: an array with processed 11ty output
      // each element has the following structure: { inputPath, outputPath, url, content }
    }
  );

  // eleventyConfig.on("eleventy.beforeConfig", () => {})
  // eleventyConfig.on("eleventy.beforeWatch", () => {})
  // eleventyConfig.on("contentMap", () => {})

  // by default, event callbacks are triggered in parallel
  eleventyConfig.setEventEmitterMode("sequential");
}
```

## [Filters](https://www.11ty.dev/docs/filters)

11ty has a set of [Built-in Universal Filters](https://www.11ty.dev/docs/filters/#eleventy-provided-filters)

- `| url`: normalize absolute path in content, allow changing deploy subdir
- `| slugify` (or `| slug`): change all non-alphanumeric to hyphen. V3 introduced memoization.
- `| log` : run `console.log`
- `| getNextCollectionItem` : get next collection item
- `| getPreviousCollectionItem` : get previous collection item
- `| inputPathToUrl` : map a template input path to output URL. V3 introduced memoization.
- `| renderTransforms` : ...

You can create custom filter using Configuration API:

```js
export default function (eleventyConfig) {
  // CAUTION: make sure you're NOT USING ARROW FUNCTION HERE
  // CAUTION: `this` keyword will create unexpected behavior
  eleventyConfig.addFilter("filterName", function (value) {
    // this.page
    // this.eleventy
    // this.env
    // this.ctx
  });
  eleventyConfig.addAsyncFilter("asyncFilterName", function (value, callback) {
    // only works for 3 template engines:
    // - Liquid
    // - Nunjucks
    // - JavaScript
  });

  // Memoization supported
  // NOTE: remember install a memoization library, e.g. `lodash.memoize`
  eleventyConfig.addFilter(
    "memoizedFilterName",
    memoize((value) => {})
  );

  // Limited filters to per template engine
  // NOTE: check for built-in filters before creating your own
  eleventyConfig.addLiquidFilter("syncLiquidFilter", function (value) {});
  eleventyConfig.addLiquidFilter(
    "asyncLiquidFilter",
    async function (value) {}
  );

  eleventyConfig.addNunjucksFilter("syncNunjucksFilter", function (value) {});
  eleventyConfig.addNunjucksFilter(
    "asyncNunjucksFilter",
    async function (value) {}
  );
  // direct async method
  eleventyConfig.addNunjucksAsyncFilter(
    "asyncNunjucksFilter2",
    function (value1, value2, callback) {
      setTimeout(function () {
        // 1st param: error obj
        // 2nd param: result data
        callback(null, "My Result");
        // syntax: {{ myValue1 | asyncNunjucksFilter2(myValue2) }}
      }, 100);
    }
  );

  eleventyConfig.addJavaScriptFunction(
    "asyncJSFilter",
    async function (value) {}
  );
}
```

## [Shortcodes](https://www.11ty.dev/docs/shortcodes/)

It's a way of writing reusable components using JavaScript. It retures a JS string or template literal.

```js
export default function (eleventyConfig) {
  // Shortcodes are available in:
  // - Markdown
  // - Liquid
  // - Nunjucks
  // - JS

  // sync
  // NOTE: Again, make sure you're not using arrow function
  eleventyConfig.addShortcode("user", function (firstname, lastName) {
    // this.page
    // this.eleventy
    // this.env
    // this.ctx
  });

  // memoization supported
  eleventyConfig.addShortcode(
    "user",
    memoize((firstname, lastName) => {
      // ...
    })
  );

  // async, version >=2.0.0
  eleventyConfig.addShortcode("user", async function (firstName, lastName) {});

  // direct async
  eleventyConfig.addAsyncShortcode(
    "user",
    async function (firstName, lastName) {}
  );

  // per-engine shortcodes
  // nunjucks
  eleventyConfig.addNunjucksShortcode(
    "user",
    function (firstname, lastName) {}
  );
  eleventyConfig.addPairedNunjucksShortcode(
    "user",
    function (content, firstname, lastName) {}
  );
  eleventyConfig.addNunjucksAsyncShortcode(
    "user",
    async function (firstname, lastName) {}
  );
  eleventyConfig.addPairedNunjucksAsyncShortcode(
    "user",
    async function (content, firstname, lastName) {}
  );
}
```

---

**Paired Shortcodes - introduce start and end tag and Nested Content:**

```njk
{% user firstName, lastName %}
  Hello {{ name }}
  Hello {% anotherShortcode %}
{% enduser %}
```

```js
export default function (eleventyConfig) {
  // sync
  eleventyConfig.addPairedShortcode(
    "user",
    function (content, firstname, lastName) {}
  );

  // async, version>=2.0.0
  eleventyConfig.addPairedShortcode(
    "user",
    async function (content, firstName, lastName) {}
  );

  // async method
  eleventyConfig.addPairedAsyncShortcode(
    "user",
    async function (content, firstName, lastName) {}
  );
}
```

## Plugins

11ty has a long list of both Official and 3rd-party Community Plugins. Official plugins live under `@11ty` NPM organization and have its name prefixed with `@11ty/`

There are **NINE** official plugins that could prove useful.

1. Image

   - [Optimize multimedia delivery on the web](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Performance/Multimedia).
   - Perform build-time image transformation,
   - Cache remote images locally.
   - Add `width` and `height` attributes.
   - Accept wide variety of image input types: `jpeg`, `png`, `webp`, `svg`, ... Does not rely on file extensions.
   - Output multiple image sizes, maintain original aspect ratio. Generate `srcset` attribute.
   - Output multiple formats: `jpeg`, `png`, `webp`, `avif`. Generate the most efficient HTML markup using `<img>` and `<picture`.
   - Fast: deduplicate in-memory and disk cache.
   - Robust, local-first: save remote images, prevent broken URLs (via `@11ty/eleventy-fetch`).
   - Install: `npm install @11ty/eleventy-img`
   - There are 5 different ways to use this plugin:
     - **Image HTML Transform**: start with this one
     - Image Data Files: use images to populate data in the Data Cascade.
     - Image JS API: low-level JS API works independently of 11ty.
     - Image Shortcodes: use universal shortcodes in Nunjucks, Liquid or 11ty.js templates.
     - Image WebC: use WebC component for WebC templates.

1. Fetch

   - A utility to fetch and cache network requests.
   - Install: `npm install @11ty/eleventy-fetch`

1. `<is-land>` for Islands Architecture

   - Smartly and efficiently load and initialize client-side components.
   - This plugin enabled a hybrid architecture: 95% SSG and 5% CSR.
   - Easy to add to existing components.
   - Zero dependencies.
   - Small digital footprint (`4.56 kB` minimized, `1.47 kB` with Brotli compression)
   - Not tightly coupled to any server frameworks or SSGs.
   - Support SSR frameworks: Svelte, Vue, Preact
   - Install: `npm install @11ty/is-land`

1. Internationalization (i18n)

   - Manage pages and linking between localized content on 11ty projects

1. RSS

   - Generate an RSS/Atom feed to allow others to subscribe to your content using a RSS feed reader.

1. Upgrade Helper

   - Update 11ty project between major version releases.

1. Syntax Highlighting

   - Code syntax highlighting using PrismJS without client-size JS.

1. Navigation

   - Create hierarchical navigation in 11ty projects, supported breadcrumbs.

1. Bundle

   - Create small plain-text bundles of code (HTML, CSS, JS, SVG, ...)

## Services

List of tech stacks that support Eleventy integration:

### Deployment and Hosting

- GitHub Pages
- GitLab Pages
- **Cloudflare Pages**: my current use.
- Cloudflare Workers Static (hardcore).

Some play really nice with 11ty's Official plugins. Cloudflare Pages preserves `.cache` directory that is used by [Eleventy Fetch](https://www.11ty.dev/docs/plugins/fetch/) plugin. So does GitHub Pages.

### CMS

CMS add a web-based interface to the site, tech and non-tech personnel can easily update the site on-the-go. 11ty is not tightly coupled to any specific CMS. 11ty works best with Headless CMS what share the same characteristics.

I use **Decap CMS**, a Headless, Git-based CMS solution:

- The data is version controlled.
- Works as-is with your existing deployment process (e.g. works with deploy preview)
- No data migration is needed if in the future you don't want to continue using Decap CMS.

## Tips

- Learn how to design a good template frontmatter, template body and data files.
- DO NOT put everything inside Page's frontmatter, use **Page's body** if you're going to rely on Markdown's format to style a large amount of text.
- Use `{{ ... | log }}` Universal Filter to debug.
- Keep your template DRY by deduplicating modules in `src/_includes/partials`. Design Partials is like design **functions**. They need to have parameter, to be pure and idempotent.
- An SSG working with remote data can turn itself into a front-end for a CMS.
- Read [Quick Tip](https://www.11ty.dev/docs/quicktips/) to learn some more best practices.
- `.html` will be Layout Template, `.md` will be Page.
- With the help of CMS, duplication is not a problem anymore and Data/Template Directory File might not needed.
- DO NOT modify Page's frontmatter data using text editor, it must be done via CMS.
