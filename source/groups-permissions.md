---
title: Groups & Permissions
---

## Groups & Actions

When it comes to controlling who can do what in your Vulcan app, it's important to understand two concepts, **groups** and **actions**. 

A **group** (such as `admins` or `mods`) represents a list of users which can perform specific actions (such as `posts.edit` or `posts.remove`). 

To test if a user can perform an action, we don't check if they belong to a specific group (e.g. `user.isAdmin === true`), but instead if *at least one of the groups they belong to* has the rights to perform the current action.  

To understand the benefits of this two-tiered approach, let's consider a common use case: creating a new “mods” group whose members can edit other user's posts. 

If we had adopted the “obvious” solution of testing if a user belongs to a set of groups before letting them perform an action, this would mean adding the “mods” group to the list everywhere where this action is performed. 

If, instead, we test if the user has permissions to perform the *action* `posts.edit.all`, we can now simply add this action to the “mods” group in a single place. 

## Documents & Fields

Vulcan has two levels of permission checks: the document level, and the field level. 

Consider a scenario where a user can edit their own posts, but an admin can edit anybody's post. Now let's add the requirement that a user can only edit a post's `title` property, but an admin can also edit a post's `status`. 

First, we'll need a **document-level** check to see if the current user can edit a given document. In Vulcan, this check lives, appropriately enough, next to the relevant mutation:

```js
edit: {
  
  name: 'postsEdit',
  
  check(user, document) {
    if (!user || !document) return false;
    return Users.owns(user, document) ? Users.canDo(user, 'posts.edit.own') : Users.canDo(user, `posts.edit.all`);
  },

  mutation(root, {documentId, set, unset}, context) {

    const document = context.Posts.findOne(documentId);
    performCheck(this, context.currentUser, document);

    // perform operation
  },

},
```

Here, we're first testing if the current user owns the document (meaning their `_id` is equal to the document's `userId`). If they do, we test if they can perform the `posts.edit.own` action. If they can't we test for the `posts.edit.all` action. 

Now comes the second check: is the user trying to modify fields they don't have access to? This check lives at the field level, in the schema:

```js
title: {
  type: String,
  viewableBy: ['guests'],
  insertableBy: ['members'],
  editableBy: ['members'],
},
status: {
  type: Number,
  viewableBy: ['guests'],
  insertableBy: ['admins'],
  editableBy: ['admins'],
},
```

The `editableBy` property takes an array of the names of the groups that can edit a given field. This is one of the few places where we test for groups and not actions, because defining new actions for each field (`posts.edit.title`, `posts.edit.status`, etc.) would be a bit too time consuming. 

Optionally, for more fine-grained permissions `viewableBy`, `insertableBy`, and `editableBy` can also take a function that returns a boolean as argument. That function takes the current user as first argument (and, for `viewableBy` and `editableBy`, the current document as second argument).

Note that we've talked about editing, but the same principles apply when it comes to inserting, or removing documents (although there is no field-level check for the `remove` operation).

## Controlling Viewing

Unlike mutations which only affect a single document, viewing usually concerns a set of documents.

Again, let's make a distinction between field-level and document-level control. Field-level control only (e.g. “only admins can see the `clickCount` field”) is fairly easy to implement thanks to the `getViewableFields()` utility, available on a resolver's `context`. It takes the current user and the current collection, and returns a list of the fields that the user is allowed to access:

```js
list: {

  name: 'moviesList',

  resolver(root, {terms, offset, limit}, context, info) {
    const options = {
      sort: {createdAt: -1},
      limit: (limit < 1 || limit > 10) ? 10 : limit,
      skip: offset,
      // keep only fields that should be viewable by current user
      fields: context.getViewableFields(context.currentUser, context.Movies),
    };
    return context.Movies.find({}, options).fetch();
  },

},
```

`getViewableFields` can also take a third `document` argument for specific document-level control (e.g. “the `privateComments` property of a document is only viewable by its owner”), but this gets trickier as it requires fetching the document first before filtering its fields. 

