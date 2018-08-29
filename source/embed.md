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

```
"embedProvider": "embedAPI",
"embedAPI": {
   "apiKey": "123foo"
}
```

Possible providers include:

- `builtin`: a built-in function.
- `embedly`: [Embedly](http://embed.ly/)
- `embedAPI`: [EmbedAPI](http://embedapi.com/)

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
      canCreate: ['members'],
      canUpdate: ['members'],
      canRead: ['guests'],
      hidden: true
    }
  },
  {
    fieldName: 'media',
    fieldSchema: {
      type: Object,
      optional: true,
      blackbox: true,
      canRead: ['guests'],
    }
  },
  {
    fieldName: 'sourceName',
    fieldSchema: {
      type: String,
      optional: true,
      canRead: ['guests'],
    }
  },
  {
    fieldName: 'sourceUrl',
    fieldSchema: {
      type: String,
      optional: true,
      canRead: ['guests'],
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
import { Embed } from 'meteor/vulcan:embed';

Embed.myProvider = {

  getData(url) {

    const metadata = getMyData(url);
    
    return metadata; // title, description, thumbnailUrl, etc.
  }

}
```

Then set the `embedProvider` setting to `myProvider`. 

## Callbacks

You can use callbacks to generate embedded media server-side: 

```js
/*

Callbacks to add media/thumbnail after submit and on edit

*/

import { addCallback, getSetting, registerSetting } from 'meteor/vulcan:core';
import { Embed } from 'meteor/vulcan:embed';

const embedProvider = getSetting('embedProvider');

// For security reason, we make the media property non-modifiable by the client and
// we use a separate server-side API call to set it (and the thumbnail object if it hasn't already been set)

// Async variant that directly modifies the post object with update()
function AddMediaAfterSubmit (post) {

  if(post.url){

    const data = Embed[embedProvider].getData(post.url);

    if (data) {

      // only add a thumbnailUrl if there isn't one already
      if (!post.thumbnailUrl && data.thumbnailUrl) {
        post.thumbnailUrl = data.thumbnailUrl;
      }

      // add media if necessary
      if (data.media && data.media.html) {
        post.media = data.media;
      }

      // add source name & url if they exist
      if (data.sourceName && data.sourceUrl) {
        post.sourceName = data.sourceName;
        post.sourceUrl = data.sourceUrl;
      }

    }

  }
  
  return post;
}
addCallback('posts.new.sync', AddMediaAfterSubmit);

function updateMediaonUpdate (modifier, post) {
  
  const newUrl = modifier.$set.url;

  if(newUrl && newUrl !== post.url){

    const data = Embed[embedProvider].getData(newUrl);

    if(data) {

      if (data.media && data.media.html) {
        if (modifier.$unset.media) {
          delete modifier.$unset.media
        }
        modifier.$set.media = data.media;
      }

      // add source name & url if they exist
      if (data.sourceName && data.sourceUrl) {
        modifier.$set.sourceName = data.sourceName;
        modifier.$set.sourceUrl = data.sourceUrl;
      }

    }
  }
  return modifier;
}
addCallback('posts.edit.sync', updateMediaonUpdate);
```
