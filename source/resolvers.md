---
title: Resolvers
---

A resolver is the function on the server that receives a GraphQL query, decides what to do with it (how to _resolve_ it), and then returns some data.

Each level of a GraphQL query (in other words, each field) can have its own custom resolver; or if no resolver exists the field will default to look for a value on its parent object.

## Data Flow

When you connect to a GraphQL-powered site, the client makes an API request to load the data it needs.

Unlike traditional REST APIs, the client doesn't just hit an API endpoint but actually sends a list of _what data it needs_ to the server. The specific document properties needed by the client are defined in a [fragment](/fragments.html).

Upon receiving that query, the server checks that the main query exists in the [GraphQL schema](/graphql-schema.html), and also validates the fragment against it.

If that validation passes, the server then looks for a **query resolver function** that will specify how to actually retrieve the data (presumably through some kind of database `find()` operation).

It will then take that data, strip any fields not specified in the fragment, and return the result to the client.

### Implicit Resolvers

Technically speaking, every single field in the fragment has to resolve to _something_. But if for example you've specified a `Post` resolver and you're asking for a post's `title` and `createdAt` properties, GraphQL will be smart enough to look for `post.title` and `post.createdAt` without you needing to define additional resolvers for these two fields.

That being said if you wanted `createdAt` to resolve to, say, `formatDate(post.createdAt)` instead of just `post.createdAt`, then you could write a custom resolver [just for that specific field](/field-resolvers.html).

## Collection Resolvers

As a time-saving convention, Vulcan makes the assumption that you as a developer will provide it with three "default" resolvers:

* `multi`: used to return a list of documents matching a set of arguments, as well as the _total number_ of documents matching the same arguments.
* `single`: used to return a single document matching an `_id` or `slug`

You can pass a `resolver` object to `createCollection` when initializing your collection. It should include these two `multi` and `single` resolvers.

Each resolver should have the following properties:

* `name`: the resolver's name.
* `resolver`: the resolver function.

Note that if the collections include private data, you'll have to implement your own security layer inside each resolver to prevent unwanted access. Check out the [Controlling Viewing](/groups-permissions.html#Controlling-Viewing) section for more details.

### Multi Resolver

The `multi` resolver is used to display lists of documents. It takes the following arguments:

* `terms`: a JSON object containing a list of terms used to filter and sort the list.
* `offset`: how many documents to offset the list by (used for paginating requests).
* `limit`: how many documents to return.

It should return an array of documents.

### Single Resolver

The `single` resolver takes a `documentId` argument, and should return a single document. If no `documentId` is provided or the id does not match any existing document, it should either return null or throw an error depending on your use case.

If you need to get only one element but you don't know its id, for example to display the most recent article on your website, the preferred pattern is to use the `multi` resolver with a limit of 1.

### Default Resolvers

Vulcan provides a set of default List, Single and Total resolvers you can use to save time by calling `getDefaultResolvers(collectionName)`:

```js
import {
  createCollection,
  getDefaultResolvers,
  getDefaultMutations
} from 'meteor/vulcan:core';
import schema from './schema.js';

const Movies = createCollection({
  typeName: 'Movie',

  schema,

  resolvers: getDefaultResolvers(options),

  mutations: getDefaultMutations(options)
});

export default Movies;
```

The `options` object can have the following properties: 

- `typeName` (String): the resolver's type name (required).
- `cacheMaxAge` (Number): a custom cache age (in seconds) for the resolvers. 

To learn more about what exactly the default resolvers do, you can [find their code here](https://github.com/VulcanJS/Vulcan/blob/devel/packages/vulcan-core/lib/modules/default_resolvers.js).

## Custom Resolvers

In some cases, you will want to go beyond the three default resolvers. For example, you might want to create a resolver that returns a single random post, or the current user. You can do so by using `addGraphQLResolvers` and `addGraphQLQuery`:

```js
import { addGraphQLResolvers, addGraphQLQuery } from 'meteor/vulcan:core';

const currentUserResolver = {
  Query: {
    currentUser(root, args, context) {
      return context.Users.findOne({ _id: args.userId });
    }
  }
};
addGraphQLResolvers(currentUserResolver);

addGraphQLQuery(`currentUser: User`);
```

`addGraphQLResolvers` adds the actual resolver function, while `addGraphQLQuery` adds the corresponding **query type** (if your new resolver is not mentioned in your GraphQL schema, you won't be able to query it).

Learn more about resolvers in the [Apollo documentation](http://dev.apollodata.com/tools/graphql-tools/resolvers.html).

## Higher-Order Components

To make working with Apollo easier, Vulcan provides you with a set of higher-order components (HoCs).

An **HoC** is simply a function you can call on a React component to give it additional props. Think of it as a mechanic's assistant following along with a little toolbox, ready to hand over a screwdriver or socket wrench at the appropriate time.

### withMulti

The `withMulti` HoC is used to display lists of documents. It takes the following options:

* `collection`: the collection on which to look for the `list` resolver.
* `fragment` or `fragmentName`: the fragment to use. If you pass `fragmentName` instead of `fragment`, the name you passed will be used to look up a fragment registered with `registerFragment`.

For example:

```js
const listOptions = {
  collection: Movies,
  fragmentName: 'MoviesItem',
};

export default withMulti(listOptions)(movies);
```

The resulting wrapped component also takes in the following options:

* `terms`: an object containing a list of querying, sorting, and filtering terms.

Note that `terms` needs to be passed not as an option, but as a prop from the _parent_ component.

The HoC then passes on the following props:

* `loading`: `true` while the data is loading.
* `results`: the loaded array of documents.
* `totalCount`: the total count of results.
* `count`: the number of current results (i.e. `results.length`).
* `refetch`: a function that can be called to trigger a query refetch.
* `loadMore`: a function that can be called to load more data.

### withSingle

The `withSingle` HoC displays a single document. It takes the same options as `withMulti`, but takes a `documentId` **prop**.

Like `terms` for `withMulti`, `documentId` needs to be passed not as an option, but as a prop from the _parent_ component.

This HoC then passes on the following child prop:

* `loading`: `true` while the data is loading.
* `document`: the loaded document.

### Data Updating

As long as you use `withMulti` in conjunction with `withCreate`, `withUpdate`, and `withDelete`, your lists will automatically be updated after any mutation. This includes:

* Inserting new items in lists after they're inserted.
* Removing items when they're removed.
* Reordering lists when an item is edited in a way that changes a sort.
* Removing an item after it's been edited when it doesn't match a list's filters anymore.

At this time, it's only possible to benefit from this auto-updating behavior if you're using the three built-in mutation HoCs, although making `withMulti` more flexible towards custom mutations is on the roadmap (PRs welcome!).

#### Alternative Approach

You can replace any of Vulcan's generic HoCs with your own tailor-made HoCs (whether it is for queries or mutations) using the `graphql` utility. Note that if you do so, you will need to manually [update your queries](http://dev.apollodata.com/react/cache-updates.html) after each mutation.