First, here's how you would define a private field's `viewableBy` property to restrict its visibility to document owners, making use of the fact that `viewableBy` can also take a function called on the user and the document:

```js
privateComments: {
  type: String,
  viewableBy: Users.owns,
  insertableBy: ['members'],
  editableBy: ['members']
},
```

And here's how to restrict the `single` resolver based on the current document:

```js
single: {
  
  name: 'moviesSingle',

  resolver(root, {documentId}, context) {
    const document = context.Movies.findOne({_id: documentId});
    return _.pick(document, _.keys(context.getViewableFields(context.currentUser, context.Movies, document)));
  },

},
```

Here's also a more complex example when filtering the `list` resolver (although in most cases, it will probably acceptable to only show private properties in the `single` view):

```js
list: {

  name: 'moviesList',

  resolver(root, {terms, offset, limit}, context, info) {
    const options = {
      sort: {createdAt: -1},
      limit: (limit < 1 || limit > 10) ? 10 : limit,
      skip: offset,
    };

    const movies = context.Movies.find({}, options).fetch();
    return movies.map(movie => {
      const editableFields = _.keys(context.getViewableFields(context.currentUser, context.Movies, movie));
      return _.pick(movie, editableFields);
    });
  },

},
```

## Back-End vs Front-End

It's important to remember that you can never really control what the client can do: just because you hide a form on the client doesn't mean a smart user can't figure out a way to trigger its operation; unless of course you disallow said operation on the server. 

That being said, it's still good practice to match your front-end's permissions later to your back-end's, so users don't run into errors. In practice, this means reusing the same checks you use on the server in your React components.

For example, before showing a user an “edit post” link, you can call the edit mutation's `check` function to make sure they're allowed to perform the operation:

```jsx
<div className="posts-item-meta">
  {Posts.options.mutations.edit.check(this.props.currentUser, post) ? this.renderEditLink() : null}
</div>
```

## Permissions API

Here's how to create and modify groups.

```js
Users.createGroup(groupName); // create a new group

Users.getGroups(user); // get a list of all the groups a user belongs to

Users.getActions(user); // get a list of all the actions a user can perform

Users.canDo(user, action); // check if a user can perform a specific action
```

Documents can be Posts, Comments, or Users. 

Note that some groups are applied automatically:

- `guests`: any non-logged-in user is considered guest. This group is special in that guests users are by definition not part of any other group.
- `members`: default group for all existing users. Is applied to every user in addition to any other groups. 
- `admins`: any user with the `isAdmin` flag set to true.

## Adding User to Groups

You can add any group string to a user's `groups` array:

```js
Users.update(user._id, {$push: {groups: 'mods'}});

// or

user.group = [...user.group, 'mods'];
```

## Assigning Actions

```js
// assuming we've created a new "mods" group
Users.groups.mods.can("posts.edit.all"); // mods can edit anybody's posts
Users.groups.mods.can("posts.remove.all"); // mods can delete anybody's posts
```

You can also define your own custom actions:

```js
Users.groups.mods.can("invite"); // new custom action
```

Here's a list of all out-of-the-box permissions:

```js
// guests actions
posts.view.approved.own
posts.view.approved.all
comments.view.own
comments.view.all
categories.view.all

// members actions
posts.view.approved.own
posts.view.approved.all
posts.view.pending.own
posts.view.rejected.own
posts.view.spam.own
posts.view.deleted.own
posts.new
posts.edit.own
posts.remove.own
posts.upvote
posts.cancelUpvote
posts.downvote
posts.cancelDownvote
comments.view.own
comments.view.all
comments.new
comments.edit.own
comments.remove.own
comments.upvote
comments.cancelUpvote
comments.downvote
comments.cancelDownvote
users.edit.own
users.remove.own
categories.view.all

// admin actions
posts.view.pending.all
posts.view.rejected.all
posts.view.spam.all
posts.view.deleted.all
posts.new.approved
posts.edit.all
posts.remove.all
comments.edit.all
comments.remove.all
users.edit.all
users.remove.all
categories.view.all
categories.new
categories.edit.all
categories.remove.all
```
