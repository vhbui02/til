# HTML Semantic Elements

<!-- tl;dr starts -->

While I was learning 11ty at this [site](https://learn-eleventy.pages.dev), I've learned quite a great deal about writing Semantic HTML. I would like to note them down.

<!-- tl;dr ends -->

1. `<main>`

- ONE page should only have ONE `<main>`
- Main content of the page, **make the page become special and unique**
- Used for: blog's content, product list, right-y admin panel, ... anything that holding the data that is the center of the page.
- TIPS: add it into the Base layout of `11ty`

When there are tons of links above main content, when someone focuses to the `<main>` and hit `Tab`, their focus will be sent to the next focusable element inside it.

```html
<!-- id and tabindex allows the element to be "programmatically focused" -->
<!-- when someone clicks a link that goes to #main-content, the <main> element will be focused -->
<main tabindex="-1" id="main-content"></main>
```

2. `<section>`

- Groups of content that shares the same topic, ideally they're having `<h1>` to `<h6>`
- Examples: New Posts, FAQ, ... or separated segments of a page
- TIPS: use `aria-label` for better accessbility.

3. `<article>`

- An independent data section, tend to be reused.
- Examples: a comment, a call-to-action, a review
- CAUTION: do not use it to wrap unrelated elements, just because they're close.

4. `<header>`

- Introduction
- Examples: a title, a logo, a banner, a menu nav (people use `<nav>` more), ...
- Use it once, there can be `<section>` or `<article>` inside.

The banner role is usually reserved for content such as brand, navigation and search:

```html
<header role="banner"></header>
```

5. `<nav>`

- Navigation panel between main contents
- Examples: a menu nav, a side bar

6. `<footer>`

- Ends the page, should only have ONE
- Examples: author's info, copyright, links to multiple places

7. `<aside>`

- Related content but didn't crucial to the main content.
- Examples: Sidebar, related blogs, quotes, tips, advertise panel

8. Accessibility

When there are 2 or more `<nav>`, having an `aria-label` attribute will have assistive technology users understand the difference between them:

```html
<nav aria-label="Primary"></nav>
<!-- another nav -->
<nav></nav>
```


A logo can have `aria-hidden="true"` attribute on it if it's purely decorative and screen reader user doesn't need to know it's there. Using `focusable="false"` can prevent older screen readers from being able to focus it.

```html
<svg aria-hidden="true" focusable="false"></svg>
```

Aria role `aria-current="page"` tells screen reader users that this item's link is to the current page that they're on

```html
<a aria-current="page"></a>
```

CSS hook `data-state="active"` adds decoration to the item to show the user they're already in that bit of the site (or the current URL is the prefix of that URL). It won't confuse screen reader users:

```html
<a data-state="active"></a>
```
