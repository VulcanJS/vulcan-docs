---
title: Schemas & Collections
---

The role of the schema is to serve as a central source of truth that will be used to generate or populate many of Vulcan's other components.

![/images/vulcan-schemas.svg](/images/vulcan-schemas.svg)

Based on your schema, Vulcan can:

* Generate a GraphQL equivalent of your schema to control your GraphQL API.
* [Control permissions](/groups-permissions.html) for accessing and modifying data.
* [Generate forms](/forms.html) on the client.
* Validate form contents on submission.
* Auto-generate [paginated, searchable datatables](/datatable.html).
* Auto-generate [smart cards](/card.html) for displaying individual documents.
* Add callbacks on document insert or edit.

## Schemas

Here is a schema example, taken from the [Movies tutorial](/example-movies.html):

```js
const schema = {
  _id: {
    type: String,
    optional: true,
    canRead: ['guests']
  },
  createdAt: {
    type: Date,
    optional: true,
    canRead: ['guests'],
    onCreate: () => {
      return new Date();
    }
  },
  userId: {
    type: String,
    optional: true,
    canRead: ['guests'],
    resolveAs: {
      fieldName: 'user',
      type: 'User',
      relation: 'hasOne',
    }
  },
  name: {
    type: String,
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members']
  },
  year: {
    type: String,
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members']
  },
  review: {
    type: String,
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members'],
    input: 'textarea',
  }
};
```

As you can see, a schema is a JavaScript object containing a list of fields, each of which is defined using a range of special properties.

## Creating Collections

Vulcan features a number of helpers to make setting up your layer faster, most of which are initialized through the `createCollection` function:

```js
const Movies = createCollection({
  typeName: 'Movie',

  collectionName: 'Movies',

  schema,

  resolvers,

  mutations,

  permissions,

  filters, 

  components,

  generateGraphQLSchema, 
});
```

The function takes the following arguments:

* `typeName`: the name of the GraphQL type that will be generated for the collection.
* `collectionName`: optionally, the name of the collection throughout your app (will be lowercased in your MongoDB database). If not provided it will be the plural of your type name.
* `dbCollectionName`: if you want to use a different name in your database you can specify it here.
* `schema`, `resolvers`, `mutations`: see below.
* `permissions`: an object defining the collection's document-level permissions
* `filters`: an object defining filters to enhance collection queries.
* `components`: an object containing helper components.
* `resolvers` (optional): an object containing `single` and `multi` query resolvers.
* `mutations` (optional): an object containing `create`, `update`, `upsert`, and `delete` mutation resolvers.
* `generateGraphQLSchema`: whether to use the objects passed above to automatically generate the GraphQL schema or not (defaults to `true`).

Note that if you do not specify `resolvers` or `mutations`, default query resolvers and mutation resolvers will be generated for you. 

#### Alternative Approach

Passing a schema, a resolver, and a mutation to `createCollection` enables a lot of Vulcan's internal synergy. That being said, you can also set `generateGraphQLSchema` to `false` and use the custom schemas, custom resolvers, and custom mutations utilities documented below to bypass this if you prefer.

## Extending Collections

You can extend a collection's options with `extendCollection(collection, options)`. For example:

```
extendCollection(Posts, { 
  callbacks: { 
    create: {
      after: [ notifyAdmins ]
    }
  }
});
```

This can be useful when you want to declare a collection in a file shared by both client and server, but want to add some options (such as callbacks) only on the server.

Note that this works for all options except the schema, which is initialized with the initial `createCollection`. To extend the schema, see `addField` below.

## Extending Schemas

Sometimes, you'll want to extend an existing schema. For example Vulcan's [Forum example](/example-forum.html) has three main collections: `Posts`, `Users`, and `Comments`. Each of them has a pre-set schema, but that schema can also be extended with custom fields.

This is how the `vulcan:newsletter` package extends the `Posts` schema with a `scheduledAt` property that keeps track of when a post was sent out as part of an email newsletter:

```js
Posts.addField({
  fieldName: 'scheduledAt',
  fieldSchema: {
    type: Date,
    optional: true
  }
});
```

The `collection.addField()` function takes either a field object, or an array of fields. Each field has a `fieldName` property, and a `fieldSchema` property.

Each field schema supports all of the [SimpleSchema properties](https://github.com/aldeed/simple-schema-js), such as `type`, `optional`, etc.

A few special properties (`canRead`, `canCreate`, `canUpdate`, `input`, and `order`) are also supported by the [Forms](forms.html) package.

You can also remove a field by calling `collection.removeField(fieldName)`. For example:

```js
Posts.removeField("scheduledAt");
```

## Nested Schemas

Your collection's basic schema is flat, but in some cases you might want to validate your data and specify its shape multiple levels deep. The two main cases where this is useful is when: 

1. You are storing your data in a nested shape in your database.
2. You are using custom resolvers to load complex objects and would like to give them GraphQL types. 

In either case, the first step is to define a new schema for your nested data, in this case customer addresses. 

Note that while you can pass a “raw” schema object to a collection directly, in other cases (such as when being used as sub-schema) it's necessary to initialize schemas to transform them into actual schema objects using the `createSchema` function: 

```js
import { createSchema } from 'meteor/vulcan:core';

const addressSchema = createSchema({
  street: {
    type: String,
    optional: false,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members'],
    max: 100, // limit street address to 100 characters
  },
  country: {
    type: String,
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members'],
  },
  zipCode: {
    type: Number,
    optional: true,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members'],
    input: 'number',
  },
});
```

Note that nested schemas also need their own `canRead`, `canCreate`, etc. properties. 

You can then call that schema from your main collection schema:

```js
const schema = {
  _id: {
    type: String,
    optional: true,
    canRead: ['guests'],
  },
  
  //...

  addresses: {
    type: Array,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members'],
  },

  'addresses.$': {
    type: addressSchema,
  },
};
```

The resulting type's name will be `${typeName}${fieldName}`, which in this case means a new `CustomerAddresses` GraphQL type will automatically be created for you. 

Note that currently there is no way to reuse nested GraphQL schemas, meaning two `shippingAddress` and `billingAddress` fields will generate two `CustomerShippingAddress` and `CustomerBillingAddress` GraphQL types even if the underlying JavaScript sub-schemas objects are the same.

The previous example assumes that the `addresses` data is stored in your database, but if you're getting it through a resolved field instead you will have to specify your type manually:

```js
const schema = {
  _id: {
    type: String,
    optional: true,
    canRead: ['guests'],
  },
  
  //...

  addresses: {
    type: Array,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members'],
    resolveAs: {
        type: '[CustomerAddresses]',
        resolver: customer => {
          return getAddresses(customer);
        },
      },
    },
  },

  'addresses.$': {
    type: addressSchema,
  },
};
```

### Disabling Nested GraphQL Schemas

If you'd like to use a nested schema for validation purposes but *don't* want to generate the corresponding GraphQL type, you can set the `blackbox: true` option on the field that has the nested schema:

```js
const customerSchema = {
  _id: {
    type: String,
    canRead: ['guests'],
  },
  name: {
    type: String,
    canRead: ['guests'],
  },
  addresses: {
    type: Array,
    canRead: ['guests'],
    blackbox: true
  },
  'addresses.$': {
    type: addressSchema,
  },
};
```
