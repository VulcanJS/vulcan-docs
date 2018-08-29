---
title: Querying on the Server
---

Most of the Data Layer section's contents applies to the client. After all, if you're on the server, you can simply connect to your database directly, so why would you need to worry about the data layer?

But the truth is that connecting to the database directly can lead to problems. For example, let's assume you have a `user` resolver on your `Post` type that gives you access to the post's author. On the client, you could write:

```js
const authorName = post.user.displayName;
```

On the server, it's easy enough to obtain that post document:

```js
const post = Posts.findOne(documentId);
```

But the document you fetch directly from MongoDB won't have a `user` property, let alone a `user.displayName` (since those are provided by the GraphQL resolver).

Thankfully, VulcanJS features a few special utilities to let you query for data through GraphQL *on the server*

### runGraphQL(query, variables)

`runGraphQL` lets you execute any GraphQL query or mutation against your schema and get its result back on the server. 

```js
import { runGraphQL } from 'meteor/vulcan:core';

async function foo = () => {
  
  const query = `
    query OneRoom($documentId: String){
      RoomsSingle(documentId: $documentId){
        name
        description
        user{
          _id
          displayName
        }
      }
    }
  `;
  return await runGraphQL(query, {documentId: '123foo'});

}
```

(Note that you'll need to be inside an `async` context to use the `await` keyword)

### collection.queryOne(documentId, options)

A common need on the server is fetching a single document by its `_id`. You can do so with `collection.queryOne`. It takes an `options` object with a `fragmentText` property. If no fragment is provided, the query will use the collection's `defaultFragment`. 

```js
const user = await Users.queryOne(userId);

// or

const user = await Users.queryOne(userId, {
  fragmentText: `
    fragment MyUsersFragment on User {
      _id
      username
      createdAt
      posts{
        _id
        title
      }
    }
  `
});
```

### `queryOne` vs `load`

It might seem like `queryOne` does the same thing as the [Dataloader layer](/performance.html#Caching-amp-Batching)'s `load`, but there's a crucial difference in how they're used. 

`queryOne` uses your existing GraphQL resolvers behind the scenes, which means it itself can't be used *inside* a resolver (or you'd risk an infinite loop). It's thus meant to be used from outside of your GraphQL resolver tree, from a server script, cron job, migration, etc. or any kind of server code unrelated to the client. 

`load` on the other hand is specifically made to be used inside resolvers and add a caching and batching layer to them. 