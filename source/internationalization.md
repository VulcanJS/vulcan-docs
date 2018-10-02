---
title: Internationalization
---

There are two different yet related aspects to internationalization: making an app or package available for translation so that it can be used in any language, and actually building multi-language sites. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/yO4MAdmiiCc?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Debugging

**If you've got the [debug package](/debug.html) enabled, an Internationalization debugging UI is available at [http://0.0.0.0:3000/debug/i18n](http://0.0.0.0:3000/debug/i18n).**


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

[TODO] You can also specify your locale as an array in order to enable fallback locales. For example, here's how you can tell Vulcan to use the `fr_CA` locale whenever possible, but to fallback to `fr_FR` if it's not available:

```js
{
  "public": {
    "locale": [ "fr_CA", "fr_FR"]
  }
}
```

### Registering Locales

For multi-language apps, you can also register one or more locales:

```js
import { registerLocale } from 'meteor/vulcan:core';

registerLocale({
  id: 'en', // will be used as a general identifier and URL slug
  aliases: ['en_US', 'en_UK'], // other strings that should be matched [TODO]
  label: 'English', // an optional label used in forms, etc.
  domains: ['http://localhost:3000', 'http://en.zenshome.jp'], // domain aliases that should use the locale [TODO]
});
```

### Picking a Locale

Vulcan can pick a locale to show users based on a number of factors. By order of priority (higher priority first): 

- If the user is logged in, the `locale` property on their profile object. 
- A `locale` cookie. [TODO]
- The current domain (`mysite.com`, `mysite.fr`, `mysite.jp`, etc.). [TODO]
- The current locale URL segment (`mysite.com/en/foo`, `mysite.com/fr/foo`, etc.). [TODO]
- Browser settings. [TODO]

Ideally, all factors should resolve similarly both on the client, and on the server during the SSR process.

A big difference between these techniques is whether you're actually using different URLs for different locales or not. Putting everything on the same domain with a simple toggle is easy to set up and easy to navigate for users (provided your locale controls are easily accessible), but you won't reap any SEO benefits from supporting multiple languages. On the other hand, having different domains can be good for SEO, but might require users to log in again if they switch locales. For those reasons, using different paths on the *same* domain can be a good middle ground. 

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

Note that calling `setLocale` will automatically set the `locale` cookie, as well as the `user.locale` property on the user profie object if the user is logged in. It will also reset the Apollo store in order to reload a fresh set of data from the server. 

Alternatively, if you're setting the locale based on the current domain, you can also dispense with `setLocale()` altogether and simply redirect users to the correct domain or URL. 

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

### getLocale()

For content that is *not* stored as an internationalized string, you can always just use `getLocale()` to pick the correct string or component:

```js
const Intro = ({}, { setLocale, getLocale }) => (
  <div className="intro">
    {getLocale() === 'en' && <Components.IntroEn/>}
    {getLocale() === 'fr' && <Components.IntroFr/>}
    {getLocale() === 'jafr' && <Components.IntroJa/>}
  </div>
)

Intro.contextTypes = {
  getLocale: PropTypes.func,
  setLocale: PropTypes.func,
};

registerComponent('Intro', Intro);
```

## Internationalizing Content

In addition to translating your app's UI, you'll sometimes want to translate your site's actual content. For example, you might be building an online store where product names need to be translated. 

### Storing Translations

When migrating from a single-language app, you'll need to change how data is stored in your database by adding a `foo_intl` field (for example `title_intl`) in accordance with Mongo's [text search](https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/) guidelines. So you'll go from:

```
{ 
  title: 'My Awesome Product', // internationalized field
  createdAt: ISODate("2018-05-17T00:41:40.247Z") // non-internationalized field
}
```

To: 

```
{ 
  title: 'My Awesome Product' 
  title_intl: [
    {
      locale: 'en',
      value: 'My Awesome Product'
    },
    {
      locale: 'fr',
      value: 'Mon Super Produit'
    }
  ],
  createdAt: ISODate("2018-05-17T00:41:40.247Z")
}
```

Note that the original `title` field then becomes optional, and is kept mainly for backwards compatibility when internationalizing existing apps, or as a fallback when no translations exist. The following is also valid if you'd rather not duplicate content:

```
{ 
  title_intl: [
    {
      locale: 'en',
      value: 'My Awesome Product'
    },
    {
      locale: 'fr',
      value: 'Mon Super Produit'
    }
  ],
  createdAt: ISODate("2018-05-17T00:41:40.247Z")
}
```

#### Translation Order

Note that it's very important that locales are stored in the same order in your database as they are declared using `registerLocale`. 

In other words, for the above examples you need to make sure you register the `en` locale before the `fr` locale, otherwise the generated form fields might not match up with your translation strings. 

#### Migration Script

Vulcan includes a migration script that will take all internationalized schema fields (i.e. those using the `getIntlString()` type) and convert their data format. 

Just call `Vulcan.migrateIntlFields(locale)` from your `meteor shell` (where `locale` is the locale of the fields' current content) or from inside a `meteor.startup()` block (note that `Vulcan` is a global, meaning it doesn't need to be imported).

### Field Schema

In order to make a field support multiple languages, all you need to do is replace its `String` type with `getIntlString()` (imported from `vulcan:core`):

```js
title: {
  type: getIntlString(),
  optional: false,
  canRead: ['guests'],
  canCreate: ['admins'],
  canUpdate: ['admins'],
},
```

Behind the scenes, Vulcan will then create a new `title_intl` schema field, which will be an Array of JSON objects. 

This generated field will also be available through your GraphQL API if you need to fetch all translations.

### GraphQL API

Thanks to the power of GraphQL, your API will not change when internationalizing a field. In other words, `title` will remain a `String` when fetched through the GraphQL API (thanks to the `@intl` custom directive applied automatically to all internationalized fields). 

There are two ways to determine that string's language: 

- Set the `locale` GraphQL header.
- Pass the locale field argument (`title(locale: "en")`).

Additionally, a special `*field*_intl` (e.g. `title_intl`) field that returns all available translations will also be available.

Note that when querying your endpoint through your Vulcan front-end, the `locale` header should usually be set automatically for you. Also, you can also specify a locale when using the GraphQL API on the server with `runQuery`: 

### Locale-Specific Content

In some cases, you'll want to restrict some content to only be displayed in a specific locale. This is not yet supported, but PRs are welcome! [TODO]

### Search

Search is not currently supported for internationalized content [TODO].
 
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

Note that email internationalization is pretty basic and does not currently support string replacement. PR welcome! [TODO]
