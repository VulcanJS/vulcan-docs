---
title: Migration Guide
---

As of December 2016, there are three major versions of Telescope:

- The “Legacy” version, which uses Meteor's publications and subscriptions and its Blaze front-end framework.
- The “Vulcan Classic” version, which keeps Meteor's publications and subscriptions but uses React on the front-end. 
- The “Vulcan Apollo” version, which replaces Meteor's publications and subscriptions with Apollo & GraphQL.  

This guide focuses on migrating from Vulcan Classic to Vulcan Apollo. If you're still on the Legacy version, I'm afraid there isn't really an easy migration path and you're probably best off starting a new codebase from scratch. 

## Git Merging

The first step will be merging in the latest commits from the `apollo` branch. I personally use [SourceTree](https://www.sourcetreeapp.com/) to help manage Git repos, but any good Git app (or even the command line) should work just fine. 

I would suggest creating a new local `apollo` branch (so you can easily go back to your last working version if things don't work out) first with:

```
git checkout -B devel-apollo
```

And then pulling in the changes:

```
git pull origin devel
```

This will probably result in at least a few Git conflicts between the old and new versions of the same files. Here are some of the most commonly affected files:

- `package.json`: fix any conflicts by keeping the most recent version of each package. You can always use a tool like [depcheck](https://www.npmjs.com/package/depcheck) later to remove unused packages.
- `packages`: for this file, it's probably easiest to just accept the [new version](https://github.com/VulcanJS/Vulcan/blob/apollo/.meteor/packages) and then manually re-add your own custom packages afterwards. 
- `release`: just keep the latest Meteor version.
- `versions`: this file will be re-generated at runtime, so you can safely pick either version or even delete it for now. 

For any other file (except those belonging to your own custom packages) you should just use the new version. 

Make sure your `packages` file now contains the following new package:

- `vulcan:routing`

Once you've fixed all Git conflicts, it's time to install any additional NPM packages with `npm install`. You can now try running your app with the usual `meteor` command. If you haven't customized anything, then everything should work more or less fine, and you can stop here!

If however you've built your own custom themes and features, you'll probably have to put in a bit of work. First, start by disabling your customizations and making sure your app runs fine. Once this is done, re-enable your custom packages and read on!

## The Telescope Namespace

The global Telescope namespace (`Telescope.*`) has been deprecated in favor of importing individual functions and objects from `vulcan:core`. For example, instead of using `Telescope.routes.add`, you would import `{ addRoutes }` from `meteor/vulcan:core`. 

## Routes

Routes are handled a bit differently in Vulcan Apollo. Instead of having routes definitions (`addRoute(...)`) and the actual routing code in the same package, route definitions now belong together with themes, while routing code lives in `vulcan:routing`. 

This means if you had any routing code (e.g. `ReactRouterSSR.Run()`) in your custom packages, you can now safely remove it and let `vulcan:routing` handle it. 

Other changes:

- `Telescope.routes.routes` -> `import { Routes } from 'meteor/vulcan:core'`
- `Telescope.routes.add` -> `import { addRoute } from 'meteor/vulcan:core'`

## Custom Fields

A few schema properties have been changed, and these will trigger schema errors until you update them:

- `canCreate` is now `canCreate`, and takes either the name of a group or a function. 
- `canUpdate` is now `canUpdate`, and takes either the name of a group or a function. 
- `publish` is replaced by `canRead`, which works similarly to `canCreate` and `canUpdate`. 
- `join` has been replaced by `resolveAs`, which you can use to specify a GraphQL resolver for the field.

In addition, there are also a few new properties:

- `preload`, which you can set to `true` to preload data (only for the `Users` schema).
## List & Single Containers

The `utilities:smart-publications` package has been deprecated, so you can also delete any `PublicationUtils.addToFields()` calls.

## Data Loading

With the move to Apollo, the `utilities:react-list-container` package is not used anymore, and `ListContainer` and `DocumentContainer` are instead replaced with `withMulti` and `withSingle`. 

The key difference is that `withMulti` is now called directly around the component that needs the data (e.g. `export default withMulti(options)(MyComponent)`) unlike `ListContainer`, which was called from that component's *parent* component. 

In practice, this means replacing something like this:

```
<ListContainer 
  collection={Posts} 
  publication="posts.list"
  selector={selector}
  options={options}
  terms={params} 
  joins={Posts.getJoins()}
  component={MyPostsList}
  cacheSubscription={false}
  limit={20}
  increment={20}
  listId={params.listId}
/>
```

With:

```
<PostsList terms={terms} />
```

And then wrapping `MyPostsList` with `withMulti(options)` inside `MyPostsList.jsx`.

Also keep in mind that the data format provided by the GraphQL API can be slightly different from was was previously available (for example, categories are now available on `posts.categories` instead of `posts.categoriesArray`). Make sure to check your component's `props` in the React inspector if you run into any issues. 

## Daily View

The daily view now has a slightly different component architecture in the `base-components` package. 

- `PostsDaily` sets the `terms` for the view and calls `PostsDailyList`.
- `PostsDailyList` is wrapped with the `withMulti` data loading HoC and then iterates over each day, calling `PostsDay`.
- `PostsDay` displays the posts for that day.

Note that data loading happens at the **list** level, not at the level of an individual day. In other words, if you're displaying five days, **all** posts for all five days will be loaded at the same time. 

## Components

Component replacement now uses a slightly different syntax. Instead of:

```js
Telescope.components.UsersProfile = MyCustomUsersProfile;
```

You now write:

```js
import { Components, replaceComponent } from 'meteor/vulcan:core';

replaceComponent('UsersProfile', MyCustomUsersProfile);
```

And to access a component, you now do:

```js
import { Components, replaceComponent } from 'meteor/vulcan:core';

<Components.UsersProfile/>
```

Finally, to extend a component, instead of

```
class MyPostsItem extends Telescope.components.PostsItem {
  ...
}
```

You now write:

```
import Telescope, { getRawComponent } from 'meteor/vulcan:core';
class MyPostsItem extends getRawComponent('PostsItem') {
  ...
}
```

## Context

Context is not used to access the current user anymore. Instead, use the `withCurrentUser` HoC. 

## Methods & Mutations

Meteor methods are now replaced with Apollo mutations. Check out the [mutations](mutations.html) section for more info, and make sure to check out the `withMutation` HoC to take care of simple, one-off mutations. 

## Forms

The `VulcanForms` component now takes care of its own data loading, and accepts a different set of properties. 

Also note that the `CanDo` component used to test for permissions has been replace with `ShowIf`. 

## Other Changes

### Callbacks

- `Telescope.callbacks` -> `import { Callbacks } from 'meteor/vulcan:core'`
- `Telescope.callbacks.add` -> `import { addCallback } from 'meteor/vulcan:core'`
- `Telescope.callbacks.remove` -> `import { removeCallback } from 'meteor/vulcan:core'`
- `Telescope.callbacks.run` -> `import { runCallbacks } from 'meteor/vulcan:core'`
- `Telescope.callbacks.runAsync` -> `import { runCallbacksAsync } from 'meteor/vulcan:core'`

### Strings

- `Telescope.strings.add` -> `import { addString } from 'meteor/vulcan:core'`;

### Settings

- `Telescope.settings.get` -> `import { getSetting } from 'meteor/vulcan:core'`;

Note that using the `Settings` collection and the `vulcan:settings` package are deprecated. 

### Utils

- `Telescope.utils` -> `import { Utils } from 'meteor/vulcan:core'`

## Database Migration

Once you're done with every other change, please run [this migration](https://gist.github.com/SachaG/99389a6cfbfa94417a39345be256a338) which will adapt user profiles by flattening out `user.telescope.foo` to `user.__foo`. 

To run it, create a `/server` folder in the root of your app (same level as `packages`) and copy the code in a `migration.js` file inside it. 
