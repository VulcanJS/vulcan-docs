---
title: Security
---

Security is an important concern for any web app, and Vulcan offers a couple strategies you can pick and choose from. 

Note that this page contains a general overview of the different aspects of security in Vulcan, but if you want more details about implementation be sure check out the [Groups & Permissions](/groups-permissions.html) section.

## Manual Approach

At its core, all a GraphQL endpoint does is receive queries and dispatch them to the appropriate resolvers or mutations. What these resolvers do is entirely up to you. 

In other words, the GraphQL endpoint itself has no direct access to your data unless you yourself write the lines of code that provide on that data.

Of course, part of the appeal of Vulcan is that it lets you avoid writing boilerplate resolvers and mutations with smart defaults. So let's see how these defaults handle security. 

## Reading Data

If you're making use of Vulcan's [default resolvers](/resolvers.html#Default-Resolvers), you have a few security mechanisms available to you, depending on whether you're trying to control access at the **document level** (”only these users can access these documents”) or at the **field level** (“only these users can access these fields on these documents”).

### Document-Level Access

First, you'll want to control which document any given user can access. For example, a common practice for sensitive data is to only let users access their own documents. 

You can do this using the [collection's global `checkAccess` function](/groups-permissions.html#Controlling-Viewing), which is called by the default `list` and `single` resolvers every time a document is requested.

Note that if you are *not* using the default resolvers, specifying `checkAccess` won't do anything unless you manually call it yourself.

### Field-Level Access

Another way of looking at security is field by field. For example, maybe you want to limit the visibility of a hidden `score` field to admins to make it harder to game your voting system. 

You can do this through [the `canRead` property](/schema-properties.html#Data-Layer-Properties) as defined on a field's schema. 

`canRead` either takes a function that takes in the current user and current document and return `true` or `false`; or an array of strings corresponding to the names of the [user groups](/groups-permissions.html#Groups-amp-Actions) who can view the document. 

#### Visibility vs Accessibility

Note that in order to be included in the GraphQL schema at all, a field needs either an `canRead`, `canCreate`, or `canUpdate` property. If none of these properties exist, the field will be left out of the schema and will not be accessible through the GraphQL endpoint at all, no matter what. 

If one of these properties *does* exist, the field will be added to the schema, but that doesn't necessarily mean querying it will actually return a value; that all depends on the contents of `canRead`.

## Writing Data

Even more important than controlling who can *read* data is controlling who can *write* it. And Vulcan's [default mutations](/mutations.html#Default-Mutations) also have a few tricks up their sleeves. 

Again, we need to make a distinction between **document-level checks** (“only these users can edit this document”) and **field-level checks** (“only these users can edit these fields of this document”).

### Document-Level Checks

The first step in securing your mutations is deciding who can perform them, and on which documents. Vulcan's default mutations offer two different approaches.

#### Group-Based Checks

Out of the box, the three default mutations will first check if a current user exists. If it does, they will then check if that user can perform the `collectionName.operationName` action. Depending on the current action, this action could be named (using the `Posts` collection as an example):

- `post.create`
- `post.update.own`
- `post.update.all`
- `post.delete.own`
- `post.delete.all`

So an easy way to enable a user to perform a given mutation is to [enable the corresponding action](/groups-permissions.html#Assigning-Actions) for a group they belong to. 

Note that for `update` and `delete` operations, two distinct actions are checked (`*.all` and `*.own`) based on whether the current user owns the current document or not. This makes it easy to let one group (for example, `mods`) edit all documents in a collection while another one can only edit their own. 

When writing your own back-end logic, you're welcome to also add your own new action names (`post.update.drafts`, `post.delete.archived`, etc.) if that helps set up your permissions structure. 

#### Manual Checks

Sometimes you need finer-grained control over mutations, or maybe you're just not using groups at all. In that case, when calling `getDefaultMutations(collectionName)` inside `createCollection` you can also pass a second `options` object to define `check` functions for the three default mutations:

```js
const mutations = getDefaultMutations('Posts', {
  newCheck: (user, document) => { /*...*/ },
  editCheck: (user, document) => { /*...*/ },
  removeCheck: (user, document) => { /*...*/ }
});
```

This is useful when a mutation check has to depend on a specific document. For example, you can imagine a scenario where documents can only be edited if their `status` is set to `draft`. In this case, simply checking for groups wouldn't be dynamic enough, and you'll need to pass an `editCheck` function instead. 

Note that if you specify manual checks for a mutation, the aforementioned group checks will be bypassed.

### Field-Level Checks

Just like you can control field-level access with `canRead`, you can control field-level data writes with `canCreate` and `canUpdate`. 

For example, maybe you only want admin users to be able to reassign a post to a different user. In which case you could make a post's `userId` only be editable by users belonging to the `admins` group.

And just like `canRead`, both `canCreate` and `canUpdate` can also take functions in addition to an array of group names. 

## Other Resolvers & Mutations

If you work with other resolvers and mutations beyond the defaults (for example, maybe you need to create a dedicated mutation for voting on a post), security will be left up to you to implement. That being said, you can still piggy-back on top of the existing Vulcan security infrastructure and reuse some of the same helpers and functions. 

