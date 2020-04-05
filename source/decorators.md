---
title: Field Decorators
---

(Available in Vulcan 1.15+)

Decorators are functions you can call on your schema fields to “augment” them and make them work with special inputs such as checkboxes, radio buttons, autocomplete inputs, and more. 

Since different UI frameworks such as Bootstrap, Material, etc. can have slightly different behaviors, decorators should be defined for each framework and imported from the framework's package (e.g. `meteor/vulcan:ui-bootstrap`).

## Autocomplete (Bootstrap)

You can make a schema field use an autocomplete input in your forms by calling the `makeAutocomplete` decorator.

It takes the field as first argument, and an `options` object as second argument that takes the following properties:

- `queryResolverName`: the name of the GraphQL resolver to query to load the list of autocomplete suggestions.
- `labelPropertyName`: the name of the property used to serve as a label on which to autocomplete.

For example, if you have a `Post` schema with a `categoriesIds` field, you would want to query the `categories` query resolver to load a list of categories as suggested items as the user types into the autocomplete field; and use a category's `name` as a label. 

The autocomplete input will default as a single-value autocomplete, but will act as a multi-value autocomplete for any `Array` field. Note that even for single-value autocompletes, for consistency you should pass a `queryResolverName` pointing to a *multi* resolver, even if only one item is only ever loaded.

### Example

```js
import { makeAutocomplete } from 'meteor/vulcan:ui-bootstrap';

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
  { queryResolverName: 'categories', labelPropertyName: 'name' })
}
```

## Checkboxgroup (Bootstrap)

The `makeCheckboxgroup` decorator will add a checkbox group input, and automatically add an `allowedValues` property to your schema based on the field's `options` property to restrict the field to allowed values. 

```js
import { makeCheckboxgroup } from 'meteor/vulcan:ui-bootstrap';

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

## Radiogroup (Bootstrap)

The `makeRadiogroup` decorator will add a radio buttin group input, and automatically add an `allowedValues` property to your schema based on the field's `options` property to restrict the field to allowed values. 

```js
import { makeRadiogroup } from 'meteor/vulcan:ui-bootstrap';

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

## Likert (Bootstrap)

The `makeLikert` decorator will add a Likert scale input, as well as automatically set the proper `Int` type for the field's array items. 

```js
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