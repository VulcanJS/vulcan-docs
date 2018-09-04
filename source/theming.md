---
title: Components & Theming
---

## Using Components

Assuming a component is registered, you can use it with `Components.Foo`. For example:

```js
import { Components } from 'meteor/vulcan:core';

const Header = props => {
  return (
    <div>
      <Components.Logo />
    </div>
  )
}
```

## Registering Components

Vulcan components are all listed in `vulcan:core`'s `Components` object. You can add new ones with the 'registerComponent` method:

```js
import { registerComponent } from 'meteor/vulcan:core';

const Logo = props => {
  return (
    <div>/* component code */</div>
  )
}
registerComponent('Logo', Logo);
```

### Components & HoCs

To understand how theming works in Vulcan, it's important to understand how components and higher-order components (HoCs) interact. 

A higher-order component's role is to wrap a regular component to pass it a specific prop (such as a list of posts, the current user, the `Router` object, etc.). You can think of HoCs as specialized assistants that each hand the component a tool it needs to do its job. 

The first argument of `registerComponent` is the component's name, the second is the component itself, and any successive arguments will be interpreted as higher-order components and wrapped around the component.

For example, this is how you'd pass the `currentUser` object to the `Logo` component:

```js
registerComponent('Logo', Logo, withCurrentUser);
```

### "Delayed" HoCs

There are a few subtle differences between registering a component with `registerComponent` and then calling it with `Components.Foo`; and the more standard `export default Foo` and `import Foo from './Foo.jsx'` ES6 syntax. 

First, you can only override a component if it's been registered using `registerComponent`. This means that if you're building any kind of theme or plugin and would like end users to be able to replace specific components, you shouldn't use import/export. 

Second, both techniques also lead to different results when it comes to higher-order components (more on this below). If you write `export withCurrentUser(Foo)`, the `withCurrentUser` function will be executed immediately, which will trigger an error because the fragment it depends on isn't properly initialized yet. 

On the other hand, `registerComponent('Foo', Foo, withCurrentUser)` *doesn't* execute the function (note that we write `withCurrentUser` and not `withCurrentUser()`), delaying execution until app's initialization. 

But what about HoC functions that take arguments? For example if you were to write:

```js
registerComponent('PostsList', PostsList, withMulti(options));
```

The `withMulti(options)` would be executed immediately, and you would have no way of overriding the `options` object later on (a common use case being overriding a fragment).

For that reason, to delay the execution until the start of the app, you can use the following alternative syntax:

```js
registerComponent('PostsList', PostsList, [withMulti, options]);
```
### Accessing Raw Components

Going back to our example:

```
const WrappedComponent = withCurrentUser(MyComponent);
```

This would result a *new* `WrappedComponent` component that has `MyComponent` as a child. This has the consequence that properties and objects you set on `MyComponent` might not exist on `WrappedComponent`. 

For that reason, Vulcan provides a `getRawComponent` utility that lets you access the unwrapped “raw” component, provided said component has been registered with `registerComponent`:

```
MyComponent.foo = "bar";
const WrappedComponent = registerComponent(MyComponent, withCurrentUser);
console.log(WrappedComponent.foo); // undefined
console.log(getRawComponent(WrappedComponent).foo); // "bar"
```

## Replacing Components

If you only need to modify a single component, you can simply override it with a new one without having to touch the original package.code

For example, if you wanted to use your own `CustomLogo` component you would do:

```js
import { replaceComponent } from 'meteor/vulcan:core';

const CustomLogo = props => {
  return (
    <div>/* custom component code */</div>
  )
}
replaceComponent('Logo', CustomLogo);
```

Note that `replaceComponent` will preserve any HoCs originally defined using `registerComponent`. In other words, in the following example:

```js
registerComponent('Logo', Logo, withCurrentUser, withRouter);
replaceComponent('Logo', CustomLogo);
```

The `CustomLogo` component will also be wrapped with `withCurrentUser` and `withRouter`.

Once you've replaced the `Logo` component with your own `CustomLogo`, `Components.Logo` will now point to `CustomLogo`. If you want an easy way to keep track of which components have been customized, you could add a `custom` attribute when calling the component as a reminder for yourself:

```js
import { Components } from 'meteor/vulcan:core';

const CustomHeader = props => {
  return (
    <div>
      <Components.Logo custom />
    </div>
  )
}
```

## Extending Components

Components are generally defined as functional stateless components, unless they contain extra logic (lifecycle methods, event handlers, etc.) in which case they'll be defined as ES6 classes.

For components defined as ES6 classes, make sure you `extend` the original component. This will let you pick and choose which methods you actually need to replace, while inheriting the ones you didn't specify in your new component.

In order to extend the original, non-wrapped component we use the `getRawComponent` method:

```js
class CustomLogo extends getRawComponent('Logo'){
  render() {
    return (
      <div>/* custom component code */</div>
    )
  }
}
replaceComponent('Logo', CustomLogo);
```

Note that using `getRawComponent` is also needed because components get registered at runtime. So in our `CustomLogo` example, `Components.Logo` would not be define dyet. 

#### Alternative Approach

The main purpose behind the components API is to enable extending and replacing components defined in third-party themes and plug-ins. However, if this is not a concern for you, you can use the standard `export default Foo` and `import Foo from './foo.jsx'` approach without any trouble. 

## Core Components

In addition to components that are part of a specific theme or example package, a few components are provided with `vulcan:core`.

### App

The `App` component takes care of loading data for the current active user and setting up internationalization. Most of the time, you shouldn't need to worry about it. 

### Error404

This is a default “page not found” error component. This ensures Vulcan has something to show even when you remove all themes and routes. 

### Icon

The icon component is a simple wrapper to display [Font Awesome](http://fontawesome.io/) icons. If you'd like to use a different icon set, you can just replace it using the usual `replaceComponent` technique. 

### Layout

This is a default layout. It'll usually be replaced with each theme's own custom layout.

### Loading

A simple loading spinner component you can use to show loading states. Its corresponding styles currently live in the `vulcan:base-styles` package. 

### ModalTrigger

A component used to display another component inside a [react-bootstrap](https://react-bootstrap.github.io/) modal popup:

```js
<Components.ModalTrigger size={size} title={context.intl.formatMessage({id: "posts.new_post"})} component={button}>
  <Components.PostsNewForm />
</Components.ModalTrigger>
```

It takes the following props:

- `size`: `large` or `small`, how wide the modal popup should be.
- `title`: the modal's title.
- `component`: the component used to trigger the modal when clicked.

### ShowIf

A component that takes a `check` function and an optional `document`, and performs the check on the document for the current user. If the check succeeds, the children are displayed. If not, `failureComponent` is displayed instead. 

```js
<Components.ShowIf check={Comments.options.mutations.edit.check} document={this.props.comment}>
  <div>
    <a className="comment-edit" onClick={this.showEdit}><FormattedMessage id="comments.edit"/></a>
  </div>
</Components.ShowIf>
```

Note that due to the way React works, children component code is executed even if the check fails (the component just won't be displayed).