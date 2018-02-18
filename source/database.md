---
title: Database Layer
---

Vulcan includes its own full-featured data layer (consiting of default resolvers and default mutations) that will let you avoid writing any database code most of the time. 

That being said, there will still be cases where you'll need to access your database directly. 

## Non-Database Methods

Before addressing how to make database calls, it's worth asking if you should be calling the database directly in the first place. Vulcan features two indirect ways to query for data from the server that, while they still call the database behind the scenes, let you make your app more "database-agnostic" by helping you limit any database-specific code to a few specific places. 

### Server Queries

Before discussing databases, it's important to remember that a document extracted from your database will often be different from the same document as provided by the GraphQL API; since the latter can feature [special GraphQL-only fields](/field-resolvers.html#GraphQL-Only-Fields).

For that reason, Vulcan also offers [server-side GraphQL queries](/server-queries.html) you can use to fetch the “GraphQL version” of a document, even on the server. This will in turn call the same GraphQL resolvers that would be normally called when receiving a query from the client. 

As server-side queries rely on the same resolvers as normal GraphQL queries, you can't use them inside any resolver (or you'd risk an infinite loop). 

### Using Dataloder

Instead, inside GraphQL resolvers you'll usually want to go through Vulcan's [Dataloader layer](/performance.html#Caching-amp-Batching) instead of calling the database directly, to make sure you don't make extra database calls.

## Connectors

In Vulcan you usually don't write any database code directly, even when you do want to perform a database operation. Instead, you call a **connector**, which is the function that actually specifies how to perform the operation. 

For example, if you need to create a new `Movies` document you can call the `create` connector: 

```js
const database = 'mongo';

Connectors[database].create(Movies, { name: 'Titanic'});
```

Which itself will call Mongo's `insert`: 

```js
Connectors.mongo = {
  create: async (collection, document, options) => {
    return await collection.insert(document);
  }
}
```

The advantage of this approach is that if you one day want to use MySQL instead of Mongo, you can simply change the value of `database` in `Connectors[database]` and let the SQL connector do the rest. 

The following connectors are available: 

- `await get(collection, selector, options)`: return a single document.
- `await find(collection, selector, options)`:  return a list of documents.
- `await count(collection, selector, options)`: return a count of documents matching the selector.
- `await create(collection, document, options)`: create a new document.
- `await update(collection, selector, modifier, options)`: update a document.
- `await delete(collection, selector, options)`: delete a document.

Note that selectors can either be objects or `_id` strings. 

Of course, if you don't foresee the need to support multiple databases in your own code, making direct databases call is also perfectly valid. 

## Databases

### MongoDB

Vulcan is powered by Meteor, which itself comes bundled with MongoDB. 

Meteor collections (and by extension Vulcan collections) support all common MongoDB operations, such as:

- `collection.find()`
- `collection.findOne()`
- `collection.update()`
- `collection.remove()`

### Other Databases

Vulcan does not currently include out-of-the-box support for other databases. That being said, just like you can make a Mongo call in your resolver, you can also call SQL or any other data source as you normally would in any Node app. 

In that regard, Vulcan is fairly database-agnostic, although it's true that the default resolvers and mutations only currently work with MongoDB. 