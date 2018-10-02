---
title: Groups & Permissions
---

## Debugging

**If you've got the [debug package](/debug.html) enabled, a groups debugging UI is available at [http://0.0.0.0:3000/debug/groups](http://0.0.0.0:3000/debug/groups).**
## Groups & Actions

When it comes to controlling who can do what in your Vulcan app, it's important to understand two concepts, **groups** and **actions**. 

A **group** (such as `admins` or `mods`) represents a list of users which can perform specific actions (such as `post.update` or `post.delete`). 

To test if a user can perform an action, we don't check if they belong to a specific group (e.g. `user.isAdmin === true`), but instead if *at least one of the groups they belong to* has the rights to perform the current action.  

To understand the benefits of this two-tiered approach, let's consider a common use case: creating a new “mods” group whose members can edit other user's posts. 

If we had adopted the “obvious” solution of testing if a user belongs to a set of groups before letting them perform an action, this would mean adding the “mods” group to the list everywhere where this action is performed. 

If, instead, we test if the user has permissions to perform the *action* `post.update.all`, we can now simply add this action to the “mods” group in a single place. 

## Documents & Fields

Vulcan has two levels of permission checks: the document level, and the field level. 

Consider a scenario where a user can edit their own posts, but an admin can edit anybody's post. Now let's add the requirement that a user can only edit a post's `title` property, but an admin can also edit a post's `status`. 

First, we'll need a **document-level** check to see if the current user can edit a given document. In Vulcan, this check lives, appropriately enough, next to the relevant mutation:

```js
updatePost: {
  
  name: 'updatePost',
  
  check(user, document) {
    if (!user || !document) return false;
    return Users.owns(user, document) ? Users.canDo(user, 'post.update.own') : Users.canDo(user, `post.update.all`);
  },

  mutation(root, {documentId, set, unset}, context) {

    const document = context.Posts.findOne(documentId);
    performCheck(this, context.currentUser, document);

    // perform operation
  },

},
```

Here, we're first testing if the current user owns the document (meaning their `_id` is equal to the document's `userId`). If they do, we test if they can perform the `post.update.own` action. If they can't we test for the `post.update.all` action. 

Now comes the second check: is the user trying to modify fields they don't have access to? This check lives at the field level, in the schema:

```js
title: {
  type: String,
  canRead: ['guests'],
  canCreate: ['members'],
  canUpdate: ['members'],
},
status: {
  type: Number,
  canRead: ['guests'],
  canCreate: ['admins'],
  canUpdate: ['admins'],
},
```

The `canUpdate` property takes an array of the names of the groups that can edit a given field. This is one of the few places where we test for groups and not actions, because defining new actions for each field (`post.update.title`, `post.update.status`, etc.) would be a bit too time consuming. 

Optionally, for more fine-grained permissions `canRead`, `canCreate`, and `canUpdate` can also take a function that returns a boolean as argument. That function takes the current user as first argument (and, for `canRead` and `canUpdate`, the current document as second argument).

Note that we've talked about editing, but the same principles apply when it comes to inserting, or removing documents (although there is no field-level check for the `remove` operation).

## Ownership

Notice the use of `User.owns` in the previous example and in the default mutations provided by Vulcan. `User.owns` relies on the `userId` field of the object to define ownership. It means that in order for the default check to work, **you'll need to specify an `userId` field in your schema.**

```js
userId: {
  type: String,
  hidden: true,
  optional: true
}
```
By default, Vulcan will set this `userId` to `currentUser._id` before it calls the `onCreate` callbacks. However, you still need to define this `userId` field in your schema in order to activate this behaviour.

Of course you can also set the `userId` to another value as you would do for any other field, for example if you are an admin that need to assign documents to other users.

## Controlling Viewing

Unlike mutations which usually only affect a single document, viewing usually concerns a set of documents.

Again, let's make a distinction between field-level and document-level control. If a client requests a list of documents, you need to check two things:

