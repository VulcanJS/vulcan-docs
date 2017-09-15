---
title: Cloudinary
---

This package is used to **cache** a image already hosted online using [Cloudinary](http://cloudinary.com). To *upload* a local image using Cloudinary check out the [Forms Upload](/forms-upload.html) package instead. 

## Config

In your **private** settings:

```
'cloudinary': {
  'apiKey': '123foo',
  'apiSecret': '456bar'
},
```

## Custom Fields

You can add the following custom fields to store image data:

```js
Posts.addField([
  {
    fieldName: 'cloudinaryId',
    fieldSchema: {
      type: String,
      optional: true,
      viewableBy: ['guests'],
    }
  },
  {
    fieldName: 'cloudinaryUrls',
    fieldSchema: {
      type: Array,
      optional: true,
      viewableBy: ['guests'],
    }
  },
  {
    fieldName: 'cloudinaryUrls.$',
    fieldSchema: {
      type: Object,
      blackbox: true,
      optional: true
    }
  }
]);
```

## Callbacks

A common way to make use of the package is through callbacks:

```js
import { getSetting, addCallback } from 'meteor/vulcan:core';
import { CloudinaryUtils } from 'meteor/vulcan:cloudinary';

const cloudinarySettings = getSetting('cloudinary');

// post submit callback
function cachePostThumbnailOnSubmit (post) {
  if (cloudinarySettings) {
    if (post.thumbnailUrl) {

      const data = CloudinaryUtils.uploadImage(post.thumbnailUrl);
      if (data) {
        post.cloudinaryId = data.cloudinaryId;
        post.cloudinaryUrls = data.urls;
      }

    }
  }
  return post;
}
addCallback('posts.new.sync', cachePostThumbnailOnSubmit);

function cachePostThumbnailOnEdit (modifier, oldPost) {
  if (cloudinarySettings) {
    if (modifier.$set.thumbnailUrl && modifier.$set.thumbnailUrl !== oldPost.thumbnailUrl) {
      const data = CloudinaryUtils.uploadImage(modifier.$set.thumbnailUrl);
      modifier.$set.cloudinaryId = data.cloudinaryId;
      modifier.$set.cloudinaryUrls = data.urls;
    }
  }
  return modifier;
}
addCallback('posts.edit.sync', cachePostThumbnailOnEdit);
```