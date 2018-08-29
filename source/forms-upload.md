---
title: Upload Package
---

This is a Vulcan package extending `vulcan:forms` to support form components for uploading one or more images. 

![Screenshot](https://res.cloudinary.com/xavcz/image/upload/v1471534203/Capture_d_e%CC%81cran_2016-08-17_14.22.14_ehwv0d.png)

Note: although this package only supports Cloudinary currently, PRs to support additional providers (S3, etc.) are warmly encouraged. 

## Dependencies

This package depends on the awesome `react-dropzone` ([repo](https://github.com/okonet/react-dropzone)), you need to install the dependency:
```
npm install react-dropzone isomorphic-fetch
```

## With Cloudinary 

### Setup

Create a [Cloudinary account](https://cloudinary.com) if you don't have one.

The upload to Cloudinary relies on **unsigned upload**:

> Unsigned upload is an option for performing upload directly from a browser or mobile application with no authentication signature, and without going through your servers at all. However, for security reasons, not all upload parameters can be specified directly when performing unsigned upload calls.

Unsigned upload options are controlled by [an upload preset](http://cloudinary.com/documentation/upload_images#upload_presets), so in order to use this feature you first need to enable unsigned uploading for your Cloudinary account from the [Upload Settings](https://cloudinary.com/console/settings/upload) page.

When creating your **preset**, you can define image transformations. I recommend to set something like 200px width & height, fill mode and auto quality. Once created, you will get a preset id.

It may look like this:

![Screenshot-Cloudinary](https://res.cloudinary.com/xavcz/image/upload/v1471534183/Capture_d_e%CC%81cran_2016-08-18_17.07.52_tr9uoh.png)

Make sure that your **preset** is named the same between Cloudinary and the one you define in the schema. Otherwise, the upload will request will fail. 

### Settings

Edit your `settings.json` and add inside the `public: { ... }` block the following entries with your own credentials:

```json
"public": {
  "cloudinary": {
    "cloudName": "YOUR_APP_NAME",
  }
}
```

### Specifying a Preset

If you'd like to specify a Cloudinary preset to be used to resize, convert, etc. your images, you can do so in the properties of the **form field**:

```js
  photos: {
    label: 'Photos',
    type: Array,
    optional: false,
    canRead: ['guests'],
    canCreate: ['members'],
    canUpdate: ['members'],
    control: FormsUpload, // use the FormsUpload form component
    form: {
      options: {
        preset: 'myCloudinaryPreset'
      },
    },
  },
```

### Adding images to Posts (as an example)
If you want to add images to a certain collection. Add the custom field, make sure to add this in your fragments.js file:

``` extendFragment('PostsList', `
    image
  `);

 extendFragment('PostsPage', `
    image
  `);```

 You can now use the uploaded image anywhere in your Vulcan app, using this:
```<img src={post.image} />```
which will look for the image field from your post.
