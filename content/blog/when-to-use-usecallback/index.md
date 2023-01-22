---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "When to Use useCallback"
subtitle: ""
summary: "When to actually use the useCallback function in React"
authors: []
tags: [React, Frontend, JavaScript, Hooks, useCallbackl]
categories: [React]
date: 2021-08-15T21:10:54-04:00
lastmod: 2021-08-15T21:10:54-04:00
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
I've worked with React a lot over the past few years. The more I work with it, the more I notice common mistakes that can slow down rendering as they pile up. Granted, in small applications (like most side projects), usually the rendering slowness will go unnoticed due to the size of the application. However, for larger applications, thinking about rendering speed becomes important.

One issue I see often is wrapping all function declarations in a component inside of a [useCallback](https://reactjs.org/docs/hooks-reference.html#usecallback) hook. As per the React docs:

> `useCallback` will return a memoized version of the callback that only changes if one of the dependencies has changed.

Essentially, without `useCallback`, the function will be recreated on every render. It's important to remember that in JavaScript, functions are first-class objects just like a normal `Object` is. This essentially just means that the function can be stored as a variable, passed as an argument to a function, and returned from a function.

# Without `useCallback`

Let's say we have a function `createUser` that is going to create a user by making an API call. The user details are stored in pieces of state within the component.

```jsx
// SignupPage.jsx
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');

const createUser = () => {
  UserAPI.createUser(email, password);
  redirectToHome();
};
```

We will return some JSX that will render the `input` elements, so we don't need to rely on other components to render the input form. Whenever a piece of state changes, React will re-render the component. This will recreate (among other things) all functions not wrapped in a `useCallback`. 

This means that the `createUser` function above is created once for each render. This can become a larger performance hit in larger applications, but I'll show in the next section why it isn't the end of the world for this case.

# With `useCallback`

Alternatively, we can wrap the function in a `useCallback` hook provided by the React library. This means that the function will not be re-evaluated unless something in the dependency array changes. Assuming that `UserAPI` and the `redirectToHome` function are both created outside of the component, we just need to declare `email` and `password` as dependencies.

```jsx
// SignupPage.jsx
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');

const createUser = useCallback(() => {
  UserAPI.createUser(email, password);
  redirectToHome();
}, [email, password]);
```

This means that, unless the `email` or `password` state changes, we will not recreate the `createUser` function. For example, if we had some `showInfoModal` state that would show some information about signing up, we could change that state without recreating the `createUser` function. 

This seems like an improvement, right? Consider the fact that the above code is actually identical to the following:

```jsx
// SignupPage.jsx
const [email, setEmail] = useState('');
const [password, setPassword] = useState('');

const nonMemoizedCreateUser = () => {
  UserAPI.createUser(email, password);
  redirectToHome();
};

const createUser = useCallback(nonMemoizedCreateUser, [email, password]);
```

So, the original function is actually still created, and we are doing even more work than before! In this instance, it seems quite wasteful. However, there are situations where `useCallback` is worth the extra cost.

# Use it With Memoized Components

The best use case for `useCallback` is when creating memoized components; that is, components wrapped in an extra `React.memo()` function. This means that the component will not re-render unless the props *actually* change (or, of course,
component-held state changes). 

Consider a situation where we have a separate `Button` component which will call the passed-in function when clicked. This component has a single prop representing the `createUser` function.

```jsx
import React, { memo } from 'react';

const Button = (props) => {
  return (
    <button onclick={props.createUser}>Create User</button>
  );
};

export default memo(Button);
```

This means that the `Button` component will not re-render unless the props change (React by default defines a "change" by shallowly comparing the current props to the previous props). If in the `SignupPage` component we do not wrap the function in a `useCallback`, then the reference will change every time, causing the `Button` component to re-render every time that the `SignupPage` component re-renders. However, if we wrap the prop in a `useCallback`, then we have much more fine-grained control over when the `Button` component re-renders.

---

When making decisions on whether or not to use `useCallback`, there are always several factors at play. However, when relying on memoized components, wrapping function props in `useCallback`s is always a safe bet.