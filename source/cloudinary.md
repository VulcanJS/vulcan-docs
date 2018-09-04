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

## makeCloudinary

You can enable Cloudinary caching on a collection using the `makeCloudinary` function:

```js
import { Posts } from 'meteor/example-forum';
import { makeCloudinary } from 'meteor/vulcan:cloudinary';

makeCloudinary({collection: Posts, fieldName: 'thumbnailUrl'});
```

The `collection` option indicates which collection you want to use Cloudinary with, while the `fieldName` option indicates which field you want to cache (which should contain an image URL). 

Note that at this time, you can only cache a single image field per collection. 

## Custom Fields

`makeCloudinary` will add the following custom fields to store image data:

```js
[
  {
    fieldName: 'cloudinaryId',
    fieldSchema: {
      type: String,
      optional: true,
      canRead: ['guests'],
    }
  },
  {
    fieldName: 'cloudinaryUrls',
    fieldSchema: {
      type: Array,
      optional: true,
      canRead: ['guests'],
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
]
```

In addition, the `cloudinaryUrl` GraphQL-only field is also available as a shortcut to get a specific image format's URL. 

It optionally takes a `format` argument when used inside a GraphQL query or fragment, for example:

```
cloudinaryUrl(format: "small")
```

## Callbacks

`makeCloudinary` will also add two callbacks on `collection.new.sync` and `collection.edit.sync` to cache your images. 