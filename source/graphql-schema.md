---
title: GraphQL Schema
---

The first piece of any GraphQL API is the **schema**, which defines what data is made available to the client.

In Vulcan, this GraphQL schema can be generated automatically from your collection's JSON schema, so you don't have to type things twice. Just pass a [SimpleSchema](https://github.com/aldeed/node-simple-schema)-compatible JSON schema as `createCollection`'s `schema` property.

## Viewing the Schema

You can view your GraphQL schema using the Meteor shell. First, launch the shell from your terminal with:

```
meteor shell
```

Then, type:

```
Vulcan.getGraphQLSchema()
```

## Custom Schemas

If you need to manually add a schema, you can also do this using the `GraphQLSchema.addSchema` function which will add one or more [GraphQL schemas](http://graphql.org/learn/schema/) to your GraphQL API:

```js
const customSchema = `
  input Custom {
    _id: String
    userId: String
    title: String
  }
`;
GraphQLSchema.addSchema(customSchema);
```
