# Redux Learning

An app has 3 parts:
- State: e.g., a counter value
- Action: code that cause an update to the state when something happens e.g., a button click
- View: the UI.

## Actions

A JS object that has a `type` field. An action is an event that describes something happened in the app.

```ts
const addTodoAction = {
  // descriptive name
  // "todos" is domain name, "todoAdded" is the event name
  type: 'todos/todoAdded',

  // additional information about the event
  payload: 'Buy milk'
}
```

## Action Creators

A function that return the [Actions](#actions).

```ts
const addTodo = text => {
  return {
    type: 'todos/todoAdded',
    payload: text
  }
}
```

## Reducers

A function, receives `state` and `action` object, do some calculation, then return a new state.

- Must be a pure function.
- Not allowed to modify existing state.
- `state` and `action` are everything needed to calculate the new state.

It acts like event listeners, when it hears an action it's interested in, it update the state in response.

```ts
const initialState = []

const todosReducer = (state, action) => {
  // check to see if this reducer care about the received action
  if (action.type === 'todos/todoAdded') {
    // if so, make an "immutable update" (i.e., create a copy with modification)
    return [...state, action.payload]
  }
  
  // otherwise return the existing state
  return state
}
```

Redux reducers reduce a set of actions (over time) into a single state. With `Array.reduce()` it happens all at once, with Redux, it happens over the lifetime of the running app.

```
const actions = [
  { type: 'todos/todoAdded', payload: 'foo' }
  { type: 'todos/todoAdded', payload: 'bar' }
  { type: 'todos/todoAdded', payload: 'baz' }
]

const initialState = []

const finalState = actions.reduce(todosReducer, initialState)
console.log(finalState); // ['foo', 'bar', 'baz']
```

## Store

The object where Redux app state lives in.

```
import { configureStore } from '@reduxjs/toolkit';

const store = configureStore({ reducer: todosReducer });

console.log(store.getState()); // []
```

## Dispatch

Update the state with `store.dispatch()`. Calling it similar to triggering an event.

```ts
store.dispatch({ type: 'todos/todoAdded' });
console.log(store.getState());
```

## Selectors

Functions to extract information from a store state value.

```ts
const selectCounterValue = state => state.value;
const currentValue = selectCounterValue(store.getState());
console.log(currentValue); // 2
```

## Procedures

1. A Redux store is created using a root reducer function.
2. The store calls the root reducer once and save the return value as its initial `state`
3. When the UI is first rendered, UI components access the current state of the Redux store and use that data to decide what to render.
4. UI components also subscribe to future store updates so they can know if the state has changed.
5. Something happens in the app, such as a user clicking a button.
6. The app code dispatch an action to the Redux store, like `dispatch({ type: "todos/todoAdded" });`
7. The store runs the reducer function again with the previous `state` and the current `action` then saves the return value as the new `state`
8. The store notifies all of the UI components that subscribed to the store that the store has been updated
9. Each UI component that need data from store checks to see if the parts of the state they need have changed.
10. Each component that sees its data has changed, forces a re-render with the new data, so it can update what's shown on the screen.
