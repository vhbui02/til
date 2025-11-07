# Zustand Best Practices

My favorite React state management library, it's simple, robust and steep learning curve.

## Cheatsheet

### Example #1: Book management

```js
// store/bookStore.ts
import { create } from "zustand";

interface IBook {
  // properties
  amount: number;
  title: string;
  // "actions"
  addAmount: (amount?: number) => void;
  decreaseAmount: (amount?: number) => void;
  updateAmount: (newAmount: number) => void;
  updateAmountAsync: (newAmount: number) => void;
}

// NOTE: custom hook naming convention, based on the fact that Zustand uses React Hooks underneath
// NOTE: curried version, or IIFE syntax
export const useBookStore = create<IBook>()(
  (set) => ({
    amount: 10,
    title: "Sans Famille",
    addAmount: (amount: number = 1) =>
      set((prevState) => ({
        // ...prevState,
        // no need, since Zustard merge properties, rather than reassigned
        amount: prevState.amount + amount,
      }), false /* if true, change behavior to reassigned */),
    decreaseAmount: (amount: number = 1) =>
      set((prevState) => ({
        amount: prevState.amount - amount,
      })),
    updateAmount: (newAmount: number) => {
      console.log("This is a sync operation");
      set((prevState) => ({
        amount: newAmount,
      })),
    },
    updateAmountAsync: async (newAmount: number) => {
      await new Promise((resolve) => setTimeout(resolve, 500));
      set((prevState) => ({
        amount: newAmount,
      }));
    },
  })
);

// =============================================================================

// App.tsx
import { useShallow } from "zustand/react/shallow"; // new API
import { useBookstore } from "./store/bookStore"; // a selector, or function that extracts data from a the book store's state

const App = () => {
  // Zustand's Store can be used almost everywhere conforming to the rules of hooks
  // unlike Redux's Providers

  // Method 1: create multiple separated states (a.k.a atomic states)
  // This can be view as "state slice subscribing", and the components only re-render if any of the state changed, unsubscribed state slice won't have any effect
  // If subscribe too many, unnecessary re-renders can occur
  const title = useBookStore((state) => state.title);
  const amount = useBookStore((state) => state.amount);
  const addAmount = useBookStore((state) => state.addAmount);
  const decreaseAmount = useBookStore((state) => state.decreaseAmount);

  // Method 2: useShallow() hook
  const { title, amount, addAmount, decreaseAmount } = useBookStore(
    useShallow((state) => ({
      title: state.title,
      amount: state.amount,
      addAmount: state.addAmount,
      decreaseAmount: state.decreaseAmount,
    }))
  );

  return (
    <div>
      <h1>Books: {amount}</h1>
      <p>Title: {title}</p>
      <button onClick={() => updateAmount(10)}>UpdateAmount</button>
      <button onClick={addAmount}>AddAmount</button>
      <button onClick={decreaseAmount}>DecreaseAmount</button>
    </div>
  );
};
```

### Example #1: Bears' meal management

```jsx
// store/bearStore.ts
import { create } from 'zustand';

type BearFamilyMealsStore = {
  // index signature
  [key: string]: string
};

const meals = [
  'A tiny, little, wee bowl',
  'A small, petite, tiny pot',
  'A wee, itty-bitty, small bowl',
  'A little, petite, tiny dish',
  'A tiny, small, wee vessel',
  'A small, little, wee cauldron',
  'A little, tiny, small cup',
  'A wee, small, little jar',
  'A tiny, wee, small pan',
  'A small, wee, little crock',
];

const useBearFamilyMealsStore = create<BearFamilyMealsStore>()(
  (set) => ({
    papaBear: 'large porridge-pot',
    mamaBear: 'middle-size porridge pot',
    babyBear: 'A little, small, wee pot',
  })
);

const BearNames = () => {
  // NOTE: subscribe to WHOLE store, even though you only use the keys
  // const bearNames = useBearFamilyMealsStore((state) => Object.keys(state));

  // NOTE: subscribe ONLY to the keys, the values don't matter
  const bearNames = useBearFamilyMealsStore(useShallow((state) => Object.keys(state)));
  return <div>{names.join(", ")}</div>;
}

const BabyBearMeal = () => {
  useEffect(() => {
    const timer = setInterval(() => {
      useBearFamilyMealsStore.setState({
        babyBear: meals[Math.floor(Math.random() * (meals.length - 1))], // pick random
      })
    });

    // cleanup function is called when component unmounts or before re-running the effect due to dependency changes
    // use case: timer, subscriptions, event listeners management
    return () => {
      clearInterval(timer);
    }
  }, [])
}

// =============================================================================

// App.tsx
export const App = () => {
  return (
    <>
      <BabyBearMeal /> // rerender, duh, the value change
      <BearNames /> // rerender, even though the keys didn't change
    </>
  )
}
```

## Curried/IIFE vs. passing directly to `create()`

- Passing directly to `create((set) => ({ ... }))` is common in JS.
- Use curried/IIFE can improve type inference and ensure type safety. Calling `create<MyState>()` can explicitly declare the type of the store's state (i.e. `MyState`), therefore allow TS to ifer the types within state creator callback function (and any subsequent middleware)

## What is `useShallow()` hook?
