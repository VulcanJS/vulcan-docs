---
title: Terms & Parameters
---

## Overview

When querying for data, you can perform two basic operations:

* **Filtering** which subset of data to fetch from the database.
* **Sorting** the resulting subset of data.

In Mongo, this is done through the **selector** and **options** objects that are passed to `Collection.find(selector, options)`.

While you could hard-code said selector and options on the server and always return the same subset of data to the client, this is not very practical if you want your users to be able to search, filter, sort, or manipulate the data in any way.

You _could_ simply let users specify their own selectors and options, but that would open the door to users accessing _any_ document in your database, and is generally regarded as an anti-pattern.

Instead, Vulcan adopts a two-tiered approach: first, the user defines query **terms** that specify the data they want. Then, these terms go through a set of successive callbacks that “translate” them into a Mongo-compatible `{selector, options}` object known as the **parameters** object (in the process catching any potential security issues).

![/images/terms-parameters.svg](/images/terms-parameters.svg)

To give a practical example, the `vulcan:posts` package translates the following terms:

```js
{
  view: 'top';
}
```

Into these parameters:

```js
{
  selector: {
    status: Posts.config.STATUS_APPROVED,
    isFuture: {$ne: true}
  }
  options: {
    sort: {
      sticky: -1,
      score: -1
    }
  }
}
```

This two-tiered strategy ensures both that the user doesn't need to spell out every rule (“only select approved posts”) every single time, and also that security risks don't creep in.

#### Alternative Approach

The generic `withList` component is hard-coded to send a `terms` argument, but if you're not using `withList` then you can send any argument you want, and bypass the terms/parameters system altogether.

### Parameter Callbacks

Every Vulcan collection has its own `collection.parameters` callback hook which you can use to add additional parameter transformations. For example, this is how the `framework-demo` implements a sort by `createdAt` on the `Movies` collection:

```js
import { addCallback } from 'meteor/vulcan:core';

function sortByCreatedAt(parameters, terms) {
  return {
    selector: parameters.selector,
    options: { ...parameters.options, sort: { createdAt: -1 } }
  };
}

addCallback('movies.parameters', sortByCreatedAt);
```

Note that each iteration of the callback takes in the `parameters` and the original `terms` object received from the client, and should return a new augmented `parameters` object.

### Using Views

You've seen above how to manually create a `sortByCreatedAt` callback, but Vulcan also provides a built-in `view` shortcut you can use to achieve the same thing more easily (available on all collections created via `createCollection`). By defining a named view, you can then reference all the view's properties at once by specifying the `view: myViewName` option in your `terms`.

For example, this is how the `best` view is defined:

```js
import { Posts } from 'meteor/vulcan:posts';

Posts.addView('best', terms => ({
  options: {
    sort: { sticky: -1, baseScore: -1 }
  }
}));
```

Instead of creating a dedicated `sortByStickyAndBaseScore` callback, you can simply specify `view: 'best'` in your `terms`.

If you'd like to specify some default options for all your views, you can also use `collection.addDefaultView`.

In this case, we want all `Posts` views to only show posts that are approved and not scheduled in the future (unless explicitly specified).

```js
Posts.addDefaultView(terms => ({
  selector: {
    status: Posts.config.STATUS_APPROVED,
    isFuture: { $ne: true } // match both false and undefined
  }
}));
```

Note that views work by extending one another. In other words, the default view is extended by the view specified by the view option, which is is then extended by any additional parameter callbacks.

#### Alternative Approach

Using views is completely optional. You can also use `collection.parameters` callbacks instead to achieve the same thing with a little bit more code.
