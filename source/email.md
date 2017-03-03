---
title: Email
---

###Customizing Emails###

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

### Email Stying
Some styling of emails can be controlled in `settings.json`:

| Setting | Default | Description |
| --- | --- | --- |
| secondaryColor | '#444444' | Background of email header
| accentColor | '#DD3416' | Header and footer text
| siteName | "Nova" | Use setting key 'title'
| tagline |  |
| footer | | Footer text
| logoUrl | |
| logoHeight | |
| logoWidth | | 

### Newsletter

In your local development environment, you can preview your newsletter at `http://localhost:3000/email/newsletter`
