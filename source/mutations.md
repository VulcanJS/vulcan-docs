---
title: Mutations
---

## GraphQL Mutations

`createCollection` also accepts a `mutations` object. This object should include three mutations, `new`, `edit`, and `remove`, each of which has the following properties:

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

You can also add your own mutations using `GraphQLSchema.addMutation` and `GraphQLSchema.addResolvers`:

```js
GraphQLSchema.addMutation('postsVote(documentId: String, voteType: String) : Post');

const voteResolver = {
  Mutation: {
    postsVote(root, {documentId, voteType}, context) {
      // do mutation
    },
  },
};

GraphQLSchema.addResolvers(voteResolver);
```

### Higher-Order Components

#### `withNew`

This HoC takes the following two options:

- `collection`: the collection to operate on.
- `fragment`: specifies the data to ask for as a return value.

And passes on a `newMutation` function to the wrapped component, which takes a single `document` argument.

#### `withEdit`

Same options as `withNew`. The returned `editMutation` mutation takes three arguments: `documentId`, `set`, and `unset`. 

#### `withRemove`

A single `collection` option. The returned `removeMutation` mutation takes a single `documentId` argument. 

Note that when using the [Forms](forms.html) module, all three mutation HoCs are automatically added for you. 

## Boilerplate Mutations

Vulcan features three standard `newMutation`, `editMutation`, and `removeMutation` mutations that are in essence thin wrappers around the standard Mongo `insert`, `update`, and `remove`. 

Note that in the context of this section, “mutation” refers to the function that performs the actual server-side database operation, and not to the [GraphQL mutations](mutations.html). 

These mutation functions should be defined *outside* your GraphQL mutation resolvers, so that you're able to call them from outside a GraphQL context (for example, to seed your database through a server script).

They take the following arguments:

- `collection`: the collection affected.
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

For example, here's the `Posts` collection's `new` mutation resolver, using the `newMutation` boilerplate mutation:

```js
import { newMutation } from 'meteor/nova:core';

const mutations = {

  new: {
    
    name: 'postsNew',
    
    check(user, document) {
      if (!user) return false;
      return Users.canDo(user, 'posts.new');
    },
    
    mutation(root, {document}, context) {
      
      performCheck(this, context.currentUser, document);

      return newMutation({
        collection: context.Posts,
        document: document, 
        currentUser: context.currentUser,
        validate: true,
        context,
      });
    },

  }

};
```

#### Alternative Approach

Instead of calling `newMutation`, `editMutation`, and `removeMutation`, you can call the usual `collection.insert`, `collection.update`, and `collection.remove`. But if you do so, make sure you perform the appropriate validation and security checks to verify if the operation should be allowed. 
