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
* Auto-generate [paginated, searchable datatables](/core-components.html#Datatable).
* Auto-generate [smart cards](/core-components.html#Card) for displaying individual documents.
* Add callbacks on document insert or edit.

## Example

Here is a schema example, taken from the [Movies tutorial](/example-movies.html):

```js
const schema = {
  // default properties

  _id: {
    type: String,
    optional: true,
    canRead: ["guests"]
  },
  createdAt: {
    type: Date,
    optional: true,
    canRead: ["guests"],
    onCreate: ({ newDocument, currentUser }) => {
      return new Date();
    }
  },
  userId: {
    type: String,
    optional: true,
    canRead: ["guests"],
    resolveAs: {
      fieldName: "user",
      type: "User",
      resolver: (movie, args, context) => {
        return context.Users.findOne(
          { _id: movie.userId },
          {
            fields: context.Users.getViewableFields(
              context.currentUser,
              context.Users
            )
          }
        );
      },
      addOriginalField: true
    }
  },

  // custom properties

  name: {
    label: "Name",
    type: String,
    optional: true,
    canRead: ["guests"],
    canCreate: ["members"],
    canUpdate: ["members"]
  },
  year: {
    label: "Year",
    type: String,
    optional: true,
    canRead: ["guests"],
    canCreate: ["members"],
    canUpdate: ["members"]
  },
  review: {
    label: "Review",
    type: String,
    optional: true,
    control: "textarea",
    canRead: ["guests"],
    canCreate: ["members"],
    canUpdate: ["members"]
  }
};
```

As you can see, a schema is a JavaScript object containing a list of fields, each of which is defined using a range of special properties.

## Creating Collections

Vulcan features a number of helpers to make setting up your layer faster, most of which are initialized through the `createCollection` function:

```js
const Movies = createCollection({
  typeName: "Movie",

  schema,

  resolvers,

  mutations
});
```

The function takes the following arguments:

* `typeName`: the name of the GraphQL type that will be generated for the collection.
* `collectionName`: optionally, the name of the collection throughout your app (will be lowercased in your MongoDB database). If not provided it will be the plural of your type name.
* `dbCollectionName`: if you want to use a different name in your database you can specify it here.
* `schema`, `resolvers`, `mutations`: see below.
* `generateGraphQLSchema`: whether to use the objects passed above to automatically generate the GraphQL schema or not (defaults to `true`).

#### Alternative Approach

Passing a schema, a resolver, and a mutation to `createCollection` enables a lot of Vulcan's internal synergy. That being said, you can also set `generateGraphQLSchema` to `false` and use the custom schemas, custom resolvers, and custom mutations utilities documented below to bypass this if you prefer.

## Extending Schemas

Sometimes, you'll want to extend an existing schema. For example Vulcan's [Forum example](/example-forum.html) has three main collections: `Posts`, `Users`, and `Comments`. Each of them has a pre-set schema, but that schema can also be extended with custom fields.

This is how the `vulcan:newsletter` package extends the `Posts` schema with a `scheduledAt` property that keeps track of when a post was sent out as part of an email newsletter:

```js
Posts.addField({
  fieldName: "scheduledAt",
  fieldSchema: {
    type: Date,
    optional: true
  }
});
```

The `collection.addField()` function takes either a field object, or an array of fields. Each field has a `fieldName` property, and a `fieldSchema` property.

Each field schema supports all of the [SimpleSchema properties](https://github.com/aldeed/meteor-simple-schema#schema-rules), such as `type`, `optional`, etc.

A few special properties (`canRead`, `canCreate`, `canUpdate`, `control`, and `order`) are also supported by the [Forms](forms.html) package.

You can also remove a field by calling `collection.removeField(fieldName)`. For example:

```js
Posts.removeField("scheduledAt");
```
