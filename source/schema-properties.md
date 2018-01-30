---
title: Schema Properties
---

A range of special properties are available to control the behavior of each document field. 

## SimpleSchema Properties

See the [SimpleSchema documentation](https://github.com/aldeed/meteor-simple-schema#schema-rules).

## Data Layer Properties

#### `viewableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` viewing the document and the `document` itself, and should return `true` or `false`.

#### `insertableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` performing the operation, and should return `true` or `false`. When generating a form for inserting new documents, the form will contain all the fields that return `true` for the current user.

#### `editableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` performing the operation, and the `document` being operated on, and should return `true` or `false`. When generating a form for editing existing documents, the form will contain all the fields that return `true` for the current user.

#### `resolveAs`

You can learn more about `resolveAs` in the [Field Resolvers](/data-loading.html#Field-Resolvers) section. 

### `onInsert`, `onEdit`, `onRemove`

These three properties can take a callback function that will run during the corresponding operation, and should return the new value of the corresponding field. 

- `onInsert(document, currentUser)`
- `onEdit(modifier, document, currentUser)`
- `onRemove(document, currentUser)`

## Forms Properties

#### `label`

The form label. If not provided, the label will be generated based on the field name and the available language strings data.

#### `control`

Either a text string (one of `text`, `textarea`, `checkbox`, `checkboxgroup`, `radiogroup`, or `select`) or a React component.

#### `order`

A number corresponding to the position of the property's field inside the form.

#### `group`

An optional object containing the group/section/fieldset in which to include the form element. Groups have `name`, `label`, and `order` properties.

For example:

```js
postedAt: {
  type: Date,
  optional: true,
  insertableBy: Users.isAdmin,
  editableBy: Users.isAdmin,
  publish: true,
  control: "datetime",
  group: {
    name: "admin",
    label: "Admin Options",
    order: 2
  }
},
```

Note that fields with no groups are always rendered first in the form.

#### `placeholder`

A placeholder value for the form field.

#### `beforeComponent`

A React component that will be inserted just before the form component itself.

#### `afterComponent`

A React component that will be inserted just after the form component itself.

#### `hidden`

Can either be a boolean or a function accepting form props as argument and returning a boolean.

Remove the field from the form altogether. To populate the field manipulate the value through the context, e.g.:

```js
this.context.updateCurrentValues({foo: 'bar'});
```

As long as a value is in this.state.currentValues it should be submitted with the form, no matter whether there is an actual form item or not.

#### `form`

An object defining the props that will be passed to the `control` component to customize it. You can pass either values, or functions that will be evaluated each time the form is generated.

```
form:{
  locale: 'fr',
  defaultValue: () => new Date(), // will be evaluated each time a form is generated
},
control: 'datetime'
```

You may sometimes want to pass a function or a React component in the form. Since all functions in the `form` object will be evaluated before rendering the form, you'll need to pass a closure that returns the desired function or component instead.
```
form:{
  locale: 'fr',
  defaultValue: () => new Date(), // will be evaluated each time a form is generated
  renderOption: () => MyCustomComponent, // will return MyCustomComponent as expected
  sortValues: () => mySortingFunction, // will return mySortingFunction as expected
},
control: 'MyCustomSelect'
```
When the form is generated, the closure is evaluated and return your component or function. Thus in `MyCustomSelect`, `props.renderOption` will equal `MyCustomComponent` as expected.


## Other Properties

#### `mustComplete` (`Users` only)

You can mark a field as `mustComplete: true` to indicate that it should be completed when the user signs up. If you're using the Forum example, a form will then pop up prompting the user to complete their profile with the missing fields. 
