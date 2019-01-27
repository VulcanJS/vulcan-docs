---
title: Callbacks
---

Vulcan uses a system of hooks and callbacks for many of its operations.

A **hook** is a specific point in the code where **callbacks** can be attached and executed. These callbacks are middleware-like functions that take in an argument, then pass it on to the next callback attached to the same hook. 

They are a handy way to inject your own code into other, pre-existing operations without having to modify the code of these operations directly. 

Callback functions can take more than one arguments, but the first argument is special in that - for sync callbacks – it will be updated each time to reflect the return value of each successive function, letting you progressively iterate over a value or object. 

## Debugging

**If you've got the [debug package](/debug.html) enabled, a callback debugging UI is available at [http://localhost:3000/debug/callbacks](http://localhost:3000/debug/callbacks).**

## Collection Callbacks

The most common place to use callbacks is after CRUD operations on your collections (in other words creating or updating a document).

### Naming

Collection callbacks follow a common naming scheme: 

- `typename.operation.type`

For example, the `async` callback of the `post` type's `update` operation will be available under the name: 

- `post.update.async`

In addition, you can use global callbacks that will apply to all e.g. `update` operations: 

- `*.update.async`

### Arguments

Functions added to the default collection callbacks will be called with a set of predetermined arguments.

`validate`, `before`, and `after` functions take two arguments, an `iterator` which should be returned by the function, and a `properties` object; while `async` callbacks only take a `properties` argument and don't need to return anything. 

#### Common Properties

All collection callbacks (as well as `onCreate`, `onUpdate`, etc. callbacks) take a `properties` object with these properties:

```js
{ 
  document,
  currentUser,
  collection, 
  context, 
}
```

#### Create

- `validate`: `(validationErrors, properties) => { ... }`
- `before`: `(document, properties) => { ... }`
- `after`: `(document, properties) => { ... }`
- `async`: `(properties) => { ... }`

#### Update

For all `update` callback functions, `properties` has the following additional properties:

```js
{
  data, // the raw mutation data sent by the client
  oldDocument, // the original document before the mutation
}
```

Note that for `validate` and `before`, `document` is a simulated preview of the new document post-mutation, while for `after` and `async` it will have been fetched fresh from the database. 

- `validate`: `(validationErrors, properties) => { ... }`
- `before`: `(data, properties) => { ... }`
- `after`: `(document, properties) => { ... }`
- `async`: `(properties) => { ... }`

#### Delete

- `validate`: `(validationErrors, properties) => { ... }`
- `before`: `(document, properties) => { ... }`
- `async`: `(properties) => { ... }`

Note: delete mutations do not have an `after` callback, as they return the document that was just deleted unmodified.

## Adding Callback Functions

For example, here's how you would add a callback to `post.update.before` to give posts an `updatedAt` date every time they are modified:

```js
import { addCallback } from 'meteor/vulcan:core';

function setUpdatedAt (data) {
  data.updatedAt = new Date();
  return data;
}
addCallback('post.update.before', setUpdatedAt);
```

Vulcan's boilerplate mutations support four distinct types of callback functions, each with their own hook:

- `validate` callbacks are called to decide if an operation should run or not. 
- `before` callbacks are called in a blocking manner before the database operation.
- `after` callbacks are called in a blocking manner after the database operation.
- `async` callbacks are called in a non-blocking manner, after the database operation. 

## Removing Callback Functions

If the callback function is named (i.e. declared using the `function foo () {}` syntax), you can also remove it from the callback using:

```js
import { removeCallback } from 'meteor/vulcan:core';

removeCallback('post.update.before', "setUpdatedAt");
```

## Running Callback Hooks

Callbacks are run using the `runCallbacks` and `runCallbacksAsync` functions:

```js
modifier = runCallbacks({ name: `movie.update.before`, iterator: data, properties: { document, currentUser }});
```

In each case, the `name` argument is the name of the callback hook, the `iterator` argument is the one iterated on by each callback function on the hook, while `properties` are just passed along from one iteration to the next.

Note that for *sync* callbacks, each callback function should return the main argument to pass it on to the next function, while *async* callbacks don't need to return anything.

## Registering Callbacks

While you don't need to register new callback hooks to use them (as long as you do run them), registering a callback makes it easier to document and use it:

```js

import { registerCallback } from 'meteor/vulcan:lib';

registerCallback({
  name: `post.create.validate`, 
  description: `Validate a document before insertion (can be skipped when inserting directly on server).`,  
  arguments: [{document: 'The document being inserted'}, {currentUser: 'The current user'}, {validationErrors: 'An object that can be used to accumulate validation errors'}], 
  runs: 'sync', 
  returns: 'document',
});
```

It takes the following arguments: 

- `name`: the name of the callback.
- `description`: a description of what the callback does. 
- `arguments`: an array of `{name: description}` objects describing the callback functions' arguments. 
- `runs`: whether the callback runs in a `sync` or `async` manner. 
- `returns`: what the callback is expected to return (for `sync` callbacks).

## Error Handling

Pass `break` on your `Error` object to cause the chain of callbacks to stop when using a *sync* callback.

```
function mySyncCallback() {
  try {
    const value = Posts.callingAFunctionThatDoesNotExistThisThrowsAnError();
  } catch (error) {
    console.error(error);
    // Cause Vulcan to halt execution of resolver and throw the error to Apollo
    error.break = true;
    throw error;
  }
}

addCallback('post.create.before', mySyncCallback);
```
