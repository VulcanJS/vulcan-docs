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