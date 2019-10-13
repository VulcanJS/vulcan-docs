---
title: Other Components
---

Out of the box, VulcanJS provides a set of useful components you can use to quickly build applications. 

These live in the `vulcan:core` package and can all be called using `<Components.XYZ/>`. Most of them are implemented using a [component library](/ui-components) such as React Bootstrap.


### App

This is the main component that wraps everything else in your app. Its main job is to initialize the internationalization context.

### Avatar

The `Avatar` component takes in a `user` and displays the user's avatar. Additionally, if you pass `link = {true}`, the avatar will be linked to the user's profile.

### Card

The `Card` component is used to display a card for a single document. It takes the following props:

- `document`: the document to display.
- `collection`: the collection the document belongs to.
- `currentUser`: if passed, an "Edit Document" button will be shown (if the user has edit permissions on the document).
- `fields`: optionally, you can pass a list of field names to limit the fields shown in the card. 

The component will do its best to "guess" how to display each of the document's properties based on their type and the collection schema:

```js
<Components.Card 
  fields={['name', 'year', 'review']} 
  collection={Movies}
  document={movie}
  currentUser={currentUser}
/>
```

### EditButton

A button that triggers a modal window for editing a document. 

### Flash

Adds a zone to dynamically show “flash” messages (errors, successes, notifications, etc.).`

### HeadTags

See [Head Tags](/head-tags.html) section.

### Layout

The default layout is just a placeholder that can be replaced by your own. 

### Loading

Show a loading animation.

### MutationButton

TODO

### NewButton

A button that triggers a modal window for creating a new document. 