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

#### `prefilledProps`

A set of props to prefill for new documents. 

#### `fields`

An array of field names, if you want to restrict the form to a specific set of fields.

#### `queryFragment`

A GraphQL fragment used to specify the data to fetch to populate `edit` forms. 

If no fragment is passed, SmartForms will do its best to figure out what data to load based on the fields included in the form.

#### `mutationFragment`

A GraphQL fragment used to specify the data to return once a mutation is complete. 

If no fragment is passed, SmartForms will only return fields used in the form, but note that this might sometimes lead to discrepancies when compared with documents already loaded on the client (an example would be a `createdAt` date added automatically on creation even though it's not part of the actual form).  

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

## Form Components

- [Upload](forms-upload.html)

## Custom Components

You'll often need to write your own form components. Fortunately, VulcanJS makes it relatively easy. The main thing you need to figure out is how your custom component will pass its value to the form's data in order to submit it back to the server. 

There are two main ways to do so.

### Using an Input

The easiest way is simply to store this value in an input form element. While this won't work with a regular `<input/>`, it will work with any [Formsy](https://github.com/christianalfoni/formsy-react) element.

In other words, you can include any of the [Formsy React Components](https://github.com/twisty/formsy-react-components) inside your own custom component, and that component's value will automatically be picked up by the form when it's submitted. 

For example, a useful pattern might be including a hidden Formsy `<Input/>`, and updating its value via your custom component's `state`:

```js
import React, { PureComponent } from 'react';
import PropTypes from 'prop-types';
import { Input } from 'formsy-react-component';

class MyCustomFormComponent extends PureComponent {
  
  constructor() {
    super();
    this.toggleMessage = this.toggleMessage.bind(this);
    this.state = {
      message = 'foo'
    };
  }

  toggleMessage() {
    this.setState({
      message: this.state.message === 'foo' ? 'bar' : 'foo'
    });
  }

  render() {

    return (
      <div className="form-group row">
        <label className="control-label col-sm-3">{this.props.label}</label>
        <div className="col-sm-9">
          <p>Message is: {this.state.message}</p>
          <a onClick={this.toggleMessage}>Toggle Message</a>
          <Input type="hidden" name="message" value={this.state.message}/>
        </div>
      </div>
    );
  }
}

export default MyCustomFormComponent;
```

### Using a Callback

Another method to achieve the same thing is to call `this.context.updateCurrentValues()` directly. This function takes an object and will update the parent `Form` component's state with its properties:

```js
import React, { PureComponent } from 'react';
import PropTypes from 'prop-types';
import { Input } from 'formsy-react-component';

class MyCustomFormComponent extends PureComponent {
  
  constructor() {
    super();
    this.toggleMessage = this.toggleMessage.bind(this);
    this.state = {
      message = 'foo'
    };
  }

  toggleMessage() {
    this.setState({
      message: this.state.message === 'foo' ? 'bar' : 'foo'
    });
    this.context.updateCurrentValues({message: this.state.message});
  }

  render() {

    return (
      <div className="form-group row">
        <label className="control-label col-sm-3">{this.props.label}</label>
        <div className="col-sm-9">
          <p>Message is: {this.state.message}</p>
          <a onClick={this.toggleMessage}>Toggle Message</a>
        </div>
      </div>
    );
  }
}

MyCustomFormComponent.contextTypes = {
  updateCurrentValues: PropTypes.func,
};

export default MyCustomFormComponent;
```

Note that you'll need to define your custom component's `contextTypes` property to make `updateCurrentValues` available on the component's `context`.

### SmartForms API

Here's an overview of the API methods available on a custom component's `context`:

#### `updateCurrentValues(object)`

This takes an object with `name: value` pairs, and will update the form state accordingly. 

#### `addToAutofilledValues(object)`

This adds a set of `name: value` pairs to the form's *autofilled* values. Autofilled values have a lower priority than "current" values. In other words, if someone fills in the `foo` field with `bar` but you then call `addToAutofilledValues({foo: 'baz'})`, the contents of `foo` will not change. 

This is useful when you want the value of one field to affect the contents or another, except when that other field has already been filled out. 

#### `getAutofilledValues()`

Get just the autofilled values.

#### `addToDeletedValues(string)`

This takes a single **field name** and adds it to the list of document properties to be deleted on the server once the form is submitted. 

Note that once a field is added to the form state's `deletedValues`, it will be deleted even if said field contains a value. 

#### `addToSubmitForm(function)`

Adds a callback that will be called on the `data` object containing all form values when the form is submitted.

#### `addToFailureForm(function)`

Adds a callback that will run with the `error` as argument if the form submission fails. 

#### `addToSuccessForm(function)`

Adds a callback that will run with the `result` as argument if the form submission succeeds.

#### `throwError(object)`

Throws an error.

#### `clearForm()`

Clears the form.

#### `getDocument()`

Gets the entire document currently being inserted or edited. 

#### `setFormState(object)`

Calls `setState` inside the `Form` component directly. Can be used as a more general method to affect form state whenever the previous API methods are not sufficient. For example, to set the form as disabled:

```
this.context.setFormState({disabled: true});
```
