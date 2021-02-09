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

```
const Movies = createCollection({

  collectionName: 'Movies',

  //...

  callbacks: {
    create: {
      validate: [(validationErrors, properties) => { return validationErrors; }],
      before: [(document, properties) => { return document; }],
      after: [(document, properties) => { return document; }],
      async: [(properties) => { /* no return value */ }],
    }
    update: {
      validate: [(validationErrors, properties) => { return validationErrors; }],
      before: [(data, properties) => { return data; }],
      after: [(document, properties) => { return document; }],
      async: [(properties) => { /* no return value */ }],
    }
    delete: {
      validate: [(validationErrors, properties) => { return validationErrors; }],
      before: [(document, properties) => { return document; }],
      after: [(document, properties) => { return document; }],
      async: [(properties) => { /* no return value */ }],
    }
  },

});
```

Note that the `update.before` callback functions take `data` as iterator argument, and not `document`. 

Each hook takes an array of callback functions. Note that you don't need to specify all hooks:

```
const Movies = createCollection({

  collectionName: 'Movies',

  //...

  callbacks: {
    create: {
      async: [ sendPostCreateNotification, createNewComment ],
    }
    update: {
      before: [ addAuthorId ],
      async: [ sendPostUpdateNotification ],
    }
  },

});
```

### Callback Hooks

The four hooks correspond to the following events in the lifecycle of a mutator:

- `validate`: when the data received from the client undergoes validation. Note that this can be skipped, for example if you are calling the mutator from the server and already trust the data. 
- `before`: a hook that can modify the data before it is submitted to the database.
- `after` a hook that runs after the database operation; but before the document is returned to the client.
- `async`: a hook that runs after the database operation in an asynchronous manner, meaning it will not return anything or affect the mutation's return value; but also won't hold back the mutation from returning while it executes. 

### Arguments

#### `validationErrors`

`validationErrors` is an array that starts empty and then gets populated by each successive validation callback. If that array is not empty, the mutation will throw an error regrouping all the issues encounted during validation. 

#### `document`

The `document` argument is the document being inserted or updated in the database.

#### `data`

`data` is a modifier object used to figure out what changes to perform in the database. It can be equal to the entire `document`, or simply contain a subset of the fields to update. 

Additionally, any field set to `null` in the `data` object will be deleted from that document in the database.

#### `properties`

The `properties` object contains properties that are optionally passed to each callback function, but are not affected by said functions (and should generally not be mutated by them). 

All `properties` objects for all callbacks of all operations share these common properties: 

- `currentUser`: the user performing the operation (can be `undefined`).
- `collection`: the collection affected by the operation. 
- `schema`: the collection schema. 
- `context` the GraphQL context of the mutation. 

In addition, each operation receives a few additional specific properties.

##### `create`

- `document`: the document to create received from the client.
- `originalDocument`: the original document before going through callbacks.

##### `update`

- `data`: the object received from the client (that will be inserted in the database).
- `originalData`: a copy of the original `data` (before it is mutated by callbacks).
- `document`: the document resulting from the mutation.
- `originalDocument`: the original document before the mutation being applied.

Note that for `validate` and `before` callbacks, `document` will be a *simulated* preview of the mutation result. 

##### `delete`

- `document`: the document being deleted.

## Global Callbacks

In addition to collection-specific callbacks, you can also define global callbacks that will run for every collections:

```
addGlobalCallbacks({
  create: {
    // same as above
  }
  update: {
    // same as above
  }
  delete: {
    // same as above
  }
});
```

Note that if you call `addGlobalCallbacks()` multiple times, the callback object will be merged together. This can be useful to add global callbacks from more than one place in your code. 

## Error Handling

By default, any error in a callback function will be swallowed to avoid affecting other callbacks or the mutation itself. You can pass `break` on your `Error` object to cause the chain of callbacks to stop when using a *sync* callback.

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

## Examples

### Error Callback

This example extends a collection on the server with a callback that checks for any post with duplicate links on create and update. 

```js
import { Connectors, extendCollection } from "meteor/vulcan:core";
import Posts from "../../modules/posts/collection.js";

const checkForDuplicates = async (validationErrors, { document, data }) => {
  const url = document.url || data.url;
  const duplicatePost = await Connectors.get(Posts, { url });
  if (duplicatePost) {
    validationErrors.push({
      id: 'app.duplicate_post',
      path: 'url',
    });
  }
  return validationErrors;
};

extendCollection(Posts, {
  callbacks: {
    create: {
      validate: [checkForDuplicates],
    },
    update: {
      validate: [checkForDuplicates],
    },
  },
});
```

Some things to note: 

- Callbacks only run on the server, so we are extending the collection on the server instead of including this code in the main collection declaration, which is shared with the client. 
- We look for the URL on both `document` (which exists within Create mutations) and `data` (for Update mutations).
- Instead of throwing an error, we add an error item to the `validationErrors` array.
- We set the path of that error to the path of our URL field so that the error can be displayed under the relevant form field when shown on the client. 
- Hooks can be called in an `async` manner. 