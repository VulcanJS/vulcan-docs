---
title: Email
---

Vulcan includes a simple-yet-powerful email-sending system.
## Debugging

**If you've got the [debug package](/debug.html) enabled, an email debugging UI is available at [http://0.0.0.0:3000/debug/emails](http://0.0.0.0:3000/debug/emails).**

## Setup

In order to send emails, you'll need to set up an email provider. You can do so by providing an SMTP URL (e.g. `smtp://username:password@smtp.mailgun.org:587/`) in your `settings.json` file under the `mailUrl` property.

Note: if your username contains an `@` make sure to replace it with `%40` to escape it. 

## Registering Emails

Just like components, emails need to be centrally registered with Vulcan.

```js
import VulcanEmail from 'meteor/vulcan:email';

VulcanEmail.addEmails({

  test: {
    template: 'test',
    path: '/email/test',
    data() {
      return {date: new Date()};
    },
    subject(data) {
      return 'This is a test';
    },
  }

});
```

The `VulcanEmail.addEmails` function takes an object that itself contains individual objects for each email. Each email object can have the following properties:

#### `template`

The name of the template used to display the email (see below).

#### `subject(data)`

Either a function that takes in the email's `data` as argument and returns the subject, or a string. 

Keep in mind that `data` will be empty in certain cases (such as when listing emails in the Emails dashboard).

#### `data(variables)`

A function that returns an object used to populate the email. It will either be passed a `variables` object by `VulcanEmail.buildAndSend`, or use `testVariables`.

#### `query`

A string containing a GraphQL query used to populate the email see below). If both `data` and `query` are provided, their results will be merged and passed on to the email.

Similarly to `data()`, the query will either use the `variables` object passed by `VulcanEmail.buildAndSend`, or use `testVariables`.

#### `path`

A path for the route at which the email will be available to preview (when using `vulcan:debug`).

#### `testVariables`

An object containing default variables passed to `query()` when testing the email. If not specified, a blank object will be used. 

## Querying For Data

Just like you can use GraphQL to query for data on the client and populate your React component's contents, you can also use it to query for data on the server and populate your Handlebars templates:

```js
VulcanEmail.addEmails({

  accountApproved: {
    template: "accountApproved",
    path: "/email/account-approved/:_id?",
    subject(data) {
      const user = _.isEmpty(data) ? {displayName: '[user]'} : data.UsersSingle;
      return "Welcome ${user.displayName}! Your account has been approved.";
    },
    query: `
      query UsersSingleQuery($documentId: String){
        UsersSingle(documentId: $documentId){
          displayName
        }
        SiteData{
          title
          url
        }
      }
    `
  }

});
```

A few things to note:

- Queries can accept variables, which will be passed by `VulcanEmail.buildAndSend()` (see below); or `testVariables` when testing.
- A single query can query resolvers for entirely different collections. This lets you mix-and-match different data types in the same email. 
- If the default resolvers (`FooList`, `FooSingle`, etc.) don't return the data you need for a specific email, don't forget you can also create your own custom resolvers with `addGraphQLResolvers` and `addGraphQLQuery`.

## Email Templates

Unlike components, emails don't use React but the [Handlebars](http://handlebarsjs.com/) templating language. Before you can use a template with Vulcan's email system, you'll need to register it. A 'wrapper' template must first be registered as a container for your templates (e.g. Vulcan-Starter/packages/example-forum/lib/server/email/templates/common/wrapper.handlebars).

In order to register a new template or override an existing one, first you must import it as a text asset in your `package.js` file (or store it in your `/public` directory):

```js
api.addAssets(['path/to/template/wrapper.handlebars',], ['server']);
```

You'll then be able to add a new template with `VulcanEmail.addTemplates`:

```js
import VulcanEmail from 'meteor/vulcan:email';

VulcanEmail.addTemplates({
  wrapper: Assets.getText('path/to/template/wrapper.handlebars')
});
```

Or override an existing one with:

```js
import VulcanEmail from 'meteor/vulcan:email';

VulcanEmail.templates.wrapper = Assets.getText('path/to/template/wrapper.handlebars');
});
```

## Sending Emails

Finally, you can send emails with `VulcanEmail.buildAndSend(options)`. The function takes an `options` object with the following fields:

- `to`: an array of addresses to which you want to send the current email.
- `emailName`: the name of the email as registered using `VulcanEmail.addEmails`.
- `variables`: the variables to pass to the email's GraphQL `query`.

The template, subject, query, etc. will all be derived from the `emailName` you pass to the function. 
