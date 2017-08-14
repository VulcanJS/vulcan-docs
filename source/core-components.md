---
title: Core Components
---

Out of the box, VulcanJS provides a set of useful components you can use to quickly build applications. 

These can all be called using `<Components.XYZ/>`.

## App

This is the main component that wraps everything else in your app. Its main job is to initialize the internationalization context.

## Layout

The default `Layout` component only serves as a placeholder.

## Avatar

The `Avatar` component takes in a `user` and displays the user's avatar. Additionally, if you pass `link = {true}`, the avatar will be linked to the user's profile.

## ModalTrigger

The `ModalTrigger` component can be used to display its child component inside a modal window when the trigger component is clicked:

```js
<Components.ModalTrigger size={size} title="New Post" component={<MyButton/>}>
  <PostsNewForm />
</Components.ModalTrigger>
```

It accepts the following props:

- `className`
- `component`: the trigger component.
- `label`: if no trigger component is passed, you can also specify a label for a text link. 
- `size`: `large` or `small` (defaults to `large`). 
- `title`: the modal popup's title.

All props are optional, but you should pass at least either `component` or `label`. 

Note that the element passed as `component` needs to accept an `onClick` handler. In some cases, it might be necessary to wrap it inside an extra `<div`:

```js
<Components.ModalTrigger size={size} title="New Post" component={<div><MyButton/></div>}>
  <PostsNewForm />
</Components.ModalTrigger>
```

## Loading

Show a loading animation.

## Icon

Show an icon. Takes a `name` property:

```js
<Components.Icon name="approve" />
```

By default, Vulcan defines the following [FontAwesome](http://fontawesome.io/icons/) icons:

```js
Utils.icons = {
  expand: "angle-right",
  collapse: "angle-down",
  next: "angle-right",
  close: "times",
  upvote: "chevron-up",
  voted: "check",
  downvote: "chevron-down",
  facebook: "facebook-square",
  twitter: "twitter",
  googleplus: "google-plus",
  linkedin: "linkedin-square",
  comment: "comment-o",
  share: "share-square-o",
  more: "ellipsis-h",
  menu: "bars",
  subscribe: "envelope-o",
  delete: "trash-o",
  edit: "pencil",
  popularity: "fire",
  time: "clock-o",
  best: "star",
  search: "search",
  approve: "check-circle-o",
  reject: "times-circle-o",
  views: "eye",
  clicks: "mouse-pointer", 
  score: "line-chart",
  reply: "reply",
  spinner: "spinner",
  new: "plus",
  user: "user",
  like: "heart",
  image: "picture-o",
};
```

You can add or change icons with:

```js
import { Utils } from 'meteor/vulcan:core';

Utils.icons.approve = "handshake-o";

## HeadTags

See [Head Tags](/head-tags.html) section.

## Card

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

## Datatable

The `Datatable` component is used to show a dynamic datatable for a given collection. It takes the following props:

- `collection`: the collection to load data from.
- `columns`: an array containing a list of columns (see below).
- `options`: the `options` object passed to `withList` in order to load the data (optional).

The `columns` array is an array of either column names, objects, or both. If you're passing objects, each one should contain the following properties:

- `name`: the name of the column.
- `order`: the order of the column within the table (optional).
- `component`: a custom component used to render this column's table cells (optional).

For example:

```js
<Components.Datatable 
  collection={Rooms} 
  columns={['name', 'description']} 
/>
```

Or:

```js
const columns = [
  'name',
  {
    name: 'email',
    order: 10,
    component: AdminUsersEmail
  },
  'createdAt',
  {
    name: 'actions',
    order: 100,
    component: AdminUsersActions
  },
];

<Components.Datatable 
  collection={Users} 
  columns={columns} 
  options={{
    fragmentName: 'UsersAdmin',
    terms: {view: 'usersAdmin'},
    limit: 20
  }}
/>
```

Note that columns do not necessarily have to correspond to schema fields. For example, the `vulcan:admin` package's Users Dashboard adds an `actions` column.
