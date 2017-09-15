---
title: Embed
---

This package will get metadata (title, image, etc.) for a given URL and use it to prefill other form fields. 

## Config

You'll need to configure the `embedProvider` setting as well as at least one provider (unless you're using the `builtin` method).

In your **private** settings:

```
"embedProvider": "builtin",
"embedly": {
  "apiKey": "123foo"
},
```

Possible providers include:

- `builtin`: a built-in function.
- `embedly`: [Embedly](http://embed.ly/)
- `embedapi`: [EmbedAPI](http://embedapi.com/)

## Form Component

Inside forms, you can use the `EmbedURL` component to display a special URL field that will prefill other fields based on its own value whenever that value changes. 

Prefillable fields include:

- `title`: the title of the linked URL.
- `body`: the description of the URL.
- `thumbnailUrl`: a thumbnail image of the URL.
- `media`: a media embed, for example a YouTube video embed.
- `sourceName`: the source name, for example the name of a YouTube channel.
- `sourceUrl`: the source URL, for example the URL of a YouTube channel.

Here's an example of including these fields as custom fields on the `Posts` collection (which already includes its own `title` and `body`):

```js
import Posts from '../posts/index.js';
import { EmbedlyURL } from 'meteor/vulcan:embed';

Posts.addField([
  {
    fieldName: 'url',
    fieldSchema: {
      control: EmbedlyURL, // we are just extending the field url, not replacing it
    }
  },
  {
    fieldName: 'thumbnailUrl',
    fieldSchema: {
      type: String,
      optional: true,
      insertableBy: ['members'],
      editableBy: ['members'],
      viewableBy: ['guests'],
      hidden: true
    }
  },
  {
    fieldName: 'media',
    fieldSchema: {
      type: Object,
      optional: true,
      blackbox: true,
      viewableBy: ['guests'],
    }
  },
  {
    fieldName: 'sourceName',
    fieldSchema: {
      type: String,
      optional: true,
      viewableBy: ['guests'],
    }
  },
  {
    fieldName: 'sourceUrl',
    fieldSchema: {
      type: String,
      optional: true,
      viewableBy: ['guests'],
    }
  }
]);
```

## Mutations

The package exposes the following GraphQL mutation:

```
getEmbedData(url: String) : JSON
```

## Custom Providers

To add a new provider, follow this structure to implement the `Embed.myProvider.getData` function:

```js
import Embed from 'meteor/vulcan:embed';

Embed.myProvider = {

  getData(url) {

    const metadata = getMyData(url);
    
    return metadata; // title, description, thumbnailUrl, etc.
  }

}
```

Then set the `embedProvider` setting to `myProvider`. 