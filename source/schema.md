---
title: Schema
---

## Default Properties

See the [SimpleSchema documentation](https://github.com/aldeed/meteor-simple-schema#schema-rules).

## Data Layer Properties

#### `viewableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` viewing the document, and should return `true` or `false`. When generating a form for inserting new documents, the form will contain all the fields that return `true` for the current user. 

#### `insertableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` performing the operation, and should return `true` or `false`. When generating a form for inserting new documents, the form will contain all the fields that return `true` for the current user. 

#### `editableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` performing the operation, and the `document` being operated on, and should return `true` or `false`. When generating a form for editing existing documents, the form will contain all the fields that return `true` for the current user. 

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

