---
title: Data Layer
---

Nova uses the [Apollo](http://apollostack.com) data layer to manage its [GraphQL API](http://graphql.org/). 

Nova features a number of helpers to make settings up that data layer fast, most of which are initialized through the `Telescope.createCollection` function:

```js
const Movies = Telescope.createCollection({

  collectionName: 'movies',

  typeName: 'Movie',

  schema,
  
  resolvers,

  mutations,

});
```

The function takes the following arguments:

- `collectionName`: the name of the collection in your MongoDB database.
- `typeName`: the name of the GraphQL type that will be generated for the collection.
- `schema`, `resolvers`, `mutations`: see below.

<h2 id="schema">Schema</h2>

The first piece of any GraphQL API is the **schema**, which defines what data is made available to the client. 

In Nova, this GraphQL schema can be generated automatically from your collection's JSON schema, so you don't have to type things twice. Just pass a [SimpleSchema](https://github.com/aldeed/meteor-simple-schema)-compatible JSON schema as `Telescope.createCollection`'s `schema` property.

You can learn more on defining your schema in the [Schema](schema.html) section.

### Custom Schemas

If you need to manually add a schema, you can also do so using the `Telescope.graphQL.addSchema` function:

```js
const customSchema = `
  input Custom {
    _id: String
    userId: String
    title: String
  }
`;
Telescope.graphQL.addSchema(customSchema);
```

<h2 id="resolvers">Resolvers</h2>

You can pass a `resolver` object to `createCollection`. It should include three `list`, `single`, and `total` resolvers.

Each resolver should have the following properties:

- `name`: the resolver's name.
- `resolver`: the resolver function.

### List Resolver

The `list` resolver is used to display lists of documents. It takes the following arguments:

- `terms`: a JSON object containing a list of terms used to filter and sort the list.
- `offset`: how many documents to offset the list by (used for paginating requests).
- `limit`: how many documents to return.

It should return an array of documents.

### Single Resolver

The `single` resolver takes a `documentId` argument, and should return a single document.

### List Total Resolver

The `total` resolver takes a `terms` argument and should return the total count of results matching these terms in the database. 

### Custom Resolvers

Just like for the schema, you can also define resolvers manually using `Telescope.graphQL.addResolvers`:

```js
const movieResolver = {
  Movie: {
    user(movie, args, context) {
      return context.Users.findOne({ _id: movie.userId }, { fields: context.getViewableFields(context.currentUser, context.Users) });
    },
  },
};
Telescope.graphQL.addResolvers(movieResolver);
```

Resolvers can be defined on any new or existing type (e.g. `Movie`).

<h2 id="mutations">Mutations</h2>

Finally, `createCollection` also accepts a `mutations` object. This object should include three mutations, `new`, `edit`, and `remove`, each of which has the following properties:

- `name`: the name of the mutation.
- `check`: a function that takes the current user and (optionally) the document being operated on, and return `true` or `false` based on whether the user can perform the operation.
- `mutation`: the mutation function.

### New Mutation

Takes a single `document` argument.

### Edit Mutation

Takes three arguments:

- `documentId`: the document to modify.
- `set`: the fields to modify (as a list of field name/value pairs, e.g.`{title: 'My New Title', body: 'My new body'}`).
- `unset`: the fields to remove (as a list of field names/booleans, e.g. `{title: true, body: true}`).

### Remove Mutation

Takes a single `documentId` argument.

### Custom Mutations

You can also add your own mutations using `Telescope.graphQL.addMutation` and `Telescope.graphQL.addResolvers`:

```js

Telescope.graphQL.addMutation('postsVote(documentId: String, voteType: String) : Post');

const voteResolver = {
  Mutation: {
    postsVote(root, {documentId, voteType}, context) {
      // do mutation
    },
  },
};

Telescope.graphQL.addResolvers(voteResolver);
```

### Boilerplate Operations

Note that even when using the `new`, `edit`, and `remove` mutations, defining the actual operation is up to you.

But if you want to save time, you can import Telescope's own boilerplate mutations from `nova:lib`:

```js
import { newMutation, editMutation, removeMutation } from 'meteor/nova:lib';
```

They take the following arguments:

- `collection`: the collectin affected.
- `document` (new) or `documentId` (edit, remove): the document or document ID.
- `set`, `unset` (edit only): the `set` and `unset` objects. 
- `currentUser`: the user performing the operation.
- `validate`: whether to validate the operation based on the current user.
- `context`: the resolver context.

If `validate` is set to `true`, these boilerplate operations will: 

- Check that the current user has permission to insert/edit each field.
- Validate the document against collection schema.
- Add `userId` to document (insert only).
- Run any validation callbacks (e.g. `movies.new.validate`).

They will then run the mutation's document (or the `set` modifier) through the collection's sync callbacks (e.g. `movies.new.sync`), perform the operation, and finally run the async callbacks (e.g. `movies.new.async`).

Callback hooks run with the following arguments:

- `new`: `newDocument`, `currentUser`.
- `edit`: `modifier`, `oldDocument`, `currentUser`.
- `remove`: `document`, `currentUser`.

You can learn more about callbacks in the [Callbacks](callbacks.html) section. 