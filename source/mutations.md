---
title: Mutations
---

A mutation is very similar to a query. It receives a GraphQL request, decides what to do with it, and then returns some data. But because mutations can result in _changes_ to your data, they have their own special `Mutation` type to differentiate them.

## GraphQL Mutations

`createCollection` also accepts a `mutations` object. This object should include three mutations, `new`, `edit`, and `remove`, each of which has the following properties:

* `name`: the name of the mutation.
* `check`: a function that takes the current user and (optionally) the document being operated on, and return `true` or `false` based on whether the user can perform the operation.
* `mutation`: the mutation function.

### New Mutation

Takes an object as argument with a single `document` property and returns a promise:

```js
this.props
  .newMutation({
    document: document
  })
  .then(/* success */)
  .catch(/* error */);
```

### Edit Mutation

Takes an object with three properties as an argument and returns a promise:

* `documentId`: the document to modify.
* `set`: the fields to modify (as a list of field name/value pairs, e.g.`{title: 'My New Title', body: 'My new body'}`).
* `unset`: the fields to remove (as a list of field names/booleans, e.g. `{title: true, body: true}`).

```js
this.props
  .editMutation({
    documentId: document._id,
    set: set,
    unset: unset
  })
  .then(/* success */)
  .catch(/* error */);
```

### Remove Mutation

Takes an object with a single `documentId` property as an argument and returns a promise.

```js
this.props
  .removeMutation({
    documentId: documentId
  })
  .then(/* success */)
  .catch(/* error */);
```

### Default Mutations

Vulcan provides a set of default New, Edit and Remove mutations you can use to save time:

```js
import {
  createCollection,
  getDefaultResolvers,
  getDefaultMutations
} from 'meteor/vulcan:core';
import schema from './schema.js';

const Movies = createCollection({
  collectionName: 'Movies',

  typeName: 'Movie',

  schema,

  resolvers: getDefaultResolvers('Movies'),

  mutations: getDefaultMutations('Movies')
});

export default Movies;
```

To learn more about what exactly the default mutations do, you can [find their code here](https://github.com/VulcanJS/Vulcan/blob/devel/packages/vulcan-core/lib/modules/default_mutations.js).

### Custom Mutations

You can also add your own mutations using `GraphQLSchema.addMutation` and `GraphQLSchema.addResolvers`:

```js
GraphQLSchema.addMutation(
  'postsVote(documentId: String, voteType: String) : Post'
);

const voteResolver = {
  Mutation: {
    postsVote(root, { documentId, voteType }, context) {
      // do mutation
    }
  }
};

GraphQLSchema.addResolvers(voteResolver);
```

## Higher-Order Components

#### `withNew`

This HoC takes the following two options:

* `collection`: the collection to operate on.
* `fragment`: specifies the data to ask for as a return value.

And passes on a `newMutation` function to the wrapped component, which takes a single `document` argument.

#### `withEdit`

Same options as `withNew`. The returned `editMutation` mutation takes three arguments: `documentId`, `set`, and `unset`.

#### `withRemove`

A single `collection` option. The returned `removeMutation` mutation takes a single `documentId` argument.

Note that when using the [Forms](forms.html) module, all three mutation HoCs are automatically added for you.

#### `withMutation`

The `withMutation` HoC provides an easy way to call a specific mutation on the server by letting you create ad-hoc mutation containers.

It takes two options:

* `name`: the name of the mutation to call on the server (will also be the name of the prop passed to the component).
* `args`: (optional) an object containing the mutation's arguments and types.

For example, here's how to wrap the `MyComponent` component to pass it an `addEmailNewsletter` function as prop:

```js
const mutationOptions = {
  name: 'addEmailNewsletter',
  args: { email: 'String' }
};
withMutation(mutationOptions)(MyComponent);
```

You would then call the function with:

```
this.props.addEmailNewsletter({email: 'foo@bar.com'})
```

## Mutators

A **mutator** is the function that actually does the work of mutating data on the server. As opposed to the _mutation_, which is usually a fairly light function called directly through the GraphQL API, a _mutator_ will take care of the heavy lifting, including validation, callbacks, etc., and should be written in such a way that it can be called from anywhere: a GraphQL API, a REST API, from the server directly, etc.

Vulcan features three standard mutators: `newMutator`, `editMutator`, and `removeMutator`. They are in essence thin wrappers around the standard Mongo `insert`, `update`, and `remove`.

These mutation functions should be defined _outside_ your GraphQL mutation resolvers, so that you're able to call them from outside a GraphQL context (for example, to seed your database through a server script).

They take the following arguments:

* `collection`: the collection affected.
* `document` (new) or `documentId` (edit, remove): the document or document ID.
* `set`, `unset` (edit only): the `set` and `unset` objects.
* `currentUser`: the user performing the operation.
* `validate`: whether to validate the operation based on the current user.
* `context`: the resolver context.

If `validate` is set to `true`, these boilerplate operations will:

* Check that the current user has permission to insert/edit each field.
* Validate the document against collection schema.
* Add `userId` to document (insert only).
* Run any validation callbacks (e.g. `movies.new.validate`).

They will then run the mutation's document (or the `set` modifier) through the collection's sync callbacks (e.g. `movies.new.sync`), perform the operation, and finally run the async callbacks (e.g. `movies.new.async`).

Callback hooks run with the following arguments:

* `new`: `newDocument`, `currentUser`.
* `edit`: `modifier`, `oldDocument`, `currentUser`.
* `remove`: `document`, `currentUser`.

You can learn more about callbacks in the [Callbacks](callbacks.html) section.

For example, here is the `Posts` collection's `new` mutation resolver, using the `newMutation` boilerplate mutation:

```js
import { newMutation } from 'meteor/vulcan:core';

const mutations = {
  new: {
    name: 'postsNew',

    check(user, document) {
      if (!user) return false;
      return Users.canDo(user, 'posts.new');
    },

    mutation(root, { document }, context) {
      performCheck(this.check, context.currentUser, document);

      return newMutation({
        collection: context.Posts,
        document: document,
        currentUser: context.currentUser,
        validate: true,
        context
      });
    }
  }
};
```

#### Alternative Approach

Instead of calling `newMutation`, `editMutation`, and `removeMutation`, you can call the usual `collection.insert`, `collection.update`, and `collection.remove`. But if you do so, make sure you perform the appropriate validation and security checks to verify if the operation should be allowed.
