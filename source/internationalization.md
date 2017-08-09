---
title: Internationalization
---

All packages included in the main Vulcan repo are ready for internationalization and come with English strings. 

## Adding Languages

To add a new language, you need to:

1. Create a new package containing the internationalized strings (you can use `vulcan:i18n-en-us` as a model).
2. Publish that package to [Atmosphere](https://atmospherejs.com/) and then add it to your app using `meteor add username:packagename`, or else keep it in your repo as a local package.
3. Set `locale` to the locale name (`fr`, `en`, `ru`, etc.) in your settings.

Note: make sure the locale you set matches the language package you're adding.

If you create a new internationalization package, let us know so we can add it here!

## Available Languages

- [fr-FR](https://github.com/VulcanJS/nova-i18n-fr-fr)
- [es-ES](https://atmospherejs.com/fcallem/nova-i18n-es-es)
- [pl-PL](https://atmospherejs.com/lusch/nova-i18n-pl-pl)
- [ru-RU](https://github.com/fortunto2/nova-i18n-ru-ru)
- [de-DE](https://atmospherejs.com/fzeidler/nova-i18n-de-de)
- [pt-BR](https://github.com/lukasag/nova-i18n-pt-br)
- [zh-CN](https://github.com/qge/nova-i18n-zh-cn)
- [hu-HU](https://github.com/pal-pinter/vulcan-i18n-hu-hu)
- [sv-SE](https://github.com/Enliven-se/vulcan-strings-i18n-sv-se)

## Using react:intl

By default, Vulcan uses the `vulcan:i18n` package for internationalization. It supports an extremely limited set of i18n features by design in order to keep the Vulcan bundle size low, but all `vulcan:i18n` APIs are 100% compatible with the [react-intl](https://github.com/yahoo/react-intl) package. 

If you do need all of react-intl's features, here's how to enable them:

1. Create a new `_vulcan-i18n` package directory

2. Inside, create the following `package.js` file:

```js
Package.describe({
  name: 'vulcan:i18n',
  summary: "Use react-intl instead of vulcan:i18n"
});

Package.onUse(function (api) {
  api.use(['vulcan:lib']);
  api.mainModule('i18n.js', 'server');
  api.mainModule('i18n.js', 'client');
});
```

And the following `i18n.js` file:

```js
export * from 'react-intl';
```

You'll now have two packages named `vulcan:i18n`, but because your newly created `_vulcan-i18n` directory starts with `_`, it will take alphabetical precedence over the default `vulcan-i18n` and be used by your app instead. 

You can keep importing from `vulcan:i18n` as usual, except `vulcan:i18n` will now simply be a clone of `react-intl`. 

## i18n messages in components

In order to have access to i18n functions like `formatMessage` you have to add `intlShape` to you component's context types

```
class MyComponent extends React.Component {
  render() {
    return(<h1>{this.context.intl.formatMessage({id: 'i18n.title'})}</h1>);
  }
};

MyComponent.contextTypes = {
  intl: intlShape
};
```
