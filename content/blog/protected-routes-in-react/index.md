---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Protected Routes in React using React-Router-Dom"
subtitle: ""
summary: "A short tutorial on how to protect routes using React-Router-Dom."
authors: []
tags: [React, Redux, JWT, Auth, Security, JavaScript, JSX, React-Router-Dom]
categories: [React, Tutorials]
date: 2020-09-08T10:49:45-04:00
lastmod: 2020-09-08T10:49:45-04:00
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
If you have ever made a modern single-page web app using React, odds are you used [react-router-dom](https://reactrouter.com/) for client-side routing. In this tutorial, I will demonstrate how to protect routes such that only logged-in users can navigate to certain places. For example, you don't want every user to be able to navigate to `/dashboard`, but anybody should be able to hit `/login`. 

This tutorial assumes that you have basic familiarity with react-router-dom and how to set up a basic router. If not, please read its [quick start page](https://reactrouter.com/web/guides/quick-start) before moving on. This article also assumes that you have a simple way to determine if a user is logged in or not, whether that be through [JSON web tokens](https://jwt.io/), [Redux](https://redux.js.org/), or something else.

# Overview

A traditional React router without protected routes might look something like this:

```jsx
import React from 'react';
import {
  BrowserRouter,
  Switch,
  Route
} from 'react-router-dom';

import LoginPage from './components/LoginPage';
import DashboardPage from './components/DashboardPage';

export default function Router() {
  return (
    <BrowserRouter>
      <Switch>
        <Route to="/login" component={LoginPage} />
        <Route to="/dashboard" component={DashboardPage} />
        <Route component={DashboardPage} />
      </Switch>
    </BrowserRouter>
  );
}
```

There is an issue with this setup; if a user tries to navigate to `/dashboard`, they **will** be able to do so without logging in. Especially if the dashboard contains sensitive information, this is dangerous behavior. To remedy this, we will be replacing the `Route` components with references to our new `ProtectedRoute` components, which will redirect to a different page (in our case, `/login`) if the user is not logged in.

# Creating the Component

First, we will create the `ProtectedRoute` component. In this tutorial I will be using a functional component, but class-based components work as well. Create a new file `ProtectedRoute.js` and place the following inside of it:

```jsx
export default function ProtectedRoute({ component: Component, ...rest }) { }
```

This means that our `ProtectedRoute` will expect a `component` prop, much like how `Route` from react-router-dom takes a `component` prop. The `...rest` part refers to the rest of the props that can be passed to `ProtectedRoute` besides the `component`. 

# Importing Auth Logic

Next, we need to import auth logic into this module to determine whether or not the user is logged in. As I mentioned earlier, I use JSON web tokens, but as long as you have a function that returns a boolean indicating whether or not the user is logged in, everything will work great. At this point, our file will look like this:

```jsx
import { isAuthenticated } from '../auth/jwt';

export default function ProtectedRoute({ component: Component, ...rest }) { }
```

# Protecting the Route

These next 5 lines of code are where the real magic happens. We will be using react-router-dom's [Redirect](https://reactrouter.com/web/api/Redirect) component to redirect to a universally accessible page if the user is not logged in. Otherwise, we will return its [Route](https://reactrouter.com/web/api/Route) component, which is what you see in a regular React router. 

We will pass the `...rest` parameter to the `Route` component, which means that everything that we pass to `ProtectedRoute` (except for the `component` prop) will then be passed to the `Route` component. We will take advantage of `Route`'s `render` prop to make it render the component passed in to our protected route:

```jsx
<Route {...rest} render={(props) => <Component {...props} />} />
```

Of course, we will only return this route if the user is logged in. If not, we will return a `Redirect` to the login page. After adding the logic and the imports in the file, it should look like this:

```jsx
import React from 'react';
import { Redirect, Route } from 'react-router-dom';

export default function ProtectedRoute({ component: Component, ...rest }) {
  return isAuthenticated() ? (
    <Route {...rest} render={(props) => <Component {...props} />} />
  ) : (
    <Redirect to="/login" />
  );
}
```

Alternatively, if you do not want to *always* redirect to the login page, consider adding adding a prop like `redirectRoute` to the component:

```jsx
export default function ProtectedRoute({ component: Component, redirectRoute, ...rest }) {
  // ...
}
```

And then the redirect component would read `<Redirect to={redirectRoute} />`. 

# Putting it All Together

Now, anytime we want to protect a route, we can use our `ProtectedRoute` component instead of the provided `Route` component. As a result, the router shown in the overview would now look like this:

```jsx
import React from 'react';
import {
  BrowserRouter,
  Switch,
  Route
} from 'react-router-dom';

import LoginPage from './components/LoginPage';
import DashboardPage from './components/DashboardPage';
import ProtectedRoute from './utils/ProtectedRoute';

export default function Router() {
  return (
    <BrowserRouter>
      <Switch>
        <Route to="/login" component={LoginPage} />
        <ProtectedRoute to="/dashboard" component={DashboardPage} />
        <ProtectedRoute component={DashboardPage} />
      </Switch>
    </BrowserRouter>
  );
}
```

Voilà! You now have the ability to make protected routes in React. Go protect those web apps!