---
title: Instagram Example
---

Here's a [video walkthrough](https://www.youtube.com/watch?v=qibyA_ReqEQ) of [the code](https://github.com/VulcanJS/Vulcan-Starter/tree/master/packages/example-instagram) for the `example-instagram` package.

### Note: Code Changes

Unlike the video, the actual package now relies on [default resolvers](/resolvers.html#Default-Resolvers) and [default mutations](/mutations.html#Default-Mutations) to simplify the code, meaning the `resolvers.js` and `mutations.js` files were removed. 

Also, resolvers for the `userId` and `commentsCount` fields are now specified directly in the schema instead of in the `resolvers.js` file.

### Image Upload

This example uses [the `vulcan:forms-upload` package](forms-upload.html), which  uploads images to [Cloudinary](http://cloudinary.com).
