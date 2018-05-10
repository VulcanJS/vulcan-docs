---
title: Internationalization
---

There are two different yet related aspects to internationalization: making an app or package available for translation so that it can be used in any language, and actually building multi-language sites. 

*Note: W.I.P., refer to the `devel` branch for most of the following.*

## Locales

A locale is a language variant, such as UK English, American English, Canadian French, France French, and so on. 

### Setting a Default Locale

The easiest way to tell Vulcan which language to use is to set the `locale` field in your `settings.json` file:

```js
{
  "public": {
    "locale": "en"
  }
}
```

This will act as the default locale and be applied unless a specific locale has been selected by the server (based on the URL, user cookie, or user data). 

### Registering Locales

For multi-language apps, you can also register one or more locales:

```js
import { registerLocale } from 'meteor/vulcan:core';

registerLocale({
  id: 'en', // will be used as a general identifier and URL slug
  aliases: ['en_US', 'en_UK'], // other strings that should be matched
  label: '🇺🇸', // an optional label used in forms, etc.
  domains: ['http://localhost:3000', 'http://en.zenshome.jp'], // domain aliases that should use the locale
});
```

### Picking a Locale

Vulcan can pick a locale to show users based on a number of factors. By order of priority (higher priority first): 

- If the user is logged in, the `locale` property on their profile object. 
- A `locale` cookie. 
- The current domain (`mysite.com`, `mysite.fr`, `mysite.jp`, etc.).
- The current locale URL segment (`mysite.com/en/foo`, `mysite.com/fr/foo`, etc.).

### Switching Locales

You can switch locales by calling `setLocale` and `getLocale` (available on the React context) from within your components: 

```js
const LanguageSwitcher = ({}, { setLocale, getLocale }) => (
  <div className="language-switcher">
    <a className={`${getLocale() === 'en' ? 'active' : ''}`} href="javascript:void(0)" onClick={()=>setLocale('en')}>EN</a>
    <a className={`${getLocale() === 'ja' ? 'active' : ''}`} href="javascript:void(0)" onClick={()=>setLocale('ja')}>日本語</a>
  </div>
)

LanguageSwitcher.contextTypes = {
  getLocale: PropTypes.func,
  setLocale: PropTypes.func,
};

registerComponent('LanguageSwitcher', LanguageSwitcher);
```

Note that calling `setLocale` will automatically set the `locale` cookie, as well as the `user.locale` property on the user profie object if the user is logged in. 

## Strings

All packages included in the main Vulcan repo are ready for internationalization and come with English strings. 

### Adding & Replacing Strings

You can add strings using the `addStrings` function:

```js
import { addStrings } from 'meteor/vulcan:core';

addStrings('en', {
  'hello_world': 'Hello world!'
});

addStrings('fr', {
  'hello_world': 'Bonjour le monde!'
});
```

Note that this can be used not only for adding new languages, but also to replace strings in the current language anywhere in your app: 

```js
import { addStrings } from 'meteor/vulcan:core';

addStrings('en', {
  'hello_world': 'Greetings planet!'
});
```

### Internationalizing Components

There are two ways to use internationalized strings inside your components, `<FormattedMessaged/>` and `intl()`. 

#### FormattedMessage

`<FormattedMessage/>` is a React component that will generated a `<span>` tag for your internationalized strings:

```js
import React from 'react';

const MyComponent = () => 
  <div>
    <FormattedMessage id="hello_world" />
  </div>
```

It also takes variables as the `values` prop:

```js
import React from 'react';
import { addStrings } from 'meteor/vulcan:core';

addStrings('en', {
  'hello_last_name_first_name': 'Hello, {firstName} {lastName}'
});

const MyComponent = ({ user }) => 
  <div>
    <FormattedMessage id="hello_world" values={{ firstName: user.firstName, lastName: user.lastName }}/>
  </div>
```

And a `defaultValue` prop as a fallback:

```js
import React from 'react';
import { addStrings } from 'meteor/vulcan:core';

addStrings('en', {
  'hello_world': 'Greetings planet!'
});

const MyComponent = ({ user }) => 
  <div>
    <FormattedMessage id="hello_world" defaultValue="Hello world!" />
  </div>
```

