---
title: Internationalization
---

Vulcan is internationalized using [react-intl](https://github.com/yahoo/react-intl/). To add a new language, you need to:

1. Create a new package containing the internationalized strings (you can use `vulcan:i18n-en-us` as a model).
2. Publish that package to Atmosphere and then add it to your app using `meteor add username:packagename.
3. Set `locale` to the locale name (`fr`, `en`, `ru`, etc.) in your settings.

Note: make sure the locale you set matches the language package you're adding.

If you create a new internationalization package, let us know so we can add it here!

- [fr-FR](https://github.com/TelescopeJS/nova-i18n-fr-fr)
- [es-ES](https://atmospherejs.com/fcallem/nova-i18n-es-es)
- [pl-PL](https://atmospherejs.com/lusch/nova-i18n-pl-pl)
- [ru-RU](https://github.com/fortunto2/nova-i18n-ru-ru)
- [de-DE](https://atmospherejs.com/fzeidler/nova-i18n-de-de)
- [pt-BR](https://github.com/lukasag/nova-i18n-pt-br)
- [zh-CN](https://github.com/qge/nova-i18n-zh-cn)
