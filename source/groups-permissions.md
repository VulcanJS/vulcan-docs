---
title: Groups & Permissions
---

User groups let you give your users permission to perform specific actions.

To test if a user can perform an action, we don't check if they belong to a specific group (e.g. `user.isAdmin === true`), but instead if *at least one of the groups they belong to* has the rights to perform the current action.  

### Permissions API

```js

Users.createGroup(groupName); // create a new group

Users.methods.addGroup(userId, groupName); // add a user to a group (server only)

Users.getGroups(user); // get a list of all the groups a user belongs to

Users.getActions(user); // get a list of all the actions a user can perform

Users.canDo(user, action); // check if a user can perform a specific action

Users.canView(user, document); // shortcut to check if a user can view a specific document

Users.canEdit(user, document); // shortcut to check if a user can edit a specific document
```

Documents can be Posts, Comments, or Users. 

Note that some groups are applied automatically without having to call `addToGroup`:

- `anonymous`: any non-logged-in user is considered anonymous. This group is special in that anonymous users are by definition not part of any other group.
- `default`: default group for all existing users. Is applied to every user in addition to any other groups. 
- `admins`: any user with the `isAdmin` flag set to true.

### Assigning Actions

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
// anonymous actions
posts.view.approved.own
posts.view.approved.all
comments.view.own
comments.view.all
categories.view.all

// default actions
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

The `*.*.all` actions are generally used as a proxy to check for permission when editing restricted properties. For example, to check if a user can edit a post's `status`, a check is made for the user's ability to perform the `posts.edit.all` action (as there is no dedicated `posts.edit.status` action).
