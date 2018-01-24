---
title: Newsletter
---


This package automatically generates and sends a newsletter for posts and comments on a predefined schedule. 

![Newsletter](http://f.cl.ly/items/0V0F351k1R1i3L1k1D0J/telescope-newsletter.png)

Every `x` days, it builds a digest consisting of the top `y` items posted in the past `x` days that haven't yet been sent out in a newsletter. It then sends the campaign, and sends you a confirmation email.

Note: this package requires using a third-party email provider ([MailChimp](http://mailchimp.com) and [Sendy](http://sendy.co) are currently supported). 

## Install

`meteor add vulcan:newsletter`

## Dependencies

#### Vulcan Dependencies

- vulcan:core
- vulcan:posts
- vulcan:comments
- vulcan:categories
- vulcan:email

#### NPM Dependencies

- [mailchimp](https://github.com/gomfunkel/node-mailchimp)
- [sendy-api](https://github.com/igord/sendy-api)

## Settings

```
"newsletter": {
  "enabled":true,
  "enabledInDev":false,
  "autoSubscribe":false,
  "frequency": [1,2,3,4,5,6,7],
  "time": "00:00",
  "provider": "mailchimp"
},
"sendy": {
  "server": "http://sendy.myapp.com/",
  "apiKey": "123foo",
  "listId": "456bar",
  "fromName": "Bruce Willis",
  "fromEmail": "hello@myapp.com",
  "replyTo": "hello@myapp.com"
},
"mailchimp": {
  "apiKey": "123foo",
  "listId": "456bar",
  "fromName": "hello@myapp.com",
  "fromEmail": "hello@myapp.com"
},
```

- `enabled`: enable/disable automated newsletter sending.
- `enabledInDev`: enable/disable newsletter while in development mode. 
- `autoSubscribe`: automatically subscribe every new user to your newsletter.
- `frequency`: which days the newsletter should go out. `1` is Monday, `2` is Tuesday, etc.
- `time`: what time the newsletter should go out.
- `provider`: the name of your newsletter provider.

#### Sendy Settings

- `server`: The URL to your self-hosted Sendy server (including trailing `/`).
- `apiKey`: Your API key.
- `listId`: List ID.
- `fromName`: "From" name.
- `fromEmail`: "From" email.
- `replyTo`: "Reply to" email.

#### MailChimp Settings

- `apiKey`: Your API key.
- `listId`: List ID.

## Test Routes

If you want to preview your email templates, you can do so at the following routes: 

- **Campaign**: [http://localhost:3000/email/newsletter](http://localhost:3000/email/newsletter)
- **Confirmation**: [http://localhost:3000/email/newsletter-confirmation](http://localhost:3000/email/newsletter-confirmation)

(Replace `http://localhost:3000` with your app's URL)

## Mutations

The package exposes the following GraphQL mutations:

- `sendNewsletter`: generate and send the next newsletter.
- `testNewsletter`: same as `sendNewsletter`, but without marking posts as sent. 
- `addUserNewsletter(userId: String)`: add a user to your subscriber list.
- `addEmailNewsletter(email: String)`: add an email to your subscriber list.
- `removeUserNewsletter(userId: String)`: remove a user from your subscriber list. 

You can call these mutations from any React component using the [`withMutation` HoC](mutations.html#Higher-Order-Components):

```
import React, { PropTypes, Component } from 'react';
import { withMutation } from 'meteor/vulcan:core';

const SendButton = ({ sendNewsletter }) => <button onClick={sendNewsletter}>Send Newsletter</button>

export default withMutation({name: 'sendNewsletter'})(SendButton);
```

## Providers

To add a provider, you can start from the `sample.js` template included in the package. 

Any new provider must implement the following methods: 

- `Newsletters.providerName.subscribe(email)`: subscribe an email to your list.
- `Newsletters.providerName.unsubscribe(email)`: unsubscribe an email from your list.
- `Newsletters.providerName.send({ title, subject, text, html, isTest})`: send a newsletter. 

## NewsletterSubscribe Component

This package also exports a `NewsletterSubscribe` custom form field component used to add a subscribe/unsubscribe button to the user account page. 

## Settings

| Setting | Example | Description |
| --- | --- | --- |
| enabled | false | Enable the automated newsletter |
| enabledInDev | false | Enable the automated newsletter while in development mode |
| frequency | [1,2,3,4,5,6,7] | The days on which to send the newsletter (1 = Monday, 2 = Tuesday, etc.) |
| time | "07:55" | When to send out the newsletter
| autoSubscribe | false | Whether new users should be automatically subscribed to the newsletter |