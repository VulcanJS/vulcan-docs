---
title: Reactive State
---

## Introduction

Vulcan's Reactive State feature offers simple global state management on the client side based on [Apollo Client reactive variables](https://www.apollographql.com/docs/react/local-state/reactive-variables/). You can think of Reactive State as a wrapper around Apollo reactive variables with some added features:

* Schema validation of objects stored in Reactive State using our familiar SimpleSchema pattern
* Object state mutation similar to React `setState()` (with Apollo reactive vars updating an object value replaces the whole object)
* The ability to reset all reactive states globally - which happens automatically when the user logs out

Use it to store session data that survives re-renders and router transitions, unlike component state. You can create as many reactive states as you need, each identified with a unique key, and each holding either a simple scalar value like a boolean or string, or an object with an optional SimpleSchema for cleaning and validation.

## Reactive state with a simple scalar value

You can create a new reactive state using `createReactiveState()`:

```js
import { createReactiveState } from 'meteor/vulcan:core';

export const exampleOpenState = 
    createReactiveState({ stateKey: 'exampleOpenState', defaultValue: false });
```

At any time you can get the state’s current value using `reactiveState()`, or in this example:

```js
let isOpen = exampleOpenState();  // isOpen is set to false
```

You can set the state to a new value using `reactiveState(newValue)`:

```js
exampleOpenState(true);
```

You can use a global state to make components reactive using [Apollo Client's useReactiveVar hook](https://www.apollographql.com/docs/react/local-state/managing-state-with-field-policies/#storing-local-state-in-reactive-variables):

```jsx
import React from 'react';
import { exampleOpenState } from './exampleOpenState.js'
import { useReactiveVar } from '@apollo/client';
[ . . . ]

const ExampelModal = ({title, children}) => {

  const handleOpen = () => {
    exampleOpenState(true);
  };
  
  const handleClose = () => {
    exampleOpenState(false);
  };
  
  return (
    <div>
      <Dialog onClose={handleClose} open={useReactiveVar(exampleOpenState.reactiveVar)}>
          
        <DialogTitle>{title}</DialogTitle>
        
        <DialogContent>{children}</DialogContent>
        
        <DialogActions>
          <Button onClick={handleClose}>Close</Button>
        </DialogActions>
      
      </Dialog>
      
      <Button onClick={handleOpen}>Open</Button>
    </div>
  );
};
```

Now you may be thinking, "why can't I just use component state or `useState`?" Well, you can, but you need to consider two things:

 1. If the component is ever unmounted and mounted again, it will lose its state.
 2. If you update the state of a component, all of its child components will be remounted and will lose their state.

Using the above example, if you open the modal, navigate to another route and then come back, the modal will no longer be open.

Where you store your state has a lot to do with how persistent you need it to be:

 * If you want that modal to start closed every time its mounted, store it in component state
 * If you want the modal's open state to persist for the whole browser session, store it in reactive state
 * If you want the state to persist even when the user comes back days later, store it in a field of the User document


## Reactive state with an object value and optional schema

In addition to simple scalars, you can also store objects in reactive state. If you also specify a schema, an error will be thrown if you try to set the state to a value that does not respect the schema.

```js
import { createReactiveState } from 'meteor/vulcan:core';

const topBarSchema = {
  messageId: {
    type: String,
    defaultValue: 'global.welcome',
  },
  showClose: {
    type: Boolean,
    defaultValue: true,
  },
  isOpen: {
    type: Boolean,
    defaultValue: false,
  },
  classes: {
    type: Array,
    optional: true,
    arrayItem: {
      type: String,
    },
  },
};

export const topBarState = createReactiveState({ 
  stateKey: 'topBarState', 
  schema: topBarSchema,
});
```

We recommend that you export the returned state and import it where you need to use it. Alternatively, you can retrieve a registered state using `getReactiveState('topBarState')`.

Let's have a closer look at the returned state. In addition to using it to get or set the state's value, it has the following properties:

 * `stateKey` - The name/id/key of the state
 * `schema` - A `SimpleSchema` instance (or `undefined` if a schema was not passed when creating the state)
 * `defaultValue` - The default value of the state, passed as a parameter when creating the state, or derived from the schema (or `undefined` if neither `defaultValue` nor `schema` was passed when creating the state)
 * `reactiveVar` - An Apollo reactive variable

Using the above example, the default value of a newly created `topBarState` would be:

```json
{
  "messageId": "global.welcome",
  "showClose": true,
  "isOpen": false
}
```

## Mutating an object state

Earlier we saw `exampleOpenState(true)`, which updates the state's value. For an object state this works in a way that's similar to React's `setState()`. If you pass an object parameter, it performs a shallow merge on the current value, updating only the properties included in the parameter.

```js
topBarState({ isOpen: true });
```

If you pass a function, you can mutate the state as you wish:

```js
topBarState(state => { 
  state.classes.push('top-bar', 'welcome-bar');
  return state;
});
```

## Resetting reactive state

You can reset a state to its default value by invoking it with `null`.

```js
topBarState(null);
```

You can reset all states that have been registered using `resetReactiveState()`. This happens automatically when a user signs out or when the Apollo client store is reset.
