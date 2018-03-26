---
title: Custom Form Components
---

## Custom Components

You'll often need to write your own form components. Fortunately, VulcanJS makes it relatively easy. The main thing you need to figure out is how your custom component will pass its value to the form's data in order to submit it back to the server. 

### Handling Values

Vulcan form components are **controlled components**, meaning that they get their value from their parent component. In other words, you can't change their value directly but must instead call the `onChange` event handler, which will itself call `this.context.updateCurrentValues()`.

You can also call `this.context.updateCurrentValues()` directly. This function takes an object and will update the parent `Form` component's state with its properties:

```js
import React, { PureComponent } from 'react';
import PropTypes from 'prop-types';
import { Input } from 'formsy-react-components';

class MyCustomFormComponent extends PureComponent {
  
  constructor() {
    super();
    this.toggleMessage = this.toggleMessage.bind(this);
    this.state = {
      message: 'foo'
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

This takes an object with `name: value` pairs, and will update the form state accordingly. Note that `name` can also be a dotted path, for nested components (e.g. `{ 'addresses.2.zipCode': 12345 }`).

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
