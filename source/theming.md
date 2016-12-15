---
title: Components & Theming
---

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

## Components & HoCs

To understand how theming works in Nova, it's important to understand how components and higher-order components (HoCs) interact. 

A higher-order component's role is to wrap a regular component to pass it a specific prop (such as a list of posts, the current user, the `Router` object, etc.). You can think of HoCs as specialized assistants that each hand the component a tool it needs to do its job. 

In practice, to create a higher-order component you call a **factory function** on the component. For example, to give the component access to the current user, you would call:

```
const WrappedComponent = withCurrentUser(MyComponent);
```

Which would result a *new* `WrappedCopmonent` component that has `MyComponent` as a child. This has the consequence that properties and objects you set on `MyComponent` might not exist on `WrappedCopmonent`. 

For that reason, Nova provides a `getRawComponent` utility that lets you access the unwrapped “raw” component, provided said component has been registered with `registerComponent`:

```
MyComponent.foo = "bar";
const WrappedComponent = registerComponent(MyComponent, withCurrentUser);
console.log(WrappedComponent.foo); // undefined
console.log(getRawComponent(WrappedComponent).foo); // "bar"
```

Try to keep this in mind as you work with components.

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

In some cases, you'll need to change the fragment used by a component. You can do this using the following pattern:

1. Get the raw component without any HoCs.
2. Wrap it with `withList` or `withDocument` using the new fragment.
3. Re-register it using the same name. 

For example:

```js
import { Components, getRawComponent, registerComponent } from 'meteor/nova:lib';
import { withList } from 'meteor/nova:core';
import gql from 'graphql-tag';

const myNewFragment = gql`
  fragment myNewFragment on Post {
    ...
  }
`;

const options = {
  collection: Posts,
  queryName: 'postsListQuery',
  fragment: myNewFragment,
};

registerComponent('PostsList', getRawComponent('PostsList'), withList(options));
```

Note that because HoC factory functions (`withList`, `withDocument`, etc.) are called right away at runtime, you can't just replace a fragment by replacing the component it's defined on since that replacement would only take place after the fragment is used. 