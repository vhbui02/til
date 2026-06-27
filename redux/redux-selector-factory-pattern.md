# Redux Selector Factory Pattern

> [!CAUTION]
> New version of Redux Toolkit prefers configuring cache size or using `weakMapMemoize`.

```
// features/orders/ordersSlice.js
import { createSlice, createSelector, lruMemoize } from '@reduxjs/toolkit';

export const selectOrdersByCategory = createSelector(
  [
    (state) => state.orders, // or "selectAllOrders"
    (state, category) => category,
  ],
  (orders, category) => orders.filter((order) => order.category === category),
  {
    memoize: lruMemoize,
    memoizeOptions: {
      maxSize: 10
    }
  }
)
```

Comparison between modern way vs selector factory:

```
| Feature | Configuring Cache Size (maxSize / lruMemoize) | Selector Factory (makeSelector + useMemo) |
| :--- | :--- | :--- |
| Component Boilerplate | Minimal: Use useSelector(state => selectItem(state, id)) directly. | High: Requires useMemo(() => makeSelector(), []) in every component. |
| Cross-Component Sharing | Yes: Two different components requesting the same argument share the cached result. | No: Each component instance has its own isolated cache. |
| Risk of Developer Error | Low: Impossible to misuse in the component layer. | High: Forgetting useMemo silently destroys memoization on every render. |
| Cache Capacity | Shared global capacity (e.g., maxSize: 10 shared across all components). | Guaranteed 1 cache slot per component instance. |
| Garbage Collection | Evicts oldest items when maxSize limit is reached (or auto-GC with weakMapMemoize). | Cache is garbage collected when the component unmounts. |
```

=> Use Cache size configuration as default, use Selector Factory when working with legacy codebases where custom cache size options are unavailable and your components render unpredictable number of instances.

TL;DR: a HOF that creates and returns a brand-new instance of a memoized selector created via `createSelector()`

```
// features/orders/ordersSlice.js
import { createSlice, createSelector } from '@reduxjs/toolkit';

// Selector Factory Function
// TIPS: HOF naming convention tend to be "makeSomething()"
export const makeSelectOrdersByCategory = () =>
  createSelector(
    [
      (state) => state.orders,
      (state, category) => category
    ],
    (orders, category) => {
      console.log(`Calculating items for category: ${category}`);
      return orders.filter((order) => order.category === category);
    }
  );

// components/CategoryList.jsx
import React, { useMemo } from 'react';
import { useSelector } from 'react-redux';
import { makeSelectOrdersByCategory } from '../features/orders/ordersSlice';

function CategoryList({ category }) {
  // the selector instance is created ONCE when the components mounts and
  // persists acress re-renders 
  // without it, calling the HOF will create a brand-new selector on every render
  const selectOrdersByCategory = useMemo(makeSelectOrdersByCategory, []);

  // 2. Call the private selector instance
  const categoryOrders = useSelector((state) =>
    selectOrdersByCategory(state, category)
  );

  return (
    <div>
      <h3>Category: {category}</h3>
      <ul>
        {categoryOrders.map((order) => (
          <li key={order.id}>{order.name}</li>
        ))}
      </ul>
    </div>
  );
}

export default CategoryList;
```
