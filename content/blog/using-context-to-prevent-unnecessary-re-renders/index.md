---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Using Context to Prevent Unnecessary Re Renders"
subtitle: ""
summary: "Using React's Context API to prevent unnecessary re-renders in React applications."
authors: []
tags: [React, Frontend, Javascript, JSX, Context, Hooks, Components]
categories: [React]
date: 2021-05-09T22:49:47-04:00
lastmod: 2021-05-09T22:49:47-04:00
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
With the rising popularity of React, we are in an era of the most responsive user interfaces we have ever seen. React uses a virtual DOM, and for every render it runs calculations to determine which parts of the actual DOM need to get updated. This means that React *only re-renders* components that need to be re-rendered, and nothing more.

However, that doesn't mean we shouldn't introduce optimizations of our own! Too many re-renders can lead to performance issues, so we should optimize wherever we can. One common pitfall I see in React applications is what I call *messenger components*, which are components that pass some props from a parent to a child, but do not actually do anything with those props. For example, consider the following three components.

```jsx
fun ction Parent(props) {
  const foo = 1;
  const bar = 2;

  return (
    <Child foo={foo} bar={bar} />
  );
}
```

```jsx
function Child(props) {
  return (
    <>
      <p>Foo is {props.foo}</p>
      <Grandchild bar={props.bar} />
    </>
  );
}
```

```jsx
function Grandchild(props) {
  return (
    <p>Bar is {props.bar}</p>
  );
}
```

In this situation, `Child` receives both `foo` and `bar` as props, but only truly "cares" about the value of `foo`; `Grandchild` cares about the value of `bar`. In this case, `Child` is a messenger component for the `bar` prop. `Child` should only re-render when the value of `foo` updates, but in this case it will also re-render when `bar` updates. How can we fix this?

# The Context API

Using React's [Context API](https://reactjs.org/docs/context.html), we can broadcast changes to values, and only components that directly subscribe to that piece of context will receive the new value. This solves the problem of messenger components re-rendering unnecessarily, because we no longer need to pass props down - we can simply directly subscribe to the value in `Grandchild`. To demonstrate this, I will add [refs](https://reactjs.org/docs/glossary.html#refs) to show the number of re-renders for each component before and after adding context.

# Before

Using the current system of prop drilling, we can see how `Child` will re-render every time either prop is updated. If you like, use [create-react-app](https://github.com/facebook/create-react-app) to bootstrap an app to see the renders in action.

```jsx
function Parent(props) {
  const [foo, setFoo] = useState(0);
  const [bar, setBar] = useState(0);

  function incrementFoo() {
    setFoo((prev) => prev + 1);
  }

  function incrementBar() {
    setBar((prev) => prev + 1);
  }

  return (
    <>
      <button onClick={incrementFoo}>Increment Foo</button>
      <button onClick={incrementBar}>Increment Bar</button>
      <Child foo={foo} bar={bar} />
    </>
  );
}
```

```jsx
function Child(props) {
  const renders = useRef(0);

  return (
    <>
      <p>Foo (in Child): {props.foo}</p>
      <p>Child renders: {renders.current++}</p>
      <Grandchild bar={bar} />
    </>
  );
}
```

```jsx
function Grandchild(props) {
  const renders = useRef(0);

  return (
    <>
      <p>Bar (in Grandchild): {props.bar}</p>
      <p>Grandchild renders: {renders.current++}</p>
    </>
  );
}
```

If you run this application, you will notice that the number of renders for the `Child` and `Grandchild` components will always be the same. However, since `Child` doesn't need to update when `bar` updates, the number of re-renders shouldn't be the same for the two components. 

But wait, why don't we just memoize `Child` so that it only re-renders when `foo` updates? Beware of this approach, because it is only half of the solution! Memoizing `Child` to not re-render when `bar` is updated leaves *nothing to re-render `Grandchild`,* so this will result in unintended consequences: `Grandchild` would only re-render when `foo` is updated.

# After

Using a combination of memoization and the Context API, we can prevent `Child` from re-rendering unnecessarily. The following application demonstrates this:

```jsx
const BarContext = createContext(null);

function Parent(props) {
  const [foo, setFoo] = useState(0);
  const [bar, setBar] = useState(0);

  function incrementFoo() {
    setFoo((prev) => prev + 1);
  }

  function incrementBar() {
    setBar((prev) => prev + 1);
  }

  return (
    <BarContext.Provider value={bar}>
      <button onClick={incrementFoo}>Increment Foo</button>
      <button onClick={incrementBar}>Increment Bar</button>
      <MemoizedChild foo={foo} />
    </BarContext.Provider>
  );
}
```

```jsx
function Child(props) {
  const renders = useRef(0);

  return (
    <>
      <p>Foo (in Child): {props.foo}</p>
      <p>Child renders: {renders.current++}</p>
      <Grandchild />
    </>
  );
}

const MemoizedChild = memo(Child);
```

```jsx
function Grandchild(props) {
  const renders = useRef(0);
  const bar = useContext(BarContext);

  return (
    <>
      <p>Bar (in Grandchild): {bar}</p>
      <p>Grandchild renders: {renders.current++}</p>
    </>
    );
}
```

As you can see, we made two updates. First, we used a piece of context to hold the reference to `bar`, and we have `Grandchild` subscribe to that piece of context using the `useContext` hook. However, this is only half of the equation; if we do not memoize `Child`, then anytime `bar` is updated in `Parent`, **`Child` will re-render.** In order to prevent this, we use the [memo](https://reactjs.org/docs/react-api.html#reactmemo) function to create a memoized version of the `Child` component that will only re-render when the props change. 

And voila! We have successfully eliminated unnecessary messenger re-renders with the Context API and some simple memoization. 

I would be remiss not to say that for most applications, re-renders don't cause much of a performance issue because most applications don't involve a lot of front-end computation. However, for applications with a lot of animations or drag-and-drops, it's more than worth it to eliminate unnecessary re-renders wherever you can. Happy coding!
