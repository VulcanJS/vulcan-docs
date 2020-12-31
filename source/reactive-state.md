---
title: Reactive State
---

## Introduction

Vulcan's Reactive State feature offers simple global state management on the client side based on [Apollo Client reactive variables](https://www.apollographql.com/docs/react/local-state/reactive-variables/) exposed as React HoCs and Hooks.

Use it to store session data that survives re-renders and router transitions, unlike component state. You can create as many reactive states as you need, each identified with a unique key, and each holding either a simple scalar value like a boolean or string or an objects with an optional SimpleSchemas for cleaning and validation.

## Reactive state with a simple scalar value

You can create a new reactive state using `createReactiveState()`:

```js
import { createReactiveState } from 'meteor/vulcan:core';

createReactiveState({ stateKey: 'ExampleModalIsOpen', defaultValue: false });
```
At any time you can get the state’s current value using `getReactiveStateValue('ExampleModalIsOpen')` which would in this case return `false`.

You can set the state to a new value using `setReactiveStateValue('ExampleModalIsOpen', true)`.

You can use a global state (like the one we created above) to make components reactive using [Apollo Client's useReactiveState hook](https://www.apollographql.com/docs/react/local-state/managing-state-with-field-policies/#storing-local-state-in-reactive-variables):
```jsx
const ExampelModal = ({title, children}) => {
  const { ExampleModalIsOpen, setExampleModalIsOpen } = 
      useReactiveState({ stateKey: 'ExampleModalIsOpen' });
  
  const handleOpen = () => {
    setExampleModalIsOpen(true);
  };
  
  const handleClose = () => {
    setExampleModalIsOpen(false);
  };
  
  return (
    <div>
      <Dialog onClose={handleClose} open={ExampleModalIsOpen}>
          
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
Now you may be thinking, _why can't I just use component state or `useState`?_ Well, you can, but if the component is ever unmounted and remounted, it will have lost its state. So, using the above example, if you open the modal, navigate to another route and then come back, the modal will no longer be open.

Where you store your state has a lot to do with how persistent you need it to be:

 * If you want that modal to start closed every time its mounted, store it in component state
 * If you want the modal's open state to persist for the whole browser session, store it in reactive state
 * If you want the state to persist even when the user comes back days later, store it in a field of the User document


## Reactive state with an object and optional schema

In addition to simple scalars, you can also store objects in reactive state:

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

const topBarState = createReactiveState({ stateKey: 'TopBarState', schema: topBarSchema });
```
You probably don't need to assign the returned state object to a variable, but let's have a closer look at it. It has the following properties:

 * `stateKey` - The name/id/key of the state
 * `schema` - A `SimpleSchema` instance (or `undefined` if a schema was not passed when creating the state)
 * `defaultValue` - The default value of the state, passed as a parameter when creating the state, or derived from the schema (or `undefined` if neither `defaultValue` nor `schema` was passed when creating the state)
 * `reactiveVar` - An Apollo reactive variable

At any time you can get this state object using `getReactiveState('TopBarState')` or you can get the state’s current value using `getReactiveStateValue('TopBarState')` which would return:

```json
{
  "messageId": "global.welcome",
  "showClose": true,
  "isOpen": false
}
```

## Mutating the state

Earlier we had a look at `setReactiveStateValue()`, which is useful for setting a simple scalar state to a new value, or setting an object state to an entirely new object.

There is another function if you want to mutate an object state, `updateReactiveStateValue()`, which uses [immutability-helper](https://github.com/kolodny/immutability-helper). Here are some examples:

```js
updateReactiveStateValue('TopBarState', { isOpen: { $set: true } });
updateReactiveStateValue('TopBarState', { classes: { $set: [] } });
updateReactiveStateValue('TopBarState', { classes: { $push: ['top-bar', 'welcome-bar'] } });
```
