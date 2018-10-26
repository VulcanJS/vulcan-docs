---
title: Forms
---

The `vulcan:forms` package provides a `SmartForm` component that lets you easily generate new document and edit document forms. 

## Features

This package can generate new document and edit document forms from a [schema](schemas.html). Features include:

- Error handling.
- Bootstrap-compatible.
- Cross-component communication (prefill a field based on another).
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
    // no canCreate or canUpdate means this field won't appear in forms
  },
  title: {
    type: String,
    optional: false,
    max: 500,
    canCreate: isLoggedIn,
    canUpdate: isOwner,
    control: "text",
    order: 1
  },
  body: {
    type: String,
    optional: true,
    max: 3000,
    canCreate: isLoggedIn,
    canUpdate: isOwner,
    control: BodyFormControl,
    order: 2
  },
  sticky: {
    type: Boolean,
    optional: true,
    defaultValue: false,
    canCreate: isAdmin,
    canUpdate: isAdmin,
    control: "checkbox",
    order: 3
  },
}
```

## Creating Forms

### New Document

Just pass the `collection` props top the `SmartForm` component:

```jsx
<Components.SmartForm
  collection={Posts}
/>
```

### Edit Document

Same as the New Document form, but also passing the `documentId` to edit. 

```jsx
<Components.SmartForm
  collection={Posts}
  documentId={post._id}
/>
```

## Props

Here are all the props accepted by the `SmartForm` component:

### Basic Props

#### `collection`

The collection in which to edit or insert a document.

#### `documentId`

If present, the document to edit. If not present, the form will be a “new document” form. 

#### `fields`

An array of field names, if you want to restrict the form to a specific set of fields.

#### `layout`

A layout property used to control how the form fields are displayed. Defaults to `horizontal`. 

#### `showRemove`

Whether to show a "delete document" link on edit forms. 

#### `prefilledProps`

A set of props used to prefill the form. 

#### `repeatErrors`

Whether to repeat validation errors at the bottom of the form. 

### Callbacks

#### `submitCallback(data)`

A callback called on form submission on the form data. Should return the `data` object as well.

#### `successCallback(document)`

A callback called on mutation success.

#### `errorCallback(document, error)`

A callback called on mutation failure.

#### `cancelCallback(document)`

If a `cancelCallback` function is provided, a "cancel" link will be shown next to the form's submit button and the callback will be called on click. 

#### `removeSuccessCallback(document)`

A callback to call when a document is successfully removed (deleted).

### Fragments

#### `queryFragment`

A GraphQL fragment used to specify the data to fetch to populate `edit` forms. 

If no fragment is passed, SmartForm will do its best to figure out what data to load based on the fields included in the form.

#### `mutationFragment`

A GraphQL fragment used to specify the data to return once a mutation is complete. 

If no fragment is passed, SmartForm will only return fields used in the form, but note that this might sometimes lead to discrepancies when compared with documents already loaded on the client. 

An example would be a `createdAt` date added automatically on creation even though it's not part of the actual form. If you'd like that field to be returned after the mutation, you can define a custom `mutationFragment` that includes it explicitly.   

## Field-Specific Data Loading

Sometimes, a specific field will need specific data in addition to its own value. For example, you might have a `category` field on a `Post` that stores a category's `_id`, but simply showing users an empty text-field where they can manually type in that `_id` isn't very user-friendly. 

Instead, you'll probably want to populate a dropdown with all your existing categories' names (and maybe also images, descriptions, etc.) to make it easier to pick the right one. This in turns means you need a way to *load* all these categories in the first place. 

This is where field-level data loading comes in. This gives you an easy way to tell Vulcan Forms that you need an extra bit of data whenever that field is displayed:

```js
categoryId: {
  type: String,
  control: 'checkboxgroup',
  optional: true,
  canCreate: ['members'],
  canUpdate: ['members'],
  canRead: ['guests'],
  query: `
    CategoriesList{
      _id
      name
      slug
      order
    }
  `,
  options: props => props.data.CategoriesList.map(category => ({
      value: category._id,
      label: category.name,
  })),
  resolveAs: ...
}
```

We're doing two things here. First, we're setting the `query` property and passing it an additional bit of GraphQL query code that will be executed when the form is loaded. 

Because the extra query code calls the `CategoriesList` resolver, whatever the resolver returns will then be available on `props.data` once our data is done loading. 

This lets us set the `options` property in order to populate our dropdown. Essentially, we're just translating a list of categories into a list of `{ value, label }` pairs.

### Using `documentId`

Field-specific queries work by adding “extra” query parts to a specially created formNewExtraQuery HoC when inserting new documents; or to the withSingle HoC when editing an existing document. 

When editing a document, you can reuse the `documentId` in your extra query parts since it will already have been made available to the main query. 

For example, you might want to restrict a list of users to those having the ability to moderate a given document:

```js
moderatorId: {
  type: String,
  control: 'select',
  optional: true,
  canCreate: ['members'],
  canUpdate: ['members'],
  canRead: ['guests'],
  query: `
    listDocumentModerators(documentId: $documentId){
      _id
      displayName
    }
  `,
  options: props => props.data.listDocumentModerators.map(user => ({
      value: user._id,
      label: user.displayName,
  })),
  resolveAs: ...
}
```

Of course, you'll also have to write your own `listDocumentModerators` [custom resolver](/resolvers.html#Custom-Resolvers) that takes in a `documentId` argument and returns the corresponding list of users. 

Note that although `currentUser` is not passed as an argument to your resolvers, it's available on the `context` object on the server.

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

This package uses [React Intl](https://github.com/yahoo/react-intl/) to automatically translate all labels. In order to do so it expects an `intl` object to be passed as part of its context. For example, in a parent component: 

```
getChildContext() {
  const intlProvider = new IntlProvider({locale: myLocale}, myMessages);
  const {intl} = intlProvider.getChildContext();
  return {
    intl: intl
  };
}
```

#### Alternative Approach

If you prefer, you can also code your own forms from scratch, either using `withCreate`, `withUpdate`, and `withDelete`, or with your own custom mutation HoCs. 
