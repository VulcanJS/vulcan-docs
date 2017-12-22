---
title: Callbacks
---

Vulcan uses a system of hooks and callbacks for many of its operations.

## Debugging

**If you've got the [debug package](/debug.html) enabled, a callback debugging UI is available at [http://0.0.0.0:3000/callbacks](http://0.0.0.0:3000/callbacks).**

## Adding Callback Functions

For example, here's how you would add a callback to `posts.edit.sync` to give posts an `editedAt` date every time they are modified:

```js
import { addCallback } from 'meteor/vulcan:core';

function setEditedAt (post, user) {
  post.editedAt = new Date();
  return post;
}
addCallback('posts.edit.sync', setEditedAt);
```

Vulcan's boilerplate mutations support three distinct types of callback functions, each with their own hook:

- `validate` callbacks are called to decide if an operation should run or not. 
- `sync` callbacks are called in a blocking manner before the database operation.
- `async` callbacks are called in a non-blocking manner, after the database operation. 

## Removing Callback Functions

If the callback function is named (i.e. declared using the `function foo () {}` syntax), you can also remove it from the callback using:

```js
import { removeCallback } from 'meteor/vulcan:core';

removeCallback('posts.edit.sync', "setEditedAt");
```

## Running Callback Hooks

Callbacks are run using the `Callbacks.runSync` and `Callbacks.runAsync` functions:

```js
modifier = Callbacks.run(`movies.edit.sync`, modifier, document, currentUser)
```

In each case, the **first** argument is the name of the callback hook, the **second** argument is the one iterated on by each callback function on the hook, while any remaining arguments are just passed along from one iteration to the next.

Note that for *sync* callbacks, each callback function should return the main argument to pass it on to the next function, while *async* callbacks don't need to return anything.

#### Alternative Approach

Creating new callback hooks is useful to make your own code extendable by other developers, but it's entirely optional. 