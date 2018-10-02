---
title: Settings
---

You can configure the following settings in your `settings.json` file.

## Debugging

**If you've got the [debug package](/debug.html) enabled, a settings debugging UI is available at [http://0.0.0.0:3000/debug/settings](http://0.0.0.0:3000/debug/settings).**

## Overview

You can call Vulcan.showSettings() in your meteor shell to get a quick overview of all settings, or go to `/settings` if you have the `vulcan:debug` package enabled. 

### Public Settings

These settings are defined on the `public` property, meaning they will be shared with the client. 

#### Global Settings

| Setting | Example | Description |
| --- | --- | --- |
| title | "My Vulcan Site" | Your site's title |
| siteUrl | "http://mysite.com" | Your site's main URL |
| tagline | "The best site ever!" | Your site's tagline or description |
| language | "en" | Your site's language |

#### Social Settings

| Setting | Example | Description |
| --- | --- | --- |
| twitterAccount | "TelescopeApp" | Your main twitter account (without the "@")|
| facebookPage | "https://facebook.com/TelescopeApp" | Your Facebook page URL|
| googleAnalyticsId | "UA-123456-9" | Your Google Analytics code |

### Server Settings

These settings are defined at the root of the settings object, (or optionally on the `private` property), and kept on the server.

#### Global Settings

| Setting | Example | Description |
| --- | --- | --- |
| mailUrl | "smtp://username:password@smtp.mailgun.org:587/" | The SMTP URL used by your email provider |

**MailGun** Note:
For `username` use your **Default SMTP Login**  and for `password` use **Default Password**. 
See screenshot below.

![](https://i.imgur.com/pRwaQzX.png)

#### OAuth Settings

You can use the settings file to store your oAuth configurations:

```js
"oAuth": {
  "twitter": {
    "consumerKey": "foo",
    "secret": "bar"
  },
  "facebook": {
    "appId": "foo",
    "secret": "bar"
  }
}
```

## registerSetting

You can optionally register a new setting with `registerSetting(name, defaultValue, description)`. This is optional, but it will let your setting be displayed on the `/settings` route and when calling `Vulcan.showSettings()`. 

Note that `registerSetting` also both works with nested settings objects (e.g. `registerSetting('newsletter.frequency', [1,3,5], 'Newsletter frequency')`).

## getSetting

To retrieve a setting, just call `getSetting(settingName)`. If no setting has been assigned in `settings.json`, the default value assigned through `registerSetting` will be used (if it exists).

Just like `registerSetting`, `getSetting` also works with nested settings objects (e.g. `getSetting('newsletter.frequency')`).
