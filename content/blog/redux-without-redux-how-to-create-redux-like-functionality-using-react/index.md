---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Redux Without Redux: How to Create Redux-like Functionality Using React"
subtitle: ""
summary: "Create a simple Redux-like module using pure React"
authors: []
tags: [React, Redux, Javascript, Frontend, Web, Mobile, Hooks, React-Redux]
categories: [React]
date: 2021-02-02T21:03:31-05:00
lastmod: 2021-02-02T21:03:31-05:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: true

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---
When it comes to frontend state management libraries, it doesn't get much better than good ol' [Redux](https://redux.js.org/). It allows you to define one or more reducers, dispatch updates to a global state, and subscribe to only the parts of state that you care about in each a component. 

However, as I have continued to incorporate Redux into smaller personal projects, I have found some of its features to be superfluous to my needs. For example, for small projects that make use of Ajax, I sometimes prefer to separate the global state from the Ajax calls. This makes something like [redux-thunk](https://github.com/reduxjs/redux-thunk) unnecessary, meaning that I would not make use of any Redux middleware (fun fact: the redux-thunk library is only [14 lines of source code](https://github.com/reduxjs/redux-thunk/blob/master/src/index.js)). 

Fortunately, implementing a basic Redux is *much* simpler than you might think! We just need a custom reducer and a couple of React hooks. This tutorial will show you how to create a simple global state, along with a dispatcher and a way to update that global state. We will be using functional components in this tutorial, and we will create a simple counter which can be incremented and decremented. Let's get started!

# The Reducer

For starters, a reducer is just a function which accepts a current state and an action and then based on that action, returns a new state. That's it. However, without some active management, you would have to keep track of the current state and pass it to the reducer every time you wanted to dispatch an action.

Fortunately, React already covers a lot of the ground for us with the [`useReducer`](https://reactjs.org/docs/hooks-reference.html#usereducer) hook. It accepts two arguments: a reducer and an initial state (it also optionally accepts a lazy state initialization function). The reducer must have the signature described above:

```jsx
function reducer(state, action) {
  return state; // some new state
}
```

To begin with, we'll define our reducer in its own file. We will also define the action types for incrementing, decrementing, and setting the counter:

```jsx
// reducer.js
const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';
const SET_COUNT = 'SET_COUNT';

export default function counter(state, action) {
  switch (action.type) {
    case INCREMENT:
      return state + 1;
    case DECREMENT:
      return state - 1;
    case SET_COUNT:
      return action.count;
    default:
      throw new Error(`Action type ${action.type} not recognized`);
  }
} 
```

We'll also want to make some *action creators,* which simplify the process of constructing an action from outside of the scope of the reducer. Actions are just objects which have a `type` property and, optionally, other properties. To avoid having to create the action object inside of a component, we define certain action creators in the reducer:

```jsx
// reducer.js
export function increment() {
  return {
    type: INCREMENT
  };
}

export function decrement() {
  return {
    type: DECREMENT
  };
}

export function setCount(count) {
  return {
    type: SET_COUNT,
    count
  };
}
```

With the reducer done, we can now begin to make our Redux functionality using a couple useful hooks.

# Incorporating the Reducer

The `useReducer` hook returns a `dispatch` function and a reference to the current state. The `dispatch` function takes care of automatically passing in the current state to the reducer, although we still have to supply the action (this emulates Redux behavior). 

In our top-level component where we want the global state to live, we declare our `dispatch` function and our global state using this hook:

```jsx
// TopLevelComponent.jsx
import React, { useReducer } from 'react';
import reducer from './reducer';

export default function TopLevelComponent(props) {
  const [count, dispatch] = useReducer(reducer, 0);

  return (
    <p>The current count is {count}.</p>
  );
}
```

Then, any time an action is dispatched using the `dispatch` function returned above, the global `count` will be updated! Now this is beginning to look a little like Redux. However, if the `dispatch` function were simply passed down through props, you might think that this looks like plain React: passing down callbacks to update state in a parent component. And you would be right. Using another hook, we can make this app even more Redux-like.

# Making `dispatch` Available Everywhere

To further emulate Redux, we should make the `dispatch` function available from any child component without directly passing it down through props. In the parent component, we can provide a context whose value is the `dispatch` function. In order to be able to access this context later, we will declare it in the reducer file:

```jsx
// reducer.js
import { createContext } from 'react';

export const CounterContext = createContext(null);
```

Then, in our top-level component, we will wrap all children in a context provider that will provide the `dispatch` function to all child components:

```jsx
// TopLevelComponent.jsx
import React, { useReducer } from 'react';
import reducer, { CounterContext } from './reducer';

export default function TopLevelComponent(props) {
  const [count, dispatch] = useReducer(reducer, 0);

  return (
    <CounterContext.Provider value={dispatch}>
      <p>The current count is {count}.</p>
    </CounterContext.Provider>
  );
}
```

Now, any child component will have access to the `dispatch` function, which they can use to update the `count` state globally! If you wish to access `count` in child components as well, it would be trivial to make another context to provide that value anywhere.

# Dispatching Actions from Child Components

Now, for the pièce de résistance! By using the [`useContext`](https://reactjs.org/docs/hooks-reference.html#usecontext) hook, we can access the value of the created context from any child component and `dispatch` actions to the global state, just like Redux:

```jsx
// ChildComponent.jsx
import React, { useContext } from 'react';
import { CounterContext, increment, decrement } from './reducer';

export default function ChildComponent(props) {
  const dispatch = useContext(CounterContext);

  return (
    <div>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
}
```

As you can see, now we can access the `dispatch` function from anywhere to dispatch actions to the global state without having to pass down callbacks as props. 

---

As you can see, we have now covered all of our bases in terms of basic Redux functionality. We can:

1. Define a reducer which updates a global state
2. Dispatch actions to update that global state
3. Access the `dispatch` function from any child component

For simple projects, this quasi-Redux implements a lot of the basic functionality without having to incorporate the whole library. Thanks for reading 😄!