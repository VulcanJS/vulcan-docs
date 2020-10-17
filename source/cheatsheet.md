---
title: Cheatsheet
---


## Queries

### Input

```js
const singleInput = {
  filter,
  sort,
  search,
  id,
  enableCache,
  enableTotal,
  contextName,
}

const multiInput = {
  filter,
  sort,
  search,
  offset,
  limit,
  enableCache,
  enableTotal,
  contextName,
}
```

### Hooks

```js
const { document, loading, error, networkStatus, refetch } = useSingle2({ 
  collection, 
  fragmentName, 
  input, 
  pollInterval, 
  queryOptions, // passed to Apollo hook
});
```

```js
const { results, loading, error, networkStatus, refetch } = useMulti2({ 
  collection, 
  fragmentName, 
  input, 
  pollInterval, 
  queryOptions, // passed to Apollo hook
});
```

## Mutations

### Hooks

```js
const [ createFunction, { data, loading, error }] = useCreate2({ 
  collection, 
  fragmentName, 
  mutationOptions, // passed to Apollo hook
});

// to use the function: 
createFunction({ input });

```

```js
const [ updateFunction, { data, loading, error }] = useUpdate2({ 
  collection, 
  fragmentName, 
  mutationOptions, // passed to Apollo hook
});

// to use the function: 
updateFunction({ input });
```

```js
const [ deleteFunction, { data, loading, error }] = useDelete2({ 
  collection, 
  fragmentName, 
  mutationOptions, // passed to Apollo hook
});

// to use the function: 
deleteFunction({ input });
```

### Mutators

```js
await createMutator({
  collection, // collection containing document to mutate
  data, // data received from client
  currentUser, // current user
  validate, // boolean, whether to validate the operation
  context, // GraphQL context
});

await updateMutator({
  documentId // id of document to mutate
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
// movies.create.*
const properties = {
  data,
  originalData: clone(data),
  currentUser,
  collection,
  context,
  schema,
};

// movies.update.*
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

// movies.delete.*
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


