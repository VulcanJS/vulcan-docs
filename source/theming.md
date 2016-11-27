---
title: Theming
---

<h2 id="customizing-components">Customizing Components</h2>

Apart from a couple exceptions, almost all React components in Nova live inside the `nova:base-components` package. 

If you only need to modify a single component, you can simply override it with a new one without having to touch the `nova:base-components` package.

For example, if you wanted to use your own `CustomLogo` component you would do:

```js
const CustomLogo = (props) => {
  return (
    <div>/* custom component code */</div>
  )
}
Telescope.components.register('Logo', CustomLogo);
```

Components are generally defined as functional stateless components, unless they contain extra logic (lifecycle methods, event handlers, etc.) in which case they'll be defined as ES6 classes.

For components defined as ES6 classes, make sure you `extend` the original component. This will let you pick and choose which methods you actually need to replace, while inheriting the ones you didn't specify in your new component:

```js
class CustomLogo extends Telescope.components.Logo{
  render() {
    return (
      <div>/* custom component code */</div>
    )
  }
}
Telescope.components.register('Logo', CustomLogo);
```

You can make the override at any point, as long as it happens before the `<Telescope.components.Logo/>` component is called from a parent component.
