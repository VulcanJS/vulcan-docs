---
title: Settings
---

You can configure the following settings in your `settings.json` file.

## Public Settings

These settings are defined on the `public` property, meaning they will be shared with the client. 

### Global Settings

| Setting | Example | Description |
| --- | --- | --- |
| title | "My Nova Site" | Your site's title |
| siteUrl | "http://mysite.com" | Your site's main URL |
| tagline | "The best site ever!" | Your site's tagline or description |
| language | "en" | Your site's language |

### Content Settings

| Setting | Example | Description |
| --- | --- | --- |
| enableNotifications | true | Whether to enable email notifications |
| postInterval | 20 | How long, in seconds, to make users wait between submitting posts |
| commentInterval | 20 | How long, in seconds, to make users wait between commenting |
| maxPostsPerDay | 10 | How many posts a user can submit per day |
| RSSLinksPointTo | "link"/"page" | Whether to point RSS links to the linked site or back to your own site |
| postsPerPage | 10 | How many posts to display per page |
| thumbnailWidth | 800 | Posts thumbnails width |
| thumbnailHeight | 600 | Posts thumbnails height |
| scoreUpdateInterval | 30 | How often, in seconds, to update posts scores. Set to `0` to disable score updating |

### Social Settings

| Setting | Example | Description |
| --- | --- | --- |
| twitterAccount | "TelescopeApp" | Your main twitter account (without the "@")|
| facebookPage | "https://facebook.com/TelescopeApp" | Your Facebook page URL|
| googleAnalyticsId | "UA-123456-9" | Your Google Analytics code |


### Newsletter Settings

| Setting | Example | Description |
| --- | --- | --- |
| enableNewsletter | false | Enable the automated newsletter |
| enableNewsletterInDev | false | Enable the automated newsletter while in development mode |
| newsletterFrequency | [1,2,3,4,5,6,7] | The days on which to send the newsletter (1 = Monday, 2 = Tuesday, etc.) |
| newsletterTime | "07:55" | When to send out the newsletter
| autoSubscribe | false | Whether new users should be automatically subscribed to the newsletter |

## Server Settings

These settings are defined on the `private` property, and kept on the server.

### Global Settings

| Setting | Example | Description |
| --- | --- | --- |
| mailUrl | "smtp://username:password@smtp.mailgun.org:587/" | The SMTP URL used by your email provider |

### OAuth Settings

You can use the settings file to store your oAuth configurations:

```js
"oAuth": {
  "twitter": {
    "consumerKey": "foo",
    "secret": "bar"
  },
  "facebook": {
    "clientId": "foo",
    "secret": "bar"
  }
}
```

### Categories

If you define categories in your settings file, they will be automatically imported into your database at app launch:

```js
  "categories": [
    {"name": "Android"},
    {"name": "iOS"},
    {"name": "Mac"},
    {"name": "PC"},
  ]
```
