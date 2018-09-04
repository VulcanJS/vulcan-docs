---
title: Schema Properties
---

A range of special properties are available to control the behavior of each document field.

## SimpleSchema Properties

See the [SimpleSchema documentation](https://github.com/aldeed/meteor-simple-schema#schema-rules).

## Data Layer Properties

#### `canRead`

Can either be a string, an array of group names, or a function.

If it's a function, it'll be called on the `user` viewing the document and the `document` itself, and should return `true` or `false`.

#### `canCreate`

Can either be a string, an array of group names, or a function.

If it's a function, it'll be called on the `user` performing the operation, and should return `true` or `false`. When generating a form for inserting new documents, the form will contain all the fields that return `true` for the current user.

#### `canUpdate`

Can either be a string, an array of group names, or a function.

If it's a function, it'll be called on the `user` performing the operation, and the `document` being operated on, and should return `true` or `false`. When generating a form for editing existing documents, the form will contain all the fields that return `true` for the current user.

#### `resolveAs`

You can learn more about `resolveAs` in the [Field Resolvers](/field-resolvers.html) section.

### `onCreate`, `onUpdate`, `onDelete`

These three properties can take a callback function that will run during the corresponding operation, and should return the new value of the corresponding field.

* `onCreate({ newDocument, currentUser })`
* `onUpdate({ data, document, currentUser })`
* `onDelete({ document, currentUser })`

## Form Properties

These schema properties are mostly used for controlling the appearance and behavior of Vulcan's auto-generated forms.

#### `label`

The form label. If not provided, the label will be generated based on the field name and the available language strings data.

#### `input` 

(legacy: `control`)

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
  canCreate: Users.isAdmin,
  canUpdate: Users.isAdmin,
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
this.context.updateCurrentValues({ foo: 'bar' });
```

As long as a value is in `this.state.currentValues` it should be submitted with the form, no matter whether there is an actual form item or not.

#### `options`

An array containing a set of options for the form (for select, checkbox, radio, etc. controls), or a function that takes the component's `props` as argument and returns an array. 

```js
status: {
  type: Number,
  optional: true,
  canRead: ['admins'],
  canCreate: ['admins'],
  canUpdate: ['admins'],
  control: 'select',
  default: 2,
  options: () => {
    return [
      {
        value: 1,
        label: 'private',
      },
      {
        value: 2,
        label: 'public',
      },
    ];
  },
  description: `The room' status (Private/Public)`
},
```

#### `query`

A query used to require extra data needed to display the form field. See [field-specific data loading](/forms.html#Field-Specific-Data-Loading).

#### `inputProperties`

An object defining the props that will be passed to the `input` component to customize it. You can pass either values, or a function that will be evaluated each time the form is generated.

```
inputProperties: {
  locale: 'fr',
  defaultValue: () => new Date(), // will be evaluated each time a form is generated
},
input: 'datetime'
```

You may sometimes want to pass a function or a React component in the form. Since all functions in the `inputProperties` object will be evaluated before rendering the form, you'll need to pass a closure that returns the desired function or component instead.

```
inputProperties: {
  locale: 'fr',
  defaultValue: () => new Date(), // will be evaluated each time a form is generated
  renderOption: () => MyCustomComponent, // will return MyCustomComponent as expected
  sortValues: () => mySortingFunction, // will return mySortingFunction as expected
},
control: 'MyCustomSelect'
```

When the form is generated, the closure is evaluated and return your component or function. Thus in 
`MyCustomSelect`, `props.renderOption` will equal `MyCustomComponent` as expected.

#### `min`

You can set `min` to force the field to be longer than a certain length (e.g. `min: 20`).

#### `max`

You can set `max` to limit the field to a certain length (e.g. `max: 140`).

#### `defaultValue`

The field's default value. Note that you can also use `onCreate` to achieve the same effect.

#### `description`

A description that will be used as help text for the field. 

## Other Properties

#### `mustComplete` (`Users` only)

You can mark a field as `mustComplete: true` to indicate that it should be completed when the user signs up. If you're using the Forum example, a form will then pop up prompting the user to complete their profile with the missing fields.
