---
title: Cheatsheet
---

## Mutations

### Mutators

```js
await createMutator({
  collection,
  data,
  currentUser,
  validate,
  context,
});

await updateMutator({
  documentId
  collection,
  data,
  currentUser,
  validate,
  context,
});

await deleteMutator({
  documentId,
  collection,
  currentUser,
  validate,
  context,
});
```

### Callbacks

#### Properties

```js
// create
const properties = {
  data,
  originalData: clone(data),
  currentUser,
  collection,
  context,
  schema,
};

// update
const properties = {
  data,
  originalData: clone(data),
  document,
  originalDocument: oldDocument,
  currentUser,
  collection,
  context,
  schema,
};

// delete
const properties = { 
  document, 
  currentUser, 
  collection, 
  context, 
  schema 
};
```

#### Mutation Callbacks

```js
createCollection({
  callbacks {
    create: {
      validate: [(validationErrors, properties) => { return validationErrors; }],
      before: [(document, properties) => { return document; }],
      after: [(document, properties) => { return document; }],
      async: [(properties) => { /* no return value */ }],
    }
    update: {
      validate: [(validationErrors, properties) => { return validationErrors; }],
      before: [(data, properties) => { return data; }],
      after: [(document, properties) => { return document; }],
      async: [(properties) => { /* no return value */ }],
    }
    delete: {
      validate: [(validationErrors, properties) => { return validationErrors; }],
      before: [(document, properties) => { return document; }],
      after: [(document, properties) => { return document; }],
      async: [(properties) => { /* no return value */ }],
    }
  }
});
```

#### Field Callbacks

```js
const schema = {
  field: {
    onCreate: (properties) => { return fieldValue },
    onUpdate: (properties) => { return fieldValue },
    onDelete: (properties) => { // return nothing }
  }
}

