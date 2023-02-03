---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Essential NPM Packages for React Beginners"
subtitle: ""
summary: "A collection of NPM packages which comprise an excellent toolset for beginners."
authors: []
tags: [React, Frontend, Javascript, JSX, NPM]
categories: [React]
date: 2021-08-08T16:20:12-04:00
lastmod: 2021-08-08T16:20:12-04:00
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
When I was first starting out in React development, I had little to no knowledge about the ecosystem in general. Not knowing the necessary tools available in the ecosystem definitely led to increased time-to-completion for personal projects. In this post, I'll discuss NPM packages that I use every day in my personal React projects which have sped up development time drastically, and are great for beginners.

# [create-react-app](https://www.npmjs.com/package/create-react-app)

This is the #1 package out there for bootstrapping React applications. It allows you to create scaffolds for React applications with a simple command: `create-react-app <project name>`. This package is maintained by Facebook, so you can be sure that it will always contain the most recent optimizations.

# [antd](https://www.npmjs.com/package/antd)

The `antd` package, short for Ant Design, is a library of React components created by [Ant Financial](https://www.antgroup.com/en). Creating UI components can be an extremely time consuming part of development, and self-created components don't make for a great user experience if you don't have great design skills (like myself). 

`antd` provides simple, out-of-the-box components for everything from lists to tables to rating systems. Everything is standardized which makes for a *great* user experience, and you can even override the CSS variables using something like [craco](https://www.npmjs.com/package/craco). I've made several websites with `antd` components and I always have a great time using it.

# [styled-components](https://styled-components.com/)

`styled-components` is a library for styling React components (or standard HTML elements) directly in your component file. By default, it exports a function `styled` which accepts a React component as an argument and applies extra styling on top of it. For example, we can add extra styles to `antd`'s `Button` component like so:

```jsx
import { Button } from 'antd';
import styled from 'styled-components';

const PaddedButton = styled(Button)`
  padding: 10px;
`;
```

There is a short-hand for applying the styling on HTML elements where you can just use a `.`:

```jsx
import styled from 'styled-components';

const RedText = styled.p`
  color: red;
`;
```

I love `styled-components` because I can style the components in the way I need without ever leaving the component file. In my opinion, the close coupling of component's styles to their state management makes for a simpler development workflow. 

**Note:** be sure to always define your styled components *outside* of the component that uses them, so that they aren't re-computed on every render.

# [react-router-dom](https://reactrouter.com/)

This is simply the easiest and best routing package out there. It gives you a set of navigational components that you can use to declare different routes in your application like the main page, a `/login` page, a `/profile` page, etc. You declare your different routes with `Route` components, and then designate which component they should render:

```jsx
import { Route, Switch, Router } from 'react-router-dom';
import LoginPage from './LoginPage';

export default function App(props) {
  return (
    <Router>
      <Switch>
        <Route exact path="/login" component={LoginPage} />
      </Switch>
    </Router>
  );
}
```

I've also written [separate post](https://dylanpowers.me/post/protected-routes-in-react/) on how to make protected routes using this package.

# [firebase](https://www.npmjs.com/package/firebase)

Firebase is more just than a package - it provides tools for everything that basic applications need like authentication, storage, and much more. It provides a declarative and simple API that makes app development incredibly simple. 

To use this package, you'll need to create a firebase account and project. After that, you can set up authentication as well as a real-time datastore or a collection-like datastore similar to [MongoDB](https://www.mongodb.com/).

# [axios](https://www.npmjs.com/package/axios)

Axios is a simple HTTP client for making API requests from the browser. It's also promise-based which means you can add success callbacks with ease.

# [classnames](https://www.npmjs.com/package/classnames)

Classnames is an incredible simple yet extremely powerful library. It does one thing and it does it well - conditionally combining CSS class names based on a set of criteria. It exports a function which takes a single object argument where the keys are the class names and the values are booleans representing whether or not to apply the class name. Here's a simple example where something should have the `highlighted` class if a piece of state is `true`:

```jsx
import classNames from 'classnames';

export default function List(props) {
  const [isHighlighted, setIsHighlighted] = useState(false);

  const listItemClassName = classNames({
    'list-item': true,
    'highlighted': isHighlighted
  });
}
```

---

With these packages, you can supercharge your React development. Go out and develop!