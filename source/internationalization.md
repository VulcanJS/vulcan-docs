---
title: Internationalization
---

All packages included in the main Vulcan repo are ready for internationalization and come with English strings. 

*Note: Vulcan currently only supports picking one global language for your entire app. Letting the end user pick their own language via a language switcher is not currently supported, but is on the roadmap.* 

## Selecting a Language

Set the `locale` field in your `settings.json` file:

```js
{
  "public": {
    "locale": "en"
  }
}
```

## Adding & Replacing Strings

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

## Internationalizing Components

There are two ways to use internationalized strings inside your components, `<FormattedMessaged/>` and `intl()`. 

### FormattedMessage

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

### formatMessage()

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

## Using react:intl

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
