---
title: Mutations
---

A mutation is very similar to a query. It receives a GraphQL request, decides what to do with it, and then returns some data. But because mutations can result in _changes_ to your data, they have their own special `Mutation` type to differentiate them.

## Mutation Steps

When talking about mutations, it's important to distinguish between the different elements that make up the overall process. For example, let's imagine that a user submits a form for creating a new document. 

1. First, this will **trigger a request** to your GraphQL endpoint, sent by Apollo Client. You can do this either by writing your own mutation higher-order component, or by using one of Vulcan's [pre-made mutation HoCs](#Mutation-Higher-Order-Components). 
2. When the GraphQL endpoint receives this request, it will look for a corresponding **mutation resolver**. Again you can either code your own resolver, or use Vulcan's [default mutations](#Default-Mutation-Resolvers).
3. The mutation resolver then calls a **mutator**. The mutator is the function that does the actual job of validating the request and mutating your data. The reason for this additional layer is that you'll often want to mutate data for *outside* your GraphQL API. By extracting that logic you're able to call the same exact mutator function whether you're inserting a new document sent by the front-end or, say, seeding your database with content extracted from an API. As usual, Vulcan offers a set of [default mutators](#Default-Mutators).
4. Finally, the mutator calls a [database connector](/database.html#Connectors) to perform the actual database operation. By abstracting out the database operation, we're able to make mutators (and by extension your entire GraphQL API) database-agnostic. This means that you can switch from MongoDB to MySQL without having to modify any of the above three layers. 

## Mutation Higher-Order Components

Vulcan includes three main default higher-order components to make calliing mutations from your React components easier. Note that when using the [Forms](forms.html) module, all three mutation HoCs are automatically added for you.

#### `withNew`

This HoC takes the following two options:

* `collection`: the collection to operate on.
* `fragment`: specifies the data to ask for as a return value.

And passes on a `newMutation` function to the wrapped component, which takes a single `document` argument.

Takes an object as argument with a single `document` property and returns a promise:

```js
this.props
  .newMutation({
    document: document
  })
  .then(/* success */)
  .catch(/* error */);
```

#### `withEdit`

Same options as `withNew`. The returned `editMutation` mutation takes three arguments: `documentId`, `set`, and `unset`.

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

#### `withRemove`

A single `collection` option. The returned `removeMutation` mutation takes a single `documentId` argument.

Takes an object with a single `documentId` property as an argument and returns a promise.

```js
this.props
  .removeMutation({
    documentId: documentId
  })
  .then(/* success */)
  .catch(/* error */);
```

#### `withMutation`

In addition to the three main mutation HoCs, The `withMutation` HoC provides an easy way to call a specific mutation on the server by letting you create ad-hoc mutation containers.

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


## Mutations Resolvers

When creating a new collection, `createCollection` accepts a `mutations` object. This object should include three mutations, `new`, `edit`, and `remove`, each of which has the following properties:

* `name`: the name of the mutation.
* `check`: a function that takes the current user and (optionally) the document being operated on, and return `true` or `false` based on whether the user can perform the operation.
* `mutation`: the mutation function.

### Default Mutation Resolvers

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

You can optionally pass a second `options` argument to `getDefaultMutations`, which can be used to override the `check` functions for each mutation: 

```js
const mutations = getDefaultMutations('Movies', {
  newCheck: (user, document) => ...,
  editCheck: (user, document) => ...,
  removeCheck: (user, document) => ...,
})
```

Vulcan's default mutations are fairly lightweight. For example here's the body of the `edit` mutation resolver (where `collectionName` is `getDefaultMutations`'s argument):

```js
async mutation(root, {documentId, set, unset}, context) {
  
  const collection = context[collectionName];

  // get entire unmodified document from database
  const document = await Connectors.get(collection, documentId);

  // check if user can perform operation; if not throw error
  Utils.performCheck(this.check, context.currentUser, document);

  // call editMutator boilerplate function
  return await editMutator({
    collection, 
    documentId: documentId, 
    set: set, 
    unset: unset, 
    currentUser: context.currentUser,
    validate: true,
    context,
  });
},
```

To learn more about what exactly the default mutations do, you can [find their code here](https://github.com/VulcanJS/Vulcan/blob/devel/packages/vulcan-core/lib/modules/default_mutations.js).

### Custom Mutations

You can also add your own mutations resolvers using `addGraphQLMutation` and `addGraphQLResolvers`:

```js
import { addGraphQLMutation, addGraphQLResolvers } from 'meteor/vulcan:core';

addGraphQLMutation(
  'postsVote(documentId: String, voteType: String) : Post'
);

const voteResolver = {
  Mutation: {
    postsVote(root, { documentId, voteType }, context) {
      // do mutation
    }
  }
};

addGraphQLResolvers(voteResolver);
```

## Mutators

A **mutator** is the function that actually does the work of mutating data on the server. As opposed to the _mutation_, which is usually a fairly light function called directly through the GraphQL API, a _mutator_ will take care of the heavy lifting, including validation, callbacks, etc., and should be written in such a way that it can be called from anywhere: a GraphQL API, a REST API, from the server directly, etc.

### Default Mutators

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

For example, here is the `Posts` collection's `new` mutation resolver, using the `newMutator` boilerplate mutation:

```js
import { newMutator } from 'meteor/vulcan:core';

const mutations = {
  new: {
    name: 'postsNew',

    check(user, document) {
      if (!user) return false;
      return Users.canDo(user, 'posts.new');
    },

    mutation(root, { document }, context) {
      performCheck(this.check, context.currentUser, document);

      return newMutator({
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

### Mutator Callbacks

Default mutators create the following callback hooks for every collection: 

- `collection.operation.validate`: called to validate the document or modifier. 
- `collection.operation.before`: called before the database operation.
- `collection.operation.after`: called after the database operation, but before the mutator returns.
- `collection.operation.async`: called in an async manner after the mutator returns. 

You can learn more about callbacks in the [Callbacks](callbacks.html) section.

### Custom Mutators

If you're writing your own resolvers you can of course also write your own mutators, either by using Vulcan's [Connectors](/database.html#Connectors) or even by accessing your database directly. 

One thing to be aware of though is that by doing this you'll bypass any callback hooks used by the default mutators, and you'll also have to take care of your own data validation. 