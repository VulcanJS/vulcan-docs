---
title: Features – [REVIEW]
---

Core Vulcan features include:

## GraphQL Schema Generation

Vulcan will automatically generate GraphQL schemas for your collections based on their [SimpleSchema](https://github.com/aldeed/meteor-simple-schema) JSON schema.

This prevents you from having to specify your schema twice in two different formats. Although please note that this feature is completely optional, and that you can also specify your schema manually if you prefer.

## Automated Forms

Vulcan will also use your schema to generate client-side forms and handle their submission via the appropriate Apollo mutation.

![https://d3vv6lp55qjaqc.cloudfront.net/items/3B0F0h3U1a0i0o372x24/Screen%20Shot%202017-12-21%20at%2015.44.39.png?X-CloudApp-Visitor-Id=f25ee64a8f9b32be400086060540ffac&v=8fc5164f](https://d3vv6lp55qjaqc.cloudfront.net/items/3B0F0h3U1a0i0o372x24/Screen%20Shot%202017-12-21%20at%2015.44.39.png?X-CloudApp-Visitor-Id=f25ee64a8f9b32be400086060540ffac&v=8fc5164f)

For example, here's how you would display a form to edit a single movie:

```js
<Components.SmartForm
  collection={Movies}
  documentId={props.documentId}
/>
```

Note that SmartForm will also take care of loading the document to edit, if it's not already loaded in the client store.

## Easy Data Loading

Vulcan features a set of data loading helpers to make loading Apollo data easier, `withMulti` (to load a list of documents) and `withSingle` (to load a single document).

For example, here's how you would use `withMulti` to pass a `results` prop containing all movies to your `movies` component:

```js
const listOptions = {
  collection: Movies,
  fragment: fragment
};

export default withMulti(listOptions)(movies);
```

You can pass a fragment to control what data is loaded for each document.

## Schema-based Security & Validation

All of Vulcan's security and validation is based on your collection's schema. For each field of your schema, you can define the following functions:

* `canRead`
* `canCreate`
* `canUpdate`

They all take the current user as argument (and optionally the document being affected) and check if the user can perform the given action.

## Groups & Permissions

Vulcan features a simple system to handle user groups and permissions. For example, here's how you would define that all users can create new movies and edit/remove their own, but only admins can edit or remove other user's movies:

```js
const defaultActions = ["movies.new", "movies.edit.own", "movies.remove.own"];
Users.groups.default.can(defaultActions);

const adminActions = ["movies.edit.all", "movies.remove.all"];
Users.groups.admins.can(adminActions);
```

You can then reference these actions in your mutation checks.

## Other Features

Out of the box, Vulcan also includes many other time-saving features, such as:

* Internationalization
* Server-side rendering
* Utilities for theming and extending components
* Email and email templates handling
* Preset boilerplate mutations

And of course, using Vulcan also means you get access to all the non-core Vulcan packages, such as `vulcan:posts`, `vulcan:comments`, `vulcan:newsletter`, `vulcan:search` etc.

## Technology

Vulcan is built on top of four main open-source technologies: React, GraphQL, Apollo, and Meteor.

### React

[React](https://facebook.github.io/react/) is Vulcan's front-end library.

### GraphQL

[GraphQL](http://graphql.org) is Vulcan's API layer, used to load data from the server with [Apollo](http://apollostack.com).

### Apollo

[Apollo](http://apollostack.com) is the set of tools used to manage the GraphQL layer. It consists of two parts, Apollo server and Apollo client.

* **Apollo server** is used to create the GraphQL API endpoint on the server.
* **Apollo client** (together with the `react-apollo` package) is used to query that endpoint and load data on the client.

Apollo is still young, but it's well-documented, evolving quickly, and supported by a great team.

### Meteor

[Meteor](http://meteor.com) is Vulcan's back-end layer that is used to handle the database as well as server and bundling the app. Note that although Meteor also provides its own data layer features, those features are not used except for user accounts.

Each layer is independent from the other two. You can take your React components and use them with a completely different stack, and – although we're not quite there yet – ultimately we also hope to make it easy to migrate out of Meteor.

The goal is to avoid getting stuck. If at any point you outgrow Vulcan, it should be possible to leave it with minimal refactoring.