Finally, you can also pass `html={true}` to evaluate the entire string as HTML:

```js
import React from 'react';
import { addStrings } from 'meteor/vulcan:core';

addStrings('en', {
  'email_me_at': `Email me at <a href="mailto:{email}">{email}</a>`
});

const MyComponent = ({ user }) => 
  <div>
    <FormattedMessage id="email_me_at" values={{ email: user.email }} html={true}/>
  </div>
```

#### formatMessage()

In some cases, you don't want to render out your string as a `<span>`. For example, you might want to use it inside an error message. You can do so using the `formatMessage` function, which requires adding the `intl` object to you component's context types

```js
import { intlShape } from 'meteor/vulcan:i18n';

class MyComponent extends React.Component {

  constructor() {
    super();
  }

  showAlert() {
    alert(this.context.intl.formatMessage({ id: 'hello_world' }));
  }

  render() {
    return <button onClick={this.showAlert}>Click me!</button>;
  }

};

MyComponent.contextTypes = {
  intl: intlShape
};
```

`formatMessages`' first argument is an object containing `id` and `defaultValue` properties, while its second argument is a `values` object (see previous section).

### Using react:intl

By default, Vulcan uses the `vulcan:i18n` package for internationalization. It supports an extremely limited set of i18n features by design in order to keep the Vulcan bundle size low, but all `vulcan:i18n` APIs (except for `html={true}`) are 100% compatible with the [react-intl](https://github.com/yahoo/react-intl) package. 

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

## Internationalizing Content

In addition to translating your app's UI, you'll sometimes want to translate your site's actual content. For example, you might be building an online store where product names need to be translated. 

### Field Schemas

In order to internationalize a field, you'll need to replace its `String` type with `getIntlString()` (imported from `vulcan:core`):

```js
title: {
  type: getIntlString(),
  optional: false,
  viewableBy: ['guests'],
  insertableBy: ['admins'],
  editableBy: ['admins'],
},
```

Note that this changes how data is stored in your database, from:

```
{ title: 'My Awesome Product' }
```

To: 

```
{ title { en: 'My Awesome Product', fr: 'Mon Super Produit'} }
```

### GraphQL API

Thanks to the power of GraphQL, your API will not change when internationalizing a field. In other words, `title` will remain a `String` when fetched through the GraphQL API (thanks to the `@intl` custom directive applied automatically to all internationalized fields). 

There are two ways to determine that string's language: 

- Set the `locale` GraphQL header.
- Pass the locale field argument (`title(locale: "en")`).

Additionally, a special `*field*Intl` (e.g. `titleIntl`) field that returns all available translations will be automatically created.

Note that when querying your endpoint through your Vulcan front-end, the `locale` header should usually be set automatically for you. 

### Migrating Content

Vulcan includes a migration script that will take all internationalized schema fields (i.e. those using the `getIntlString()` type) and convert their data format. 

Just call `Vulcan.migrateIntlFields(locale)` from your `meteor shell` (where `locale` is the locale of the fields' current content) or from inside a `meteor.startup()` block (note that `Vulcan` is a global, meaning it doesn't need to be imported).
 
## Internationalizing Emails

Since emails are stored and sent on the server outside of the React component tree, their internationalization process is a little bit different. 

### Setting a Locale

You can pass a `locale` option when sending email: 

```js
await VulcanEmail.buildAndSend({
  to: user.email,
  emailName: 'bookingsInquired',
  variables: { documentId: booking._id },
  locale: 'en',
});
```

### Email Templates

Within an email template, internationalization strings are available as the `__` object: 

```
<h1>{{__.welcome_email_title}}</h1>
```

If your i18n strings keys contain special characters (dots, spaces, etc.), you can escape them using the following `[...]` syntax:

```
<h1>{{__.[emails.inquiry_approved]}}</h1>
```

Note that email internationalization is pretty basic and does not currently support string replacement. PR welcome!