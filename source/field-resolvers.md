---
title: Field Resolvers
---

Although Vulcan uses SimpleSchema JSON schema to generate a GraphQL schema, it's important to understand that the two can be very different.

Note: this section will make more sense if you've already read the [Resolvers](/resolvers.html) section.

## Overview

For example, you might have a `userId` property in your `Movie` schema, but then you want this property to _resolve_ as a `user` object in the GraphQL schema (and in turn, in your client store). You can use `resolveAs` to accomplish this:

```
userId: {
  type: String,
  optional: true,
  canRead: ['guests'],
  resolveAs: {
    fieldName: 'user',
    arguments: 'foo: Int = 10',
    type: 'User',
    resolver: (movie, args, context) => {
      if (!movie.userId) return;
      const { currentUser, Users } = context;
      const user = await Users.loader.load(movie.userId);
      return Users.restrictViewableFields(currentUser, Users, user);
    },
    addOriginalField: true
  }
},
```

We are doing five things here:

1. Specifying that the field should be named `user` in the API.
2. Specifying that the field can take a `foo` argument that defaults to `10`.
3. Specifying that the `user` field returns an object of GraphQL type `User`.
4. Defining a `resolver` function that indicates how to retrieve that object.
5. Specifying that the original field (`userId`, with type `String`) should also be added to our GraphQL schema.

Note that it is recommended that you pick a **different** name for the "real" database field (`userId`) and the field in your schema (`user`), especially if you set `addOriginalField` to `true`. This way, you will be able to unambiguously require either the `userId` or the full `user` object just by specifying which field you need.

Also, the resolved type is a GraphQL type, not a primitive type or a schema. Therefore, it must necessarilly be a string (`'User'` in the previous example). If you rather need an array of user, the resolved type will be `'[User]'`. For primitive types, you must provide the name as a string, instead of the constructor : `'Date'` for a `Date`, `'String'` for a `String` etc.

## Custom Types

Creating a collection with `createCollection` will automatically create the associated GraphQL type, but in some case you might want to resolve a field to a GraphQL type that doesn't correspond to any existing collection.

Here's how the `vulcan:voting` package defines a new `Vote` GraphQL type:

```
import { addGraphQLSchema } from 'meteor/vulcan:core';

const voteSchema = `
  type Vote {
    itemId: String
    power: Float
    votedAt: String
  }

  union Votable = Post | Comment
`;

addGraphQLSchema(voteSchema);
```

Which can then be used as the value for `type` in a `resolveAs` field.

## GraphQL-Only Fields

Note that a field doesn't have to “physically” exist in your database to be able to generate a corresponding GraphQL field.

For example, you might want to query the Facebook API to get the number of likes associated with a post's URL:

```
likesNumber: {
  type: Number,
  optional: true,
  canRead: ['guests'],
  resolveAs: {
    type: 'Number',
    resolver: async (post, args, context) => {
      return await getFacebookLikes(post.url);
    }
  }
},
```

This will create a `likesNumber` field in your GraphQL schema (and your Apollo store) even though no such field has to exist in your database.

Note that for GraphQL only fields it is ok to leave out the `fieldName` property (which will default to using the same name as the schema field) since there is no actual database field of the same name.

## Relations

A common use case for field resolvers is fetching one or more item associated with the current document, such as the user corresponding to a post's `userId` field; or the array of categories correponding to its `categoriesIds` field. 

While you can achieve this by explicitly writing out the resolver as in the `userId` example above, Vulcan also offers a shortcut syntax using the `relation` property:

```
userId: {
  type: String,
  optional: true,
  canRead: ['guests'],
  resolveAs: {
    fieldName: 'user',
    type: 'User',
    relation: 'hasOne', // one of 'hasOne' or 'hasMany'
  }
},
```

Since this is a `hasOne` relation, Vulcan will assume that the current field stores the `_id` of the related document to look up; and because we know that the type of the returned item should be `User` we can also easily figure out that we need to search the `Users` collection.

In fact you can even leave out the `relation` property altogether, and Vulcan will do its best to guess the correct relation based on the type of the field (`hasOne` for regular fields, `hasMany` for arrays):

```
userId: {
  type: String,
  optional: true,
  canRead: ['guests'],
  resolveAs: {
    fieldName: 'user',
    type: 'User',
  }
},
```

### Relations, Cards, and Datatables

As an added bonus, if you define the `userId` field as having a relation to the `User` type (either implicitly or explicitly with the `relation` property), Cards and Datatables will be able to understand this relation and use smarter formatting when displaying this data. 

## Field Resolvers vs Denormalization

Another approach to achieve the same thing as field resolvers is denormalization, in other words "physically" storing the same information in your database. 

For example, assuming you wanted to show a post author's `displayName`, you could write a resolver that fetches the `user` object, or you could simply store a new `authorDisplayName` property on the post document directly. 

So how do you decide which approach to pick? Here are a few general guidelines. 

Resolvers are good when:

- The resolver doesn't require any extra database calls. For example, generating a `createdAtFormatted` date string from a `createdAt` timestamp.
- You want to avoid duplicating data. For example, copying a post author's entire `user` object on the post itself is probably a bad idea because that object can quickly get out of sync with the main `Users` document. 
- Keeping the denormalized data up to date gets too complex. If you find yourself writing three or four callbacks to update a single property, a resolver might turn out to be better. 

On the other hand, denormalization is better when:

- You want to sort documents by the field. Resolved field don't actually exist in your database, meaning you can't sort by them. 
- Generating the data through a resolver would require extra database calls. 
- The data changes infrequently. 
