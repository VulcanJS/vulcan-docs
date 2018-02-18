---
title: Forms
---

The `vulcan:forms` package provides a `SmartForm` component that lets you easily generate new document and edit document forms. 

## Features

This package can generate new document and edit document forms from a [schema](schemas.html). Features include:

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
    // no insertableBy or editableBy means this field won't appear in forms
  },
  title: {
    type: String,
    optional: false,
    max: 500,
    insertableBy: isLoggedIn,
    editableBy: isOwner,
    control: "text",
    order: 1
  },
  body: {
    type: String,
    optional: true,
    max: 3000,
    insertableBy: isLoggedIn,
    editableBy: isOwner,
    control: BodyFormControl,
    order: 2
  },
  sticky: {
    type: Boolean,
    optional: true,
    defaultValue: false,
    insertableBy: isAdmin,
    editableBy: isAdmin,
    control: "checkbox",
    order: 3
  },
}
```

## Creating Forms

### New Document

Just pass the `collection` props top the `SmartForm` component:

```jsx
<SmartForm 
  collection={Posts}
/>
```

### Edit Document

Same as the New Document form, but also passing the `documentId` to edit. 

```jsx
<SmartForm 
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

An example would be a `createdAt` date added automatically on creation even though it's not part of the actual form. If you'd like that field to be returned after the mutation, you can define a custom `mutationFragment` that includes it explicitely.   

## Field-Specific Data Loading

Sometimes, a specific field will need specific data in addition to its own value. For example, you might have a `category` field on a `Post` that stores a category's `_id`, but simply showing users an empty text-field where they can manually type in that `_id` isn't very user-friendly. 

Instead, you'll probably want to populate a dropdown with all your existing categories' names (and maybe also images, descriptions, etc.) to make it easier to pick the right one. This in turns means you need a way to *load* all these categories in the first place. 

This is where field-level data loading comes in. This gives you an easy way to tell Vulcan Forms that you need an extra bit of data whenever that field is displayed:

```js
categoryId: {
  type: String,
  control: 'checkboxgroup',
  optional: true,
  insertableBy: ['members'],
  editableBy: ['members'],
  viewableBy: ['guests'],
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

### Using Variables (`devel` branch)

Field-specific queries work by adding “extra” query parts to a specialy created `formNewExtraQuery` HoC when inserting new documents; or to the `withDocument` HoC when editing an existing document. In either case, the main query will be called with the following `extraTerms` argument:

```js
const extraTerms = {
  documentId,
  currentUser,
  view: `${collectionName}ExtraQueryView` // where collectionName is the collection the form belongs to
}
```

You can use this `extraTerms` object for more complex data loading strategies. For example, let's imagine you want to load the list of all `Photos` taken by a user that are associated with the current `Articles` document. 

Since our `extraTerms` object is hard-coded to expect an `ArticlesExtraQueryView` [view](/terms-parameters.html#Using-Views), let's create it. It will take in the `extraTerms` object and return the appropriate database selector:

```js
Photos.addView('ArticlesExtraQueryView', extraTerms => ({
  selector: {
    userId: extraTerms.currentUser && extraTerms.currentUser._id,
    articleId: extraTerms.documentId,
  }
}));
```

Note that we add the view to the `Photos` collection, because that's what we want it to return, but the view is named `ArticlesExtraQueryView` because the form is for editing a document from the `Articles` collection. 

With the view defined, we can pass `extraTerms` to our default `PhotosList` resolver as the `terms` argument:

```js
featuredPhotoId: {
  type: String,
  control: 'select',
  optional: true,
  insertableBy: ['members'],
  editableBy: ['members'],
  viewableBy: ['guests'],
  query: `
    PhotosList(terms: $extraTerms){
      _id
      url
      height
      width
      credit
    }
  `,
  options: props => props.data.PhotosList.map(photo => ({
      value: photo._id,
      label: photo.name,
  })),
  resolveAs: ...
}
```

Note that because all extra queries share the same `extraTerms` object, it's currently not possible to load two different datasets for the same collection unless you create your own custom resolvers. 

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

#### Alternative Approach

If you prefer, you can also code your own forms from scratch, either using `withNew`, `withEdit`, and `withRemove`, or with your own custom mutation HoCs. 
