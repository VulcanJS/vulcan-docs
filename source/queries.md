---
title: Queries
---

When using Vulcan's default resolvers, two types of queries are automatically generated for every collection: the `single` query (which returns a single document) and the `multi` query (which returns a list of documents). 

A set of default query resolvers is generated on the server to power these queries; and a set of client-side higher-order components and hooks is provided to make working with those queries easier. 

## API

All Vulcan queries follow the single argument pattern. In other words, they take a single input argument that encapsulates all of the query's variables; and they return a single output object that contains all the query data.

### Single Query

Here is an example single query definition for a `Movie` type:

```
movie(input: SingleMovieInput): SingleMovieOutput
```

And here's an example query:

```
query myMovie {
  movie(input: { where: { name: { _eq: "Die Hard" } } } ) {
    result {
      _id
      name
      year
      description
    }
  }
}
```

### Multi Query

Here is an example multi query definition for a `Movie` type:

```
movies(input: MultiMovieInput): MultiMovieOutput
```

And here's an example query:

```
query myMovies {
  movies(input: { where: { year: { _eq: 1999 } } } ) {
    results {
      _id
      name
      year
      description
    }
  }
}
```

### Input Argument

Both single and multi queries take the same arguments (with the exception of `limit`, which is only available for the multi query). The only difference is that in the case of the single query, if the `where` argument selects more than one document only the first document of the set will be returned. 

The `input` takes the following properties:

- `where`: the object that lets you select the specific documents to return. 
- `orderBy`: how to order the results.
- `limit`: how many results to return.
- `offset`: how many results to skip.
- `search`: a shortcut for searching across all searchable fields at once.
- `_id`: a shortcut for selecting a specific document by `_id`. 
- `enableCache`: whether to enable caching for this query
- `allowNull`: whether to return `null` instead of throwing an error when no document is found.

Here is an example input type for the `Movie` type:

```
input SingleMovieInput {
  where: MovieWhereInput
  orderBy: MovieOrderByInput
  search: String
  _id: String
  enableCache: Boolean
  allowNull: Boolean
}
```

You can learn more about these arguments in the [filtering](/filtering.html) section. 

## Client

Vulcan also offers built-in client-side helpers that will generate the right GraphQL query for you. 

### Hooks

#### useSingle

TODO

#### useMulti

TODO

### Higher-Order Components

To make working with Apollo easier, Vulcan provides you with a set of higher-order components (HoCs).

An **HoC** is simply a function you can call on a React component to give it additional props. Think of it as a mechanic's assistant following along with a little toolbox, ready to hand over a screwdriver or socket wrench at the appropriate time.

#### withMulti

The `withMulti` HoC is used to display lists of documents. It takes the following options:

##### Options

* `collection`: the collection on which to look for the `list` resolver.
* `collectionName`: alternatively, you can also pass the collection name as a string.
* `fragment` or `fragmentName`: the fragment to use. If you pass `fragmentName` instead of `fragment`, the name you passed will be used to look up a fragment registered with `registerFragment`.
* `input`: an optional `input` object used to filter the data received (see [filtering](/filtering.html) section). This argument can also be passed dynamically as a prop (see below).

For example:

```js
const listOptions = {
  collection: Movies,
  fragmentName: 'MoviesItem',
};

export default withMulti(listOptions)(movies);
```

##### Accepted Props

The resulting wrapped component also accepts a *dynamic* `input` object as a prop.

This can be useful when you want your data to change based on some user action such as selecting filtering options in a dropdown, changing sorting order, etc.

The dynamic `input` will always take priority over any “static” `input` defined in the HoC's initial `options`. 

##### Passed-on Props

The HoC then passes on the following props to the wrapped component:

* `loading`: `true` while the data is loading.
* `results`: the loaded array of documents.
* `totalCount`: the total count of results.
* `count`: the number of current results (i.e. `results.length`).
* `refetch`: a function that can be called to trigger a query refetch.
* `loadMore`: a function that can be called to load more data.

#### withSingle

The `withSingle` HoC displays a single document. 

##### Options

It takes the same options as `withMulti`, with the addition of `_id`.

##### Accepted Props

It takes the same props as `withMulti`, with the addition of `_id`.

##### Passed-on Props

This HoC then passes on the following child prop:

* `loading`: `true` while the data is loading.
* `document`: the loaded document.
* `refetch`: a function that can be called to trigger a query refetch.

## Server

### Resolvers

As a time-saving convention, Vulcan makes available two default resolvers:

* `multi`: used to return a list of documents matching a set of arguments, as well as the _total number_ of documents matching the same arguments.
* `single`: used to return a single document matching an `_id` or `slug`

You can pass a `resolver` object to `createCollection` when initializing your collection. It should include these two `multi` and `single` resolvers.

Each resolver should have the following properties:

* `name`: the resolver's name.
* `resolver`: the resolver function.

Note that if the collections include private data, you'll have to implement your own security layer inside each resolver to prevent unwanted access. Check out the [Controlling Viewing](/groups-permissions.html#Controlling-Viewing) section for more details.

#### Multi Resolver

The `multi` resolver is used to display lists of documents. It takes the following arguments:

* `terms`: a JSON object containing a list of terms used to filter and sort the list.
* `offset`: how many documents to offset the list by (used for paginating requests).
* `limit`: how many documents to return.

It should return an array of documents.

#### Single Resolver

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

### Custom Resolvers

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
