---
title: Custom Queries & Aggregations
---

#### ⭐️ New in Vulcan 1.17.0

Vulcan makes it really easy to query and mutate collections using GraphQL. But sometimes you want to query data on the server with a schema that does not match a database collection - for example an aggregation or an array of objects. `registerCustomQuery()` allows you to add custom GraphQL types from a schema object and resolvers to query the data.

## Registering a Custom Query

You can register a custom query using `registerCustomQuery()` on the server:

```js
// on the server
registerCustomQuery({
  typeName: 'Agent',
  description: 'Query resolved from an in-memory array of Agents on the server',
  resolver,
  schema: agentSchema,
  filterable,
});
```

This function generates and configures the following:

 * The GraphQL type
 * The filter input object
 * The sort input object
 * The multi input object
 * The multi output object
 * The query
 * The resolver
 * The default fragment (see )


The function takes the following arguments:

 * `typeName`: the name of the GraphQL type that will be generated - the same as when [creating a collection](/schemas.html#Creating-Collections)
 * `description`: a description to document the schema
 * `resolver`: the query resolver function
 * `schema`: (optional) a JavaScript object containing a list of fields defining the [schema](/schema-properties.html) of the new type
 * `graphQLType`: (optional) instead of a `schema`, you can provide a `graphQLType` as a template literal
 * `filterable`: an array of filterable fields, each one an object with a `name` and GraphQL `type`

## Defining the Resolver

The resolver is an async function with the usual [resolver arguments](https://www.apollographql.com/docs/apollo-server/data/resolvers/#resolver-arguments). It should return an object with the `results` and `totalCount`.

In this example `agents` exists as an array of agent objects defined on the server. We will later query it from the client using `withMulti`.

```js
// on the server
import { registerCustomQuery } from 'meteor/vulcan:core';
import { agents } from './agents';
import agentSchema from './schema';
import _map from 'lodash/map';
import _filter from 'lodash/filter';

const resolver = async function (root, { input }, context, info) {
  const { search } = input;

  let agents = _map(agents, (agent, key) => {
    return {
      _id: agent.id,
      title: agent.title,
      description: agent.description,
    };
  });

  agents = _filter(agents, agent => agent.title.includes(search) || agent.description.includes(search));

  return { results: agents, totalCount: agents.length };
};
```

The resolver should enforce permissions using `context.currentUser`.

## Providing a Schema

You specify the schema the same way as you would for a [collection](/schemas.html#Creating-Collections). To make the process even simpler, we have added support for [SimpleSchema Shorthand Definitions](https://github.com/aldeed/simpl-schema#shorthand-definitions) to `createSchema()`.
The new `defaultCanRead` parameter assigns the given permissions to all fields where `canRead` is `undefined`.

```js
// on the client & server
import { createSchema } from 'meteor/vulcan:core';

const agentSchema = createSchema({
  _id: String,
  title: String,
  description: String,
}, undefined, undefined, ['members']);

export default agentSchema;
```

As an alternative, you can pass a `graphQLType` as a template literal instead of a `schema`.

## Specifying Filterable Fields

```js
// on the server
const filterable = [{
  name: 'view',
  type: 'String',
}];
```

## Registering the Default Fragment

You can reuse the schema to register the default fragment (listing all readable fields) with just a couple of lines of code:

```js
// on the client & server
registerCustomDefaultFragment({
  typeName: 'Agent',
  schema: agentSchema,
  fragmentName: 'AgentsDefaultFragment',
});
```

## Running the Query

On the client you can get the query results as usual with `useMulti2` or `withMulti2`. The only difference is that you pass `typeName` instead of `collection`.
```js
const { results, loading, error, networkStatus, refetch } = useMulti2({ 
  typeName: 'Agent', 
  fragmentName: 'AgentsDefaultFragment',
  input, 
  pollInterval, 
  queryOptions,
});
```

## Aggregation

You can use this same pattern to register an aggregate query. Inside the resolver use `aggregate()` on the collection that you are aggregating.

```js
const resolver = async function (root, { input }, context) {
  const { search } = input;
  const { currentUser } = context;
  
  const pipeline = [
    { // Stage 1
      $match: {
        activatedAt: { $exists: true },
      },
    },
    { // Stage 2
      $lookup: {
        from: 'supplierstubs',
        localField: 'supplierCode',
        foreignField: 'supplierId',
        as: 'supplierstub',
      },
    },
      
    // Ommitted stages 3 to 6 for brevity
    
    { // Stage 7
      $sort: {
        supplierCode: 1,
      },
    },
  
  ];
  
  if (search) {
    pipeline.splice(1, 0, {
      $match: {
        $or: [
          { name: { $regex: search, $options: 'i' } },
          { supplierCode: { $regex: search, $options: 'i' } },
        ],
      },
    });
  }
  
  const docs = await Suppliers.aggregate(pipeline);
  
  return { results: restrictedDocs, totalCount: restrictedDocs.length };
};
```
