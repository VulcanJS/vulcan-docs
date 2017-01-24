---
title: Forms
---

The `nova:forms` package provides a `SmartForm` component that lets you easily generate new document and edit document forms. 

## Features

This package can generate new document and edit document forms from a [SimpleSchema](https://github.com/aldeed/meteor-simple-schema) schema. Features include:

- Error handling.
- Bootstrap-compatible.
- Cross-component communciation (prefill a field based on another).
- Callbacks on form submission, success, and failure.
- Support for basic form controls (input, textarea, radio, etc.).
- Support for custom form controls.
- Submission to Meteor methods. 

## Usage

Example schema:

```js

import BodyFormControl from './components/BodyFormControl.jsx';

const isLoggedIn = (user) => !!user;
const isOwner = (user, document) => user._id === document.userId;
const isAdmin = (user) => user.isAdmin;

const PostsSchema = new SimpleSchema({
  postedAt: {
    type: Date,
    optional: true
    // no insertableIf or editableIf means this field won't appear in forms
  },
  title: {
    type: String,
    optional: false,
    max: 500,
    insertableIf: isLoggedIn,
    editableIf: isOwner,
    control: "text",
    order: 1
  },
  body: {
    type: String,
    optional: true,
    max: 3000,
    insertableIf: isLoggedIn,
    editableIf: isOwner,
    control: BodyFormControl,
    order: 2
  },
  sticky: {
    type: Boolean,
    optional: true,
    defaultValue: false,
    insertableIf: isAdmin,
    editableIf: isAdmin,
    control: "checkbox",
    order: 3
  },
}
```

New document form:

```jsx
<SmartForm 
  collection={Posts}
/>
```

Edit document form:

```jsx
<SmartForm 
  collection={Posts}
  document={post}
/>
```

## Props

#### `collection`

The collection in which to edit or insert a document.

#### `fragment`

A fragment that defines what data you expect back from the mutation. 

If no fragment is passed, SmartForms will only return fields used in the form, but that might lead to discrepancies when compared with documents already loaded on the client (an example would be a `createdAt` date added automatically on creation even though it's not part of the actual form).  

#### `schema`

If you prefer, you can also specify a schema instead of a collection.

#### `documentId`

If present, the document to edit. If not present, the form will be a “new document” form. 

#### `submitCallback(data)`

A callback called on form submission on the form data. Should return the `data` object as well.

#### `successCallback(document)`

A callback called on method success.

#### `errorCallback(document, error)`

A callback called on method failure.

#### `cancelCallback()`

If provided, will show a "cancel" link next to the form's submit button. 

#### `labelFunction`

A function to call on field names to get the label.

#### `prefilledProps`

A set of props to prefill for new documents. 

## Context

The main `SmartForm` components makes the following objects available as context to all its children:

#### `autofilledValues`

An object containing optional autofilled properties. 

#### `addToAutofilledValues({name: value})`

A function that takes a property, and adds it to the `autofilledValues` object. 

#### `throwError({content, type})`

A callback function that can be used to throw an error. 

#### `getDocument()`

A function that lets you retrieve the current document from a form component.

## Handling Values

The component handles three different layers of input values:

- The value stored in the database (when editing a document).
- The value being currently inputted in the form element.
- An “autofilled” value, typically provided by an *other* form element (i.e. autofilling the post title from its URL).

The highest-priority value is the user input. If there is no user input, we default to the database value provided by the `props`. And if that one is empty too, we then look for autofilled values. 

## i18n

This package uses [React Intl](https://github.com/yahoo/react-intl/) to automatically translate all labels. In order to do so it expects an `intl` object ot be passed as part of its context. For example, in a parent component: 

```
getChildContext() {
  const intlProvider = new IntlProvider({locale: myLocale}, myMessages);
  const {intl} = intlProvider.getChildContext();
  return {
    intl: intl
  };
}
```