1. Which of these documents the client is allowed to access.
2. Of the allowed documents, which fields they're allowed to view. 

Note that while GraphQL lets the client *ask* for the fields they want, it's up to you to decide whether or not to accept that request on a per-field basis. 

Practically speaking, controlling access in this manner usually requires four steps:

1. Define a `checkDocument` function that returns whether a user can access a document.
2. Use that function to iterate over an array and filter out unaccessible documents.
3. Define a `checkField` function that checks whethere a user can access a *field* of a document. 
4. Use it to iterate over each document returned at step 2.

Let's see how the `vulcan:posts` package implements each step. First, a `Posts.checkAccess` function is defined in `collection.js`:

```js
const Posts.checkAccess = (currentUser, post) => {
  if (Users.isAdmin(currentUser) || Users.owns(currentUser, post)) {
    return true;
  } else if (post.isFuture) {
    return false;
  } else { 
    const status = _.findWhere(Posts.statuses, {value: post.status});
    return Users.canDo(currentUser, `posts.view.${status.label}`);
  }
}
```

If a user is an admin, or if they're the owner of the document in question, we always give them access. In the opposite case, if the document is scheduled to appear in the future, we always *deny* access. Finally, for non-future documents, we check the post's status and whether the user can perform the relevant action (`post.read.pending`, `post.read.approved`, etc.) as defined in `permissions.js`. 

Of course, your own `check` function can also be much simpler depending on your needs.

We can then use this function to filter out any unviewable posts in the `resolvers.js` file, using Underscore's `filter`:

```js
const viewablePosts = _.filter(posts, post => Posts.checkAccess(currentUser, post));
```

Step 3 and 4 are taken care of together through the `Users.restrictViewableFields` utility which take the current user, the collection, and either a document or an array of documents:

```js
const restrictedPosts = Users.restrictViewableFields(currentUser, Posts, viewablePosts);
```

Behind the scenes, this uses each field's `canRead` property to either check if the current user is member of the right group (if `canRead` is a group string) or passes the viewable check (if `canRead` is a function). 

## Back-End vs Front-End

It's important to remember that you can never really control what the client can do: just because you hide a form on the client doesn't mean a smart user can't figure out a way to trigger its operation; unless of course you disallow said operation on the server. 

That being said, it's still good practice to match your front-end's permissions later to your back-end's, so users don't run into errors. In practice, this means reusing the same checks you use on the server in your React components.

For example, before showing a user an “edit post” link, you can call the edit mutation's `check` function to make sure they're allowed to perform the operation:

```jsx
<div className="posts-item-meta">
  {Posts.options.mutations.update.check(this.props.currentUser, post) ? this.renderEditLink() : null}
</div>
```

When displaying a list of items, you can rely on [terms and parameters](http://docs.vulcanjs.org/terms-parameters.html) to require only the data the current user is authorized to see from the database. This is actually mandatory if you want Mongo to count the total number of documents that are really available to the user, which is needed for pagination and "load more" buttons.

This example view tells the database to look only for documents that are either owned by the user or already published. **However you still need to define a `checkAccess` method**, since a malicious user could decide to bypass the view and ask all the data from the database.

```js
// filter the data fetched from the database
const myRestrictedView = terms => ({
  selector: {
    $or: [
      {userId: terms.userId},
      {isFuture: false}
    ]
  }
})
Posts.addView("myRestrictedView", myRestrictedView)
```

Example usage with Datatable:

```jsx
// /!\ this is only a frontend restriction and does not actually secure the database
<DataTable
  options={{ terms: { view: "myRestictedView" }}}
  collection={Posts}
/>
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
Users.groups.mods.can("post.update.all"); // mods can edit anybody's posts
Users.groups.mods.can("post.delete.all"); // mods can delete anybody's posts
```

You can also define your own custom actions:

```js
Users.groups.mods.can("invite"); // new custom action
```
