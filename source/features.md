---
title: Features
---

Core Nova features include:

<h2 id="schema-generation">GraphQL Schema Generation</h2>

Nova will automatically generate GraphQL schemas for your collections based on their [SimpleSchema](https://github.com/aldeed/meteor-simple-schema) JSON schema. 

This prevents you from having to specify your schema twice in two different formats. Although please note that this feature is completely optional, and that you can also specify your schema manually if you prefer. 

<h2 id="automated-forms">Automated Forms</h2>

Nova will also use your schema to generate client-side forms and handle their submission via the appropriate Apollo mutation. 

For example, here's how you would display a form to edit a single movie:

```js
<NovaForm 
  collection={Movies} 
  documentId={props.documentId}
  queryName="moviesListQuery"
  showRemove={true}
/>
```

The `queryName` option tells NovaForm which query should be automatically updated once the operation is done, while the `showRemove` option add a “Delete Movie” button to the form. 

Note that NovaForm will also take care of loading the document to edit, if it's not already loaded in the client store. 

<h2 id="data-loading">Easy Data Loading</h2>

Nova features a set of data loading helpers to make loading Apollo data easier, `withList` (to load a list of documents) and `withDocument` (to load a single document). 

For example, here's how you would use `withList` to pass a `results` prop containing all movies to your `MoviesList` component:

```js
const listOptions = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragment: fragment,
};

export default withList(listOptions)(MoviesList);
```

You can pass a fragment to control what data is loaded for each document.

<h2 id="security-validation">Schema-based Security & Validation</h2>

All of Nova's security and validation is based on your collection's schema. For each field of your schema, you can define the following functions:

- `viewableIf`
- `insertableIf`
- `editableIf`

They all take the current user as argument (and optionally the document being affected) and check if the user can perform the given action. 

<h2 id="groups-permissions">Groups & Permissions</h2>

Nova features a simple system to handle user groups and permissions. For example, here's how you would define that all users can create new movies and edit/remove their own, but only admins can edit or remove other user's movies:

```js
const defaultActions = [
  "movies.new",
  "movies.edit.own",
  "movies.remove.own",
];
Users.groups.default.can(defaultActions);

const adminActions = [
  "movies.edit.all",
  "movies.remove.all"
];
Users.groups.admins.can(adminActions);
```

You can then reference these actions in your mutation checks. 

<h2 id="other-features">Other Features</h2>

Out of the box, Nova also includes many other time-saving features, such as:

- Internationalization
- Server-side rendering
- Utilities for theming and extending components
- Email and email templates handling
- Preset boilerplate mutations

And of course, using Nova also means you get access to all the non-core Nova packages, such as `nova:posts`, `nova:comments`, `nova:newsletter`, `nova:search` etc.
