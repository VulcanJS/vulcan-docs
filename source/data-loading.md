---
title: Data Loading
---

## Data Flow

When you connect to a GraphQL-powered site, the client makes an API request to load the data it needs. 

Unlike traditional REST APIs, the client doesn't just hit an API endpoint but actually sends a list of *what data it needs* to the server. The specific document properties needed by the client are defined in a **fragment**. 

Upon receiving that query, the server validates the fragment against the main GraphQL **schema**.

If that validation passes, the server then looks for a **query resolver** that will specify how to actually retrieve the data (presumably through some kind of database `find()` operation).

Data mutations (inserting, editing, or removing a document) function in a similar way.

The client sends a query that includes the name of the mutation, the mutation arguments, as well as specifies what data the client expects back once the mutation is complete. 

The server then looks for a **mutation resolver** and executes it.



## Resolvers

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

Just like for the schema, you can also define resolvers manually using `GraphQLSchema.addResolvers`:

```js
const movieResolver = {
  Movie: {
    user(movie, args, context) {
      return context.Users.findOne({ _id: movie.userId }, { fields: context.getViewableFields(context.currentUser, context.Users) });
    },
  },
};
GraphQLSchema.addResolvers(movieResolver);
```

Resolvers can be defined on any new or existing type (e.g. `Movie`).

### Custom Queries

You can also use `GraphQLSchema.addQuery` to define your own queries:

```js
GraphQLSchema.addQuery(`currentUser: User`);
```

### Caching & Batching

You can optionally use [Dataloader](https://github.com/facebook/dataloader) inside your resolvers to get server-side better performance through caching and batching.

To understand how this works, let's suppose you're displaying five posts on your homepage, each of which has an author. 

Without batching, this would result in one database call to fetch the five posts, then one call per post to fetch the author as each `Post.author` resolver is called, for a total of six database queries. 

With batching enabled, these five calls to the `users` collection are batched together, for a total of two database calls. 

Additionally, with caching enabled, any queries for the same documents (for example, your posts also have `commenters`) won't hit the database at all but instead load data from Dataloader's cache. 

Note that the cache is **per-request**, meaning that it will not persist across multiple reloads or different users (which would otherwise lead to outdated data).

To load data from Dataloader instead of the database, you can use the following two functions:

- `collection.loader.load(_id)`: takes the `_id` of a document.
- `collection.loader.loadMany(_ids)`: takes an array of `_id`s. 

If the documents requested are not present in the cache, Dataloader will automatically query the database for them. 

Aditionally, you can also manually add documents to the cache with:

- `collection.loader.prime(_id, document)`

Finally, note that `load` and `loadMany` can only take `_id`s, and Mongo selectors. In those cases, you can simply keep querying the database directly with `collection.find()` and   `collection.findOne()`. 

## Higher-Order Components


To make working with Apollo easier, Vulcan provides you with a set of higher-order components (HoC). 

An HoC is simply a function you can call on a React component to give it additional props.

### withList

The `withList` HoC is used to display lists of documents. It takes the following options:

- `queryName`: an arbitrary name for the query.
- `collection`: the collection on which to look for the `list` resolver.
- `fragment` or `fragmentName`: the fragment to use. If you pass `fragmentName` instead of `fragment`, the name you passed will be used to look up a fragment registered with `registerFragment`. 

For example:

```js
const listOptions = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragment: MoviesItem.fragment,
};

export default withList(listOptions)(MoviesList);
```

The resulting wrapped component also takes in the following options:

- `terms`: an object containing a list of querying, sorting, and filtering terms.

Note that `terms` needs to be passed not as an option, but as a prop from the *parent* component.

The HoC then passes on the following props:

- `loading`: `true` while the data is loading.
- `results`: the loaded array of documents.
- `totalCount`: the total count of results.
- `count`: the number of current results (i.e. `results.length`).
- `refetch`: a function that can be called to trigger a query refetch.
- `loadMore`: a function that can be called to load more data. 

### withDocument

The `withDocument` HoC displays a single document. It takes the same options as `withList`, but takes a `documentId` **prop**. 

Like `terms` for `withList`, `documentId` needs to be passed not as an option, but as a prop from the *parent* component.

This HoC then passes on the following child prop:

- `loading`: `true` while the data is loading.
- `document`: the loaded document.


### Data Updating

As long as you use `withList` in conjunction with `withNew`, `withEdit`, and `withRemove`, your lists will automatically be updated after any mutation. This includes:

- Inserting new items in lists after they're inserted.
- Removing items when they're removed.
- Reordering lists when an item is edited in a way that changes a sort.
- Removing an item after it's been edited when it doesn't match a list's filters anymore. 

At this time, it's only possible to benefit from this auto-updating behavior if you're using the three built-in mutation HoCs, although making `withList` more flexible towards custom mutations is on the roadmap (PRs welcome!).

#### Alternative Approach

You can replace any of Vulcan's generic HoCs with your own tailor-made HoCs (whether it's for queries or mutations) using the `graphql` utility. Note that if you do so, you will need to manually [update your queries](http://dev.apollodata.com/react/cache-updates.html) after each mutation. 
