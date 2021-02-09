---
title: API Schemas
---

API schemas lets you define schema fields that only exist in your GraphQL API, meaning these "virtual" fields don't appear in your forms and don't get saved to your database.

### Defining API Schemas

These fields support the following properties: 

- `canRead`: same as main schema canRead, with one notable exception: for apiSchema fields, canRead will default to [guests] (e.g. the field is public).
- `description`: this is used as help text for the GraphQL API. 
- `typeName`: this is the GraphQL type (and not the JavaScript type, unlike "normal" fields!) the field should get resolved to.
- `arguments`: the field's arguments. 
- `resolver`: the field's resolver function.

Note that canCreate and canUpdate are not supported on apiSchema fields, since these fields by definition can not be stored in the database.

For example:

```js
const apiSchema = {
  latestPost: {
    canRead: ['members'],
    description,
    typeName: 'Post',
    resolve: () => { //... }
  }
}
```

### Adding API Schemas

You can add an API schema to a collection with:

```js
extendCollection(Posts, {
  apiSchema,
});
```