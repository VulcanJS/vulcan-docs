---
title: Architecture
---

## Vulcan's Building Blocks

Vulcan is built on top of three main open-source technologies: React, GraphQL, and Meteor. 

- [React](https://facebook.github.io/react/) is Vulcan's front-end library.
- [GraphQL](http://graphql.org) is Vulcan's API layer, used to load data from the server with [Apollo](http://apollostack.com). 
- [Meteor](http://meteor.com) is Vulcan's back-end layer, used to handle the database as well as server and bundle the app. 

Each layer is independent from the other two: you can take your React components and use them with a completely different stack, and – although we're not quite there yet – ultimately we also hope to make it easy to migrate out of Meteor.

The goal is to avoid getting stuck: if at any point you outgrow Vulcan, it should be possible to leave it with minimal refactoring. 

## A Note About Apollo

[Apollo](http://apollostack.com) is the set of tools used to manage the GraphQL layer. It consists of two parts, Apollo server and Apollo client. 

- **Apollo server** is used to create the GraphQL API endpoint on the server.
- **Apollo client** (together with the `react-apollo` package) is used to query that endpoint and load data on the client. 

Apollo is still young, but it's well-documented, evolving quickly, and supported by a great team. 

## Code Architecture

In keeping with this idea of flexibility and modularity, Vulcan's application structure is a bit different than most other Meteor apps. 

In Vulcan, each feature has its own distinct **Meteor package** which can be freely added or removed. For example, if your current theme doesn't use comments, you can get rid of every comments-related back-end feature with:

```
meteor remove nova:comments
```

Generally speaking, Vulcan packages fall in one of five categories, and they're all listed in your `.meteor/packages` file.

### Core Packages

These are the core packages that make the [Vulcan framework](tutorial-framework.html):

- `nova:lib`
- `nova:core`
- `nova:forms`
- `nova:routing`
- `nova:email`
- `nova:users` 

Every Vulcan app will usually include them, but note that strictly speaking only `nova:core` (which itself depends on `nova:lib`) cannot be removed from Vulcan apps. 

### Feature Packages

These includes all the main Vulcan features:

- `nova:events`
- `nova:settings`
- `nova:posts`
- `nova:newsletter`
- `nova:search`
- `nova:notifications`
- `nova:getting-started`
- `nova:categories`
- `nova:voting`
- `nova:embedly`
- `nova:api`
- `nova:rss`
- `nova:subscribe`

These can all be added and removed, but note that they might have dependencies on each other (for example `nova:comments` depends on `nova:posts`).

Any extra plug-in you install would also fall in this category.

### Theme Packages

These packages are the one that control what your app actually looks and feels like, and they're the ones you'll usually want to replace (or more likely, extend).

- `nova:base-components`
- `nova:base-styles`
- `nova:email-templates`
- `nova:i18n-en-us`

Theme packages can include React components, CSS styles, email templates, or just language strings. 

### Meteor Packages

Sometimes you'll also want to include either official or third-party Meteor packages to add extra features to your app. While this is usually done from within a package's `package.js` file, you can also add packages directly in your `packages` file in some instances, such as for controlling user accounts:

- `accounts-password`
- `accounts-twitter`

### Custom Packages

Finally, any new package you create to extend Vulcan will fall in the “custom package” category. 

## Extend, Don't Edit

Another key point of Vulcan's philosophy is to never edit the core codebase (i.e. what you get from the Vulcan repo) directly if you can avoid it. 

Editing Vulcan's code makes it harder to keep it up to date when things change in the main Vulcan repo, as your modifications might conflict with a new updated version. 

Instead, Vulcan includes many hooks and patterns that enable you to tweak and extend core features from your *own* packages without having to actually modify their code. 

## Register & Execute

Many Vulcan objects (such as fragments, routes, components, etc.) follow a “register and execute” pattern, in which:

1. All items are first registered in a centralized array.
2. The items are then all executed at runtime. 

This two-tiered approach has two benefits. First, it means items can be overriden in between registration and execution. For example if a theme registers a component, your custom code can then extend it and the final component object that is created at runtime will be the extended version.

Second, things like React Router or the app's GraphQL schema can only be initialized once. By having you register items first, Vulcan is able to centralize objects from various sources, and then combine them all in a single initialization call. 

## The Vulcan Core Package

Unless mentioned otherwise, all Vulcan utilities function are imported from the `nova:core` Meteor package:

```js
import { createCollection } from 'meteor/nova:core';

const Posts = createCollection({...});
```

Note that a lot of these utilities actually live in the `nova:lib` package, but they're re-exported from `nova:core` for convenience's sake. 
