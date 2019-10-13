---
title: Components
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
registerComponent({ name: 'Logo', component: Logo });
```

### Components & HoCs

To understand how theming works in Vulcan, it's important to understand how components and higher-order components (HoCs) interact. 

A higher-order component's role is to wrap a regular component to pass it a specific prop (such as a list of posts, the current user, the `Router` object, etc.). You can think of HoCs as specialized assistants that each hand the component a tool it needs to do its job. 

For example, this is how you'd pass the `currentUser` object to the `Logo` component:

```js
registerComponent({ name: 'Logo', component: Logo, hocs: [withCurrentUser] });
```

### "Delayed" HoCs

There are a few subtle differences between registering a component with `registerComponent` and then calling it with `Components.Foo`; and the more standard `export default Foo` and `import Foo from './Foo.jsx'` ES6 syntax. 

First, you can only override a component if it's been registered using `registerComponent`. This means that if you're building any kind of theme or plugin and would like end users to be able to replace specific components, you shouldn't use import/export. 

Second, both techniques also lead to different results when it comes to higher-order components (more on this below). If you write `export withCurrentUser(Foo)`, the `withCurrentUser` function will be executed immediately, which will trigger an error because the fragment it depends on isn't properly initialized yet. 

On the other hand, `registerComponent('Foo', Foo, withCurrentUser)` *doesn't* execute the function (note that we write `withCurrentUser` and not `withCurrentUser()`), delaying execution until app's initialization. 

But what about HoC functions that take arguments? For example if you were to write:

```js
registerComponent({ name: 'PostsList', component: PostsList, hocs: [withMulti(options)] });
```

The `withMulti(options)` would be executed immediately, and you would have no way of overriding the `options` object later on (a common use case being overriding a fragment).

For that reason, to delay the execution until the start of the app, you can use the following alternative syntax:

```js
registerComponent({ name: 'PostsList', component: PostsList, hocs: [ [withMulti, options] ] });
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

### Shortcut Syntax

Throughout the documentation and the example projects you might also see this equivalent `registerComponent` shortcut syntax:

```
registerComponent('ComponentName', Component, hoc1, hoc2, ...);
```

The first argument of `registerComponent` will be the component's name, the second the component itself, and any successive arguments will be interpreted as higher-order components and wrapped around the component.

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

Note that using `getRawComponent` is also needed because components get registered at runtime. So in our `CustomLogo` example, `Components.Logo` would not be defined yet. 

#### Alternative Approach

The main purpose behind the components API is to enable extending and replacing components defined in third-party themes and plug-ins. However, if this is not a concern for you, you can use the standard `export default Foo` and `import Foo from './foo.jsx'` approach without any trouble. 
