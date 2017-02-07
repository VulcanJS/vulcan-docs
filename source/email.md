---
title: Email
---

<h2 id="customizing-emails">Customizing Emails</h2>

Unlike components, emails don't use React but Spacebars, a variant of the Handlebars templating language.

All email templates live in the `nova:email-templates` package. In order to register a new template or override an existing one, first you must import it as a text asset in your `package.js` file (or store it in your `/public` directory):

```js
api.addAssets(['path/to/template/newReply.handlebars',], ['server']);
```

You'll then be able to load the contents of the file in your code with:

```js
Assets.getText("path/to/template/newReply.handlebars")
```

You can add a template with:

```js
import NovaEmail from 'meteor/nova:email';

NovaEmail.addTemplates({
  newReply: Assets.getText("path/to/template/newReply.handlebars")
});
```

Or override an existing one with:

```js
import NovaEmail from 'meteor/nova:email';

NovaEmail.templates.newReply = Assets.getText("path/to/template/newReply.handlebars");
});
```
