---
title: Collections & Schemas
---


## Creating Collections

Vulcan features a number of helpers to make setting up your layer faster, most of which are initialized through the `createCollection` function:

```js
const Movies = createCollection({

  collectionName: 'movies',

  typeName: 'Movie',

  schema,
  
  resolvers,

  mutations,

});
```

The function takes the following arguments:

- `collectionName`: the name of the collection in your MongoDB database.
- `typeName`: the name of the GraphQL type that will be generated for the collection.
- `schema`, `resolvers`, `mutations`: see below.
- `generateGraphQLSchema`: whether to use the objects passed above to automatically generate the GraphQL schema or not (defaults to `true`). 

#### Alternative Approach

Passing a schema, resolvers, and mutations to `createCollection` enables a lot of Vulcan's internal synergy. That being said, you can also set `generateGraphQLSchema` to `false` and use the custom schemas, custom resolvers, and custom mutations utilities documented below to bypass this if you prefer. 

## Schemas

The first piece of any GraphQL API is the **schema**, which defines what data is made available to the client. 

In Vulcan, this GraphQL schema can be generated automatically from your collection's JSON schema, so you don't have to type things twice. Just pass a [SimpleSchema](https://github.com/aldeed/meteor-simple-schema)-compatible JSON schema as `createCollection`'s `schema` property.

### Custom Schemas

If you need to manually add a schema, you can also do so using the `GraphQLSchema.addSchema` function:

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

## Schema Properties

### Default Properties

See the [SimpleSchema documentation](https://github.com/aldeed/meteor-simple-schema#schema-rules).

### Data Layer Properties

#### `viewableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` viewing the document, and should return `true` or `false`. When generating a form for inserting new documents, the form will contain all the fields that return `true` for the current user.

#### `insertableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` performing the operation, and should return `true` or `false`. When generating a form for inserting new documents, the form will contain all the fields that return `true` for the current user.

#### `editableBy`

Can either be an array of group names or a function.

If it's a function, it'll be called on the `user` performing the operation, and the `document` being operated on, and should return `true` or `false`. When generating a form for editing existing documents, the form will contain all the fields that return `true` for the current user.

#### `resolveAs`

Although Vulcan uses your SimpleSchema JSON schema to generate your GraphQL schema, it's important to understand that the two can be different. 

For example, you might have a `userId` property in your `Posts` schema, but want this property to *resolve* as a `user` object in your GraphQL schema (and in turn, in your client store). You can use `resolveAs` to accomplish this:

```
resolveAs: 'user: User'
```

Note that in this case the `User` GraphQL type is generated automatically out of the box since `Users` is created using `createCollection`, but you could also add it manually:

```
const userSchema = `
  type User {
    _id: String
    email: String
    ...
  }
`;

GraphQLSchema.addSchema(userSchema);
```

You'll also need to write the `Post.user` resolver (where `Post` is the GraphQL type of the current collection) and manually add it to your global schema:

```
const postResolvers = {
  Post: {
    user(post, args, context) {
      return context.Users.findOne({ _id: post.userId }, { fields: context.getViewableFields(context.currentUser, context.Users) });
    },
  },
};

GraphQLSchema.addResolvers(postResolvers);
```

Or, maybe you want to resolve a `tagsId` list of IDs in your database as a `tags` array in your GraphQL schema:

```
resolveAs: 'tags: [Tags]'
```

Again, make sure that the `Tags` GraphQL type exists, and than you write a `tags` resolver for the current collection type. 

The JSON schema field doesn't even need to actually exist in the database. Let's suppose that every comment has a `postId` property, but that a post doesn't know anything about its own comments. You could create a new `commentsPlaceholder` field in your `Posts` schema and set:

```
resolveAs: 'comments: [Comments]'
```

`commentsPlaceholder` will never actually contain anything in your database, but it will still resolve to `comments` in your GraphQL schema. 

### Forms Properties

#### `label`

The form label. If not provided, the label will be generated based on the field name and the available language strings data.

#### `control`

Either a text string (one of `text`, `textarea`, `checkbox`, `checkboxgroup`, `radiogroup`, or `select`) or a React component.

#### `order`

A number corresponding to the position of the property's field inside the form.

#### `group`

An optional object containing the group/section/fieldset in which to include the form element. Groups have `name`, `label`, and `order` properties.

For example:

```js
postedAt: {
  type: Date,
  optional: true,
  insertableBy: Users.isAdmin,
  editableBy: Users.isAdmin,
  publish: true,
  control: "datetime",
  group: {
    name: "admin",
    label: "Admin Options",
    order: 2
  }
},
```

Note that fields with no groups are always rendered first in the form.

#### `placeholder`

A placeholder value for the form field.

#### `beforeComponent`

A React component that will be inserted just before the form component itself.

#### `afterComponent`

A React component that will be inserted just after the form component itself.

#### `hidden`

Can either be a boolean or a function accepting form props as argument and returning a boolean.

Remove the field from the form altogether. To populate the field manipulate the value through the context, e.g.:

```js
this.context.updateCurrentValues({foo: 'bar'});
```

As long as a value is in this.state.currentValues it should be submitted with the form, no matter whether there is an actual form item or not.

### Other Properties

#### `required` (`Users` only)

You can mark a field as `required: true` to indicate that it should be completed when the user signs up. If you're using the `nova:base-components` theme, a form will then pop up prompting the user to complete their profile with the missing fields. 

## Custom Fields

Out of the box, Vulcan has three main collections: `Posts`, `Users`, and `Comments`. Each of them has a pre-set schema, but that schema can also be extended with custom fields.

For example, this is how the `nova:newsletter` package extends the `Posts` schema with a `scheduledAt` property that keeps track of when a post was sent out as part of an email newsletter:

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

Each field schema supports all of the [SimpleSchema properties](https://github.com/aldeed/meteor-simple-schema#schema-rules), such as `type`, `optional`, etc.

A few special properties (`viewableBy`, `insertableBy`, `editableBy`, `control`, and `order`) are also supported by the [Forms](forms.html) package.

You can also remove a field by calling `collection.removeField(fieldName)`. For example:

```js
Posts.removeField('scheduledAt');
```