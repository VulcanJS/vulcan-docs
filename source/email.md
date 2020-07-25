---
title: Email
---

Vulcan includes a simple-yet-powerful email-sending system.
## Debugging

**If you've got the [debug package](/debug.html) enabled, an email debugging UI is available at [http://localhost:3000/debug/emails](http://localhost:3000/debug/emails).**

## Overview

Setting up emails in Vulcan is a three-step process. 

First, you'll need to load and register an email's **template**. This is the Handlebars file that controls the actual markup that will be sent out. 

Then, you can **define the email itself**. Defining a new email means not only specifying which previously registered template to use, but also how to generate the subject line, figure out who to send out the email to, and how to query for the data used to populate that email. 

Finally, you can **send the email**. Because so much of the work has happened upfront this is usually just a matter of specifying which email to send, and optionally including some variables to customize it.

## Setup

In order to send emails, you'll need to set up an email provider. You can do so by providing an SMTP URL (e.g. `smtp://username:password@smtp.mailgun.org:587/`) in your `settings.json` file under the `mailUrl` property.

Note: if your username contains an `@` make sure to replace it with `%40` to escape it. 

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

## Registering Emails

Just like components, emails need to be centrally registered with Vulcan.

```js
import VulcanEmail from 'meteor/vulcan:email';

VulcanEmail.addEmails({

  test: {
    template: 'test',
    testPath: '/email/test',
    data() {
      return {date: new Date()};
    },
    subject(data) {
      return 'This is a test';
    },
  }

});
```

### Email Properties

The `VulcanEmail.addEmails` function takes an object that itself contains individual objects for each email. Each email object can have the following properties:

#### `template`

The name of the template used to display the email (see below).

#### `to(data)`

Either a function that takes in the email's `data` as argument and returns the recipient's email, or a string. 

#### `subject(data)`

Either a function that takes in the email's `data` as argument and returns the subject, or a string. 

#### `data(variables)`

A function that returns an object used to populate the email. It will either be passed a `variables` object by `VulcanEmail.buildAndSend`, or use `testVariables`.

#### `query`

A string containing a GraphQL query used to populate the email see below). If both `data` and `query` are provided, their results will be merged and passed on to the email.

Similarly to `data()`, the query will either use the `variables` object passed by `VulcanEmail.buildAndSend`, or use `testVariables`.

### Test Properties

Additionally, you can define some test properties to make previewing and testing the email easier. These properties will be used when previewing the email from the email debug dashboard.

#### `testPath`

A path for the route at which the email will be available to preview (when using `vulcan:debug`). If the path contains paramerters (such as `/email/post-approved/:postId?`), these parameters will be passed as variables to the `data()` function as well as the `query`.

#### `testVariables`

An object containing default variables passed to `query()` when testing the email. If not specified, a blank object will be used. 

This is useful to simulate an actual email sending function, since said function would usually pass variables to the 

You can also pass a `testVariables(params)` function instead, in which case the `params` object will contain any route params you've defined. For example, if you've set the test route for an email to be `/email/post-approved/:postId?` then visit `/email/post-approved/foo123`, then `params` will be equal to `{ postId: 'foo123' }`. 

### Full Example

```js
import VulcanEmail from 'meteor/vulcan:email';
import get from 'lodash/get';

const postsQuery = `
  query PostsSingleQuery($input: SinglePostInput!){
    siteData {
      title
      url
    }
    post(input: $input){
      result{

        title
        url
        htmlBody

        scheduledAtFormatted
        paidAtFormatted
        sponsorshipPriceFormatted

        user{
          pageUrl
          displayName
        }
      }
    }
  }
`;

VulcanEmail.addEmails({
  postApproved: {
    template: "postApproved",
    to(data) {
      return get(data, 'data.post.result.user.email', 'foo@example.com')
    },
    subject(data) {
      const dummyPost = { title: "[title]" };
      const post = get(data, 'data.post.result', dummyPost)
      return "Your post “" + post.title + "” has been approved";
    },
    query: postsQuery,
    testPath: "/email/post-approved/:postId?",
    testVariables({ postId }) {
      return { input: { id: postId } };
    },
  },
}
```

As you can see, just like you can use GraphQL to query for data on the client and populate your React component's contents, you can also use it to query for data on the server and populate your Handlebars templates.

A few things to note:

- Queries can accept variables, which will be passed by `VulcanEmail.buildAndSend()` (see below); or `testVariables` when testing.
- A single query can query resolvers for entirely different collections. This lets you mix-and-match different data types in the same email. 
- If the default resolvers (`post`, `posts`, etc.) don't return the data you need for a specific email, don't forget you can also create your own custom resolvers with `addGraphQLResolvers` and `addGraphQLQuery`.

## Sending Emails

Finally, you can send emails with `VulcanEmail.buildAndSend(options)`. The function takes an `options` object with the following fields:

- `emailName`: the name of the email as registered using `VulcanEmail.addEmails`.
- `variables`: (optional) the variables to pass to the email's GraphQL `query` and the `data` function.

Additionally, you can also specify `to` and `subject` properties, otherwise they will be derived from the email definition associated with the `emailName` you specified.
