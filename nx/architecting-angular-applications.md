# Architecting Angular Application

Historically speaking, devs strongly associated `tsconfig.base.json` with `paths` mappings with monorepos - repositories containing multiple deployable applications (e.g., web app, mobile app, backend) that share libraries.

Nx supports 2 workspace models:
1. Integrated Monorepo: classic `apps/` and `/libs`
2. Standalone Application: a setup designed to build and deploy exactly one application. NX supports this setup for Angular project: `npx create-nx-workspace --preset=angular-standalone`

```
myshop
|-- e2e
|   |-- ...
|   |-- playwright.config.ts
|   \-- tsconfig.json
|-- public/
|-- src           # NOTE: if src/ at root => standalone application. If apps/ at root => monorepo
|   |-- app
|   |   |-- ...
|   |   |-- app.component.ts
|   |   |-- app.config.ts
|   |   \-- app.routes.ts
|   |-- index.html
|   |-- ...
|   \-- main.ts
|-- eslint.config.mjs
|-- jest.config.ts
|-- jest.preset.js
|-- nx.json
|-- project.json
|-- tsconfig.app.json
|-- tsconfig.editor.json
|-- tsconfig.json
\-- tsconfig.spec.json
```

First attempt to modularize application:

```
src/
|-- app/
|   |-- auth/          # Authentication feature
|   |-- products/      # Product management feature
|   |-- cart/          # Shopping cart feature
|   \-- checkout/      # Checkout feature
|-- assets/
|-- styles/
\-- ...
```

Cons:
- Folder-by-type separation.
- No machine-level boundary enforcement, rely on human discipline => big ball of mud is imminent.
- Hard to maintain as the app scales.

Second attempt: domain-specific packages/libraries

```
myshop/
|-- src/                  # Main application
\-- packages/             # Library projects
    |-- orders/           # Order Domain
    |-- checkout/         # Checkout Domain
    |-- users/            # User Domain
    |-- shipping/         # Shipping Domain
    |-- products/         # Product Domain
        |-- api/                       # API and state management
        |-- feat-product-list/         # Product listing feature
        |-- feat-product-detail/       # Product detail feature
        |-- feat-product-reviews/      # Product reviews feature
        |-- ui-product-card/           # Reusable product card component
        \-- ui-product-carousel/       # Product carousel component
    \-- ...
```

Ideal structure:

1. Application shell: routing, bootstraping, layout composition
2. Domain libraries:
- Feature libraries: `feat-*`, implement specific business logic
- UI libraries: `ui-*`, implement presentational (NOT logic!) components
- Data access libraries: `api`, implement API communication and state management
- Utils libraries: pure helper functions

Q: To add library or not to add library checklist
A:
- modify multiple libs for a single feature change? => new lib
- circular deps? => DI via HOF
- unclear ownership?
- complex deps where simple features importing from many libs
- duplication in types and utils among domains
- libs unchanged for too long? change it!

Q: When do Standalone Application upgrade to Integrated Monorepo `npx nx g convert-to-monorepo`:
A:
- Different scaling requirement? Admin and customer-facing shouln't have the same scale setup.
- Not all users need all features.
- Different parts of the app need to evolve at different speeds. E.g., admin devs faster, but customer-facing requires stability.
- Customer-facing app have no admin features => more secure

```
fe-playground/
|-- apps/
|   \-- first-example/           # Wrapper: entry points and global routing
\-- libs/                        # 95% of your code actually lives
    |-- counter/                 # The "Discussions" Domain
    |   |-- api/                 # API & Query hooks
    |   |   \-- src/
    |   |       |-- lib/         # Default name
    |   |       |   \-- get-counter.ts 
    |   |       \-- index.ts     # Public API barrel file
    |   \-- features/            # Pages, smart orchestration components, ...
    |       \-- src/
    |           |-- lib/ counter.tsx
    |           \-- index.ts
    \-- shared/                  # Global shared code
        |-- ui/                  # Global design system (Shadcn, custom UI primitives, ...)
        |   \-- button/          # Pure presentational primitives
        |       \-- src/
        |           |-- lib/ (button.tsx, button.stories.tsx)
        |           \-- index.ts
        \-- utils/               # Global helper functions (cn helpers, formatters)
```

Q: Three scopes vs Two scopes

In my playground, I had two options:

One,
```
|-- libs
|   `-- social-media
|       |-- data-access
|       |   |-- users
|       |   |-- posts
|       |   `-- comments
|       `-- ui
|           |-- users
|           |-- posts
|           `-- comments
```

Two,
```
|-- libs
|   `-- social-media                  # DOMAIN (Folder only, no code)
|       |-- posts                     # SUB-DOMAIN (Folder only, no code)
|       |   |-- data-access           # Project Name: social-media-posts-data-access
|       |   |-- features              # Project Name: social-media-posts-features
|       |   `-- ui                    # Project Name: social-media-posts-ui
|       `-- comments                  # SUB-DOMAIN
|           |-- data-access           # Project Name: social-media-comments-data-access
|           `-- ui                    # Project Name: social-media-comments-ui
```

Two is more superior:
1. Better caching: NX track changes per-project (i.e., app/lib), not per-file. Putting `posts/` and `comments/` inside the same `social-media-data-access` library will invalidate the cache for the entire library. If there are two seperate libraries: `social-media-posts-data-access` and `social-media-comments-data-access`, changing a comment selector leaves the posts state safely.
2. Strict boundary enforcement: maybe something like "posts" can import from "comments", but "comments" can't import from "posts".
3. Redux Slice Isolation: Redux slice represents a specific, isolated piece of state. Giving `posts` its own `data-access` allows itself to be snapped into `apps/social-media` root Redux store.

At my workspace, a colleague of mine proposed an even better structure:

