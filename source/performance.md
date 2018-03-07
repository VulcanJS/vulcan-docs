---
title: Performance
---

GraphQL makes it easy to create rich schemas with multiple nested layers, but that can also come at a performance cost. Thankfully, Vulcan offer you a few ways to get the benefits of GraphQL without suffering too much of a slow-down. 

## Caching & Batching

You can optionally use [Dataloader](https://github.com/facebook/dataloader) inside your resolvers to get better server-side performance through caching and batching.

To understand how this works, let's suppose you're displaying five posts on your homepage, each of which has an author. 

Without batching, this would result in one database call to fetch the five posts, then one call per post to fetch the author as each `Post.author` resolver is called, for a total of six database queries. 

With batching enabled, these five calls to the `users` collection are batched together, for a total of two database calls. 

Additionally, with caching enabled, any queries for the same documents (for example, your posts also have `commenters`) won't hit the database at all but instead load data from Dataloader's cache. 

Note that the cache is **per-request**, meaning that it will not persist across multiple reloads or different users (which would otherwise lead to outdated data).

To load data from Dataloader instead of the database, you can use the following two functions:

- `collection.loader.load(_id)`: takes the `_id` of a document.
- `collection.loader.loadMany(_ids)`: takes an array of `_id`s. 

If the documents requested are not present in the cache, Dataloader will automatically query the database for them. 

Additionally, you can also manually add documents to the cache with:

- `collection.loader.prime(_id, document)`

Finally, note that `load` and `loadMany` can only take `_id`s. If you instead need to query for data using more complex Mongo selectors, you can simply keep querying the database directly with `collection.find()` and `collection.findOne()`. 

Learn more: [Using Dataloader to batch and cache database calls](https://www.youtube.com/watch?v=55Ep5KBTIQE).


## Performance Optimization

If you use complex GraphQL queries and resolvers a lot, you might notice some slow loading times. Here's a few things you can try to speed things up.

- [Video Case Study: GraphQL Performance Optimization](https://www.youtube.com/watch?v=M8Jmz8q2sUk)

#### Add Mongo Indexes

Make sure you're adding indexes for any frequently used Mongo selectors. For example, if you're often looking up all posts associated with a specific user ID, you'll want to add an index for that:

```js
Posts.rawCollection().createIndex({userId: 1});
```

#### Use Dataloader

Dataloader ensures your data is loaded from your server's in-memory cache instead of the database whenever possible, leading to much faster load times. 

You can use it whenever you're requesting a single document or multiple documents based on their `_id`:

```js
// without Dataloader
const room = Rooms.findOne(booking.roomId);

// with Dataloader
const room = await Rooms.loader.load(booking.roomId);
```

Multiple documents:

```js
// without Dataloader
const photos = Photos.find(post.photoIds).fetch();

// with Dataloader
const photos = await Rooms.loader.loadMany(post.photoIds);
```

#### Remove Unnecessary Fields

Sometimes you might be asking for a field you don't actually need, especially if you reuse the same fragments in multiple places. Always make sure you don't load any data you don't use, especially if these fields in turn trigger their own resolvers.

#### Denormalize Data

Although GraphQL makes it possible to keep data normalized inside your database in theory, in practice it can still be better for performance to denormalize (i.e. copy) some properties. 

For example, let's imagine you want to know how many comments a user has made. You could definitely have a `commentCount` property with its own custom resolver that counts all comments with the right `userId`. 

But on the other hand, it would be far better from a performance point of view to simply store that information as a field on each user document and increment/decrement it every time a user creates or deletes a comment. 

#### Ensure Fast Server-DB Speeds

At some point, the last bottleneck is the time it takes for the server to get data back from the database (assuming you're not hosting both on the same instance). 

For example, if you're in Tokyo and your database is too, but your server is in New York, your data has to go from Tokyo to New York then back to Tokyo, leading to slower load times than if everything was just stored in New York.

For that reason, try to have both your server and database stored within the same Amazon (or equivalent) server cluster. 

#### Use Caching

Finally, services like [Apollo Engine](https://www.apollographql.com/engine/) make it easy to implement a global [caching layer](https://github.com/apollographql/engine-docs/blob/master/source/caching.md) by using [Apollo Cache Control](https://github.com/apollographql/apollo-cache-control).

Unlike Dataloader's cache, this is not a temporary field-by-field cache; but a persistent cache that caches the entire GraphQL response. 
