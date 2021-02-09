---
title: Field Decorators
---

(Available in Vulcan 1.15+)

Decorators are functions you can call on your schema fields to “augment” them and make them work with special inputs such as checkboxes, radio buttons, autocomplete inputs, and more. 

## Autocomplete

You can make a schema field use an autocomplete input in your forms by calling the `makeAutocomplete` decorator.

It takes the field as first argument, and an `options` object as second argument that takes the following properties:

- `autocompletePropertyName`: the name of the property used to serve as a label on which to autocomplete.
- `queryResolverName` (optional): the name of the GraphQL resolver to query to load the list of autocomplete suggestions.

For example, if you have a `Post` schema with a `categoriesIds` field, you would want to query the `categories` query resolver to load a list of categories as suggested items as the user types into the autocomplete field; and use a category's `name` as a label. 

Note that for relation fields (where `resolveAs.relation` exists), Vulcan will try to guess the appropriate resolver name if you don't specify it explicitly. 

The autocomplete input will default as a single-value autocomplete, but will act as a multi-value autocomplete for any `Array` field. Note that even for single-value autocompletes, for consistency you should pass a `queryResolverName` pointing to a *multi* resolver, even if only one item is only ever loaded.

### Example

```js
import { makeAutocomplete } from 'meteor/vulcan:core';

const postSchema = {
  categoriesIds: makeAutocomplete({
    type: Array,
    arrayItem: {
      type: String,
      optional: true
    },
    label: 'Categories',
    optional: true,
    canCreate: ['members'],
    canUpdate: ['owners'],
    canRead: ['guests'],
    resolveAs: {
      fieldName: 'categories',
      type: '[Category]',
      relation: 'hasMany'
    }
  },
  { queryResolverName: 'categories', autocompletePropertyName: 'name' })
}
```

## Checkboxgroup

The `makeCheckboxgroup` decorator will add a checkbox group input, and automatically add an `allowedValues` property to your schema based on the field's `options` property to restrict the field to allowed values. 

```js
import { makeCheckboxgroup } from 'meteor/vulcan:core';

const schema = {
  checkboxGroupField: makeCheckboxgroup({
    type: Array,
    arrayItem: { 
      type: String 
    },
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['owners'],
    options: fieldOptions
  })
}
```

## Radiogroup

The `makeRadiogroup` decorator will add a radio buttin group input, and automatically add an `allowedValues` property to your schema based on the field's `options` property to restrict the field to allowed values. 

```js
import { makeRadiogroup } from 'meteor/vulcan:core';

const schema = {
  radioGroupField: makeRadiogroup({
    type: Array,
    arrayItem: { 
      type: String 
    },
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['owners'],
    options: fieldOptions
  })
}
```

## Likert

The `makeLikert` decorator will add a Likert scale input, as well as automatically set the proper `Int` type for the field's array items. 

```js
import { makeLikert } from 'meteor/vulcan:core';

const schema = {
  likertField: makeLikert({
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['owners'],
    options: [
      { value: 'reservation', label: 'Reservation process' },
      { value: 'ease_of_use', label: 'Ease of use' },
      { value: 'design', label: 'Visual design' },
      { value: 'support', label: 'Customer support' },
      { value: 'properties', label: 'Number of properties' },
      { value: 'accomodations', label: 'Quality of accomodations' },
      { value: 'pricing', label: 'Price of accomodations' }
    ],
  })
}
```