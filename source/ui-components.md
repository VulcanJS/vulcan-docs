---
title: UI Components
---

While Vulcan itself doesn't provide its own UI components (besides core utilites such as `Layout`, `App`, etc.), it does make use of a common component API that can serve as a bridge between your app and UI frameworks such as [React Bootstrap](https://react-bootstrap.github.io/) or [Material UI](https://www.material-ui.com/#/).

By using this API instead of depending on a framework's components directly, Vulcan plugins and projects can make it easy to easily swap from one UI framework to another. 

## Form Components

- Checkbox
- Checkboxgroup
- Date
- Datetime
- Default
- Email
- FormControl
- Number
- Radiogroup
- Select
- Textarea
- Time
- URL

## Other Components

### Alert

### Button

### Dropdown

### Modal

### ModalTrigger

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

Note that the element passed as `component` needs to accept an `onClick` handler. In some cases, it might be necessary to wrap it inside an extra `<div>`:

```js
<Components.ModalTrigger size={size} title="New Post" component={<div><MyButton/></div>}>
  <PostsNewForm />
</Components.ModalTrigger>
```

### Table

### Tooltip