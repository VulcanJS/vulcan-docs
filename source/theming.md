---
title: Components & Theming
---

## Using Components

There are two ways to use components.

- Outisde a compomnent, you can call `getComponent('Foo')`.
- Inside a component, you can simply use `<Components.Foo/>`.

The advantage of calling the `getComponent` function instead of using `Components.Foo` directly, is that you can ensure that the `Foo` component is only resolved when the function is called, which in turns makes it possible to override them anywhere in your code. 

## Registering Components

Nova components are all listed in `nova:core`'s `Components` object. You can add new ones with the 'registerComponent` method:

```js
const Logo = props => {
  return (
    <div>/* component code */</div>
  )
}
registerComponent('Logo', Logo);
```

The first argument of `registerComponent` is the component's name, the second is the component itself, and any successive arguments will be interpreted as higher-order components and wrapped around the component:

```js
registerComponent('Logo', Logo, withCurrentUser, withRouter);
```

### Components & HoCs

To understand how theming works in Nova, it's important to understand how components and higher-order components (HoCs) interact. 

A higher-order component's role is to wrap a regular component to pass it a specific prop (such as a list of posts, the current user, the `Router` object, etc.). You can think of HoCs as specialized assistants that each hand the component a tool it needs to do its job. 

In practice, to create a higher-order component you call a **factory function** on the component. For example, to give the component access to the current user, you would call:

```
const WrappedComponent = withCurrentUser(MyComponent);
```

Which would result a *new* `WrappedComponent` component that has `MyComponent` as a child. This has the consequence that properties and objects you set on `MyComponent` might not exist on `WrappedCopmonent`. 

For that reason, Nova provides a `getRawComponent` utility that lets you access the unwrapped “raw” component, provided said component has been registered with `registerComponent`:

```
MyComponent.foo = "bar";
const WrappedComponent = registerComponent(MyComponent, withCurrentUser);
console.log(WrappedComponent.foo); // undefined
console.log(getRawComponent(WrappedComponent).foo); // "bar"
```

Try to keep this in mind as you work with components.

### "Delayed" HoCs

In some cases, you don't want a HoC function to be executed immediately. For example if you write:

```js
registerComponent('PostsList', PostsList, withList(options));
```

The `withList(options)` will be executed immediately, and you will have no way of overriding the `options` object later on (a common use case being overriding a fragment).

To delay the execution until the start of the app, you can use the following alternative syntax:

```js
registerComponent('PostsList', PostsList, [withList, options]);
```

## Replacing Components

Apart from a couple exceptions, almost all React components in Nova live inside the `nova:base-components` package. 

If you only need to modify a single component, you can simply override it with a new one without having to touch the `nova:base-components` package.

For example, if you wanted to use your own `CustomLogo` component you would do:

```js
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

## Overriding Fragments

In some cases, you'll need to change the fragment used by a component. To do so, you can re-register a new fragment

```js
import { registerFragment } from 'meteor/nova:lib';
import gql from 'graphql-tag';

const CustomPostsListFragment = gql`
  fragment CustomPostsList on Post {
    _id
    title
    url
    color # new custom property!
  }
`;

registerFragment(CustomPostsListFragment, 'PostsList');
registerFragment(CustomPostsListFragment, 'PostsPage');
```

You can learn more about fragments in the [Data Layer](/data-layer.html#fragments) section. 