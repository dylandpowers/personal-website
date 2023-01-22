---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "How to Use React-Redux-Firebase"
subtitle: ""
summary: "A short tutorial on how to use this really cool package bringing together the best of React, Redux, and Firebase."
authors: []
tags: [React, Redux, Firebase, JavaScript, React-Redux, React-Redux-Firebase, useFirestoreConnect, useFirebaseConnect, Auth, Authentication, User]
categories: [React]
date: 2020-06-24T10:25:29-04:00
lastmod: 2020-06-24T10:25:29-04:00
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
In this post, I will show you how to use the [react-redux-firebase](https://www.npmjs.com/package/react-redux-firebase) package to incorporate Redux bindings for Firebase into your React project. This allows you to incorporate Firebase and Firestore data  into global state without any extra work, as well as handle auth in a super simple way. If you are unfamiliar with [React](https://reactjs.org/), [Redux](https://redux.js.org), or [Firebase](https://firebase.google.com/), please spend some time familiarizing yourself with those before jumping in.

This is **not** a beginning-to-end tutorial; rather, I will show you how the `react-redux-firebase` (RRF) package to make your app flow much simpler. For setup instructions, check out [the documentation](https://www.npmjs.com/package/react-redux-firebase).

# Auth

### Sign In

First, I will show you how this package makes auth in your application so much simpler. Without RRF, you would have to perform the sign in flow and then `dispatch` an action to the store with the user information; then, any time you needed that, you would have to use a `useSelector` hook to get that sign in information. It might look something like this:

```jsx
import React from 'react';
import firebase from 'firebase';
import { useDispatch } from 'react-redux';

export default function Component() {
	
  const dispatch = useDispatch();
	
  function signIn(email, password) {
    firebase.auth().signInWithEmailAndPassword(email, password)
      .then((userCredential) => dispatch({
        type: 'AUTH',
        user: userCredential
      })).catch((err) => console.error(err));
  }

  function signOut() {
    firebase.auth().signOut()
      .then(() => dispatch({
        type: 'AUTH',
        user: undefined
      })).catch((err) => console.error(err));
  }

  // ...
}
```

Then when you want to query for the currently signed-in user in another component, you would do something like

```jsx
const user = useSelector((state) => state.auth);
```

However, when you use the `useFirebase` hook provided by `react-redux-firebase`, you don't have to dispatch any actions:

```jsx
import React from 'react';
import { useFirebase } from 'react-redux-firebase';

export default function Component() {
	
  const firebase = useFirebase();

  function signIn(email, password) {
    firebase.signIn({
      email,
      password
    }).catch((err) => console.error(err));
  }

  function signOut() {
    firebase.signOut()
      .catch((err) => console.error(err));
  }

  // ...
}
```

That's it! No extra state for you to handle, RRF does it all internally. Then, simply query for the currently signed-in user as such:

```jsx
const user = useSelector((state) => state.firebase.auth);
```

### Registration

If you also want to store some extra data on the side, such as user `firstName` and `lastName`, you specify that when you sign up a user, as the second argument to the `createUser()` call. **This makes that information readily available in the store as part of the** `profile`.

```jsx
import React from 'react';
import { useFirebase } from 'react-redux-firebase';

export default function Component() {
	
  const firebase = useFirebase();

  function register(email, password, firstName, lastName) {
    firebase.createUser({
      email,
      password
    },
    {
      email,
      firstName,
      lastName
    }).catch((err) => console.error(err));
  }

  // ...
}
```

In this example, `email`, `firstName`, and `lastName` are all stored as part of the user's `profile`. You can access this information anywhere using the `profile` part of the store:

```jsx
const { email, firstName, lastName } = useSelector((state) => state.firebase.profile);
```

Isn't that awesome?

# Data Storage

### Cloud Firestore

Firestore, being a document-oriented database, is my preference when developing an app on top of Firebase. The Firestore JavaScript SDK provides listeners that automatically listen for changes in Firestore documents. However, if you want to listen for changes to an entire collection and update state accordingly, the code becomes quite clunky:

```jsx
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import firebase from 'firebase';

export default function Component() {
  const dispatch = useDispatch();

  const todos = useSelector((state) => state.todos);

  firebase.firestore().collection('todos')
    .onSnapshot((querySnapshot) => {
      const updatedTodos = {};
      querySnapshot.forEach((doc) => {
        updatedTodos[doc.id] = doc.data();
      });
      dispatch({
        type: 'TODOS_UPDATED',
        todos: updatedTodos
      });
    });

  // ...
}
```

However, with RRF's `useFirestoreConnect` hook, you can attach a listener that will automatically listen and unlisten as needed, *and* you can make the data a part of Redux state.

```jsx
import React from 'react';
import { useFirestoreConnect } from 'react-redux-firebase';

export default function Component() {
	
  useFirestoreConnect({
    collection: 'todos'
  });

  const todos = useSelector((state) => state.firestore.data.todos);

  // ...
}
```

In the two above examples, both `todos` variables with evaluate to the same thing; there are two obvious differences:

1. The first example involves much more code, and is much harder to read.
2. The second example requires no manual global state management, as that is managed by the hook.

This extremely useful hook also offers three other really cool features:

1. If you have a nested collection, you can choose how the listener stores the collection as part of global state. For example, if you have a nested collection `baz` which is located at `foo/bar/baz`, then without this feature you would have to select the collection as follows:

    ```jsx
    const baz = useSelector((state) => state.firestore.data.foo.bar.baz);
    ```

    However, using the `storeAs` argument, you can easily change it to anything you like:

    ```jsx
    useFirestoreConnect({
      collection: 'foo/bar/baz',
      storeAs: 'baz'
    });

    const baz = useSelector((state) => state.firestore.data.baz);
    ```

2. If you use the `ordered` property of `firestore` instead of `data`, the data will become sorted. To change how the data is sorted, pass an `orderBy` argument to the hook:

    ```jsx
    useFirestoreConnect({
      collection: 'foo/bar/baz',
      orderBy: 'createdAt',
      storeAs: 'baz'
    });

    const baz = useSelector((state) => state.firestore.ordered.baz);
    ```

3. If you just want to listen to a specific document (rather than an entire collection), that's entirely possible too. You will still have to select the individual document, but the listener will only listen to the single document, not the entire collection:

    ```jsx
    useFirestoreConnect({
      collection: 'foo',
      doc: 'bar'
    });

    const bar = useSelector((state) => state.firestore.data.foo['bar']);
    ```

Absolutely amazing.

### Real-time Database

The custom hook `useFirebaseConnect` for listening to the real-time database is very similar to the above example, with a few differences based on the general difference between the two query languages. Before RRF, you would have to fetch data from a path like this:

```jsx
import React from 'react';
import { useDispatch, useSelector } from 'react-redux';
import firebase from 'firebase';

export default function Component() {
	
  const dispatch = useDispatch();

  const todos = useSelector((state) => state.todos);

  firebase.database().ref('todos')
    .on('value', (snapshot) => {
      dispatch({
        type: 'TODOS_UPDATED',
        todos: snapshot.val()
      });
    });

  // ...
}
```

However, by using the `useFirebaseConnect` hook, we no longer have to manually manage global state. Instead, the hook manages listening and unlistening as well as updating our global state:

```jsx
import React from 'react';
import { useSelector } from 'react-redux';
import { useFirebaseConnect } from 'react-redux-firebase';

export default function Component() {
	
  useFirebaseConnect({
    path: 'todos'
  });

  const todos = useSelector((state) => state.firebase.data.todos);

  // ...
}
```

Isn't that much cleaner? You can also use special [query parameters](https://firebase.google.com/docs/reference/js/firebase.database.Query) to change how the data is sorted, among other things:

```jsx
useFirebaseConnect({
  path: 'todos',
  queryParams: ["orderByKey"]
});

const todos = useSelector((state) => state.firebase.ordered.todos);
```

Basically, anything that you could do with the standard SDK, the hook exposes to you as well.

# Conclusion

If you're developing an app with React, Redux, and Firebase, then you should be using this library to make your code much cleaner. Auth has never been simpler. Plus, the custom Firebase and Firestore hooks do so much extra leg work for you, so you can focus on writing application code. 

*Note: header image taken from [this tutorial](https://www.youtube.com/watch?v=TlpX0aMYmrk) from [Aberraouf Zine](https://www.youtube.com/channel/UCIWTUpwIzh_P73LlbX20VQA).*