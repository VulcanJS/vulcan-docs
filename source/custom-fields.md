---
title: Custom Fields
---

Out of the box, Nova has three main collections: `Posts`, `Users`, and `Comments`. Each of them has a pre-set schema, but that schema can also be extended with custom fields.

For example, this is how the `nova:newsletter` package extends the `Posts` schema with a `scheduledAt` property that keeps track of when a post was sent out as part of an email newsletter:

```js
Posts.addField({
  fieldName: 'scheduledAt',
  fieldSchema: {
    type: Date,
    optional: true
  }
});
```

The `collection.addField()` function takes either a field object, or an array of fields. Each field has a `fieldName` property, and a `fieldSchema` property.

Each field schema supports all of the [SimpleSchema properties](https://github.com/aldeed/meteor-simple-schema#schema-rules), such as `type`, `optional`, etc.

A few special properties (`insertableIf`, `editableIf`, `control`, and `order`) are also supported by the [Forms](forms.html) package.

You can also remove a field by calling `collection.removeField(fieldName)`. For example:

```js
Posts.removeField('scheduledAt');
```
