# NX golden rules for utils

Never create a single, massive `utils` or `shared/utils`.
- Becoming massive is imminent.
- Destroy the benefits of NX's dependency graph.

Favors granular utility libraries.

E.g., if `products` app imports a math helper from a massive `shared/utils` library, any changes to that library will cause retested+rebuilt. A change to the string helper by other teammates will trigger it.

```
packages/
|-- products/
|   |-- data-access/
|   |-- feat-product-list/
|   \-- util-sku-parser/     <-- Highly specific to products
\-- shared/
    |-- util-date/           <-- Global, but scoped to dates
    \-- util-math/           <-- Global, but scoped to math
```
