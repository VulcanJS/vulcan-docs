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

The generic `withMulti` component is hard-coded to pass a `terms` argument to a collection's GraphQL resolvers, but if you're not using `withMulti` then you can send any argument you want, and bypass the terms/parameters system altogether.

## Parameter Callbacks

Every Vulcan collection has its own `collection.parameters` callback hook which you can use to add additional parameter transformations. 

(In addition, collections also have two distinct `collection.parameters.client` and `collection.parameters.server` hooks that only run in their respective environments, although the regular `collection.parameters` hook is usually enough.)

### Simple Callbacks 

For example, this is how you could implement a sort by `createdAt` on the `Movies` collection:

```js
import { addCallback, Utils } from 'meteor/vulcan:core';

function addSortByCreatedAt(parameters, terms) {
  Utils.deepExtend(parameters, { options: { sort: { createdAt: -1 } } });
}

addCallback('movies.parameters', addSortByCreatedAt);
```

Note that each iteration of the callback takes in the `parameters` and the original `terms` object received from the client (which you may or may not use), and should return a new augmented `parameters` object. 

Since multiple callbacks can run at the same time, it's generally a good idea to extend the `parameters` argument rather than overwrite it to avoid cancelling out the effects of previous parameter callbacks. 

### Advanced Callbacks 

You've seen above how to implement a “static” callback that always returns the same thing. But because callbacks receive `terms` as their second argument, they can also implement more advanced logic. For example, here's how you could add a callback to filter out products by `minPrice` and `maxPrice`: 

```js
function addMinPriceMaxPriceParameters (parameters, terms) {

  let price = {};

  if (terms.minPrice) {
    price.$gte = parseInt(terms.minPrice);
  }

  if (terms.maxPrice) {
   price.$lte = parseInt(terms.maxPrice);
  }

  if (!_.isEmpty(price)) {
    parameters.selector.price = price;
  }

  return parameters;
}
addCallback('products.parameters', addMinPriceMaxPriceParameters);
```

Note that we've written our callback in a way that supports having `minPrice`, `maxPrice`, or both be undefined. After all, since `terms` comes from the client we don't have complete control over it and have to be ready to handle any possible permutation. 

## Existing Parameters 

While you can manually create callbacks to deal with any arbitrary property, Vulcan can also handle a few built-in `terms` properties out of the box (available on all collections created via `createCollection`).

### Limit

You can set `terms.limit` to limit the number of results returned. 

### Query

You can set `terms.query` to perform a search, assuming you've set some of your collection fields to `searchable: true`. 

### View

Vulcan's built-in `view` property lets you define shortcuts that aggregate multiple parameters in a single function. 

Let's go back to our Movies example's `createdAt` sort:

```js
import { addCallback, Utils } from 'meteor/vulcan:core';

function addSortByCreatedAt(parameters, terms) {
  Utils.deepExtend(parameters, { options: { sort: { createdAt: -1 } } });
}

addCallback('movies.parameters', addSortByCreatedAt);
```

The issue here is that our `{ createdAt: -1 }` sort will *always* be applied whenever we load data from the Movies collection. But what if we also wanted the ability to sometimes sort movies by name as well?

One solution would be to make our callback depend on `terms.sort`. But this will quickly devolve into a mess of `if` or `switch` statements as you try to handle every possible value. 

Here is how you can achieve this by creating two views instead: 

```js
// import Movies collection
import { Movies } from './collection.js';

Movies.addView('new', terms => ({
  options: {
    sort: { createdAt: -1 }
  }
}));

Movies.addView('alphabetical', terms => ({
  options: {
    sort: { name: 1 }
  }
}));
```

You can then pick a view by setting `terms.view` to either `new` or `alphabetical`. Another big advantage of views is that they can return complex objects. For example, here's how you'd create a view to show a user's posts:

```js
Posts.addView('userPosts', terms => ({
  selector: {
    userId: terms.userId,
    status: Posts.config.STATUS_APPROVED,
    isFuture: {$ne: true}
  },
  options: {
    limit: 5,
    sort: {
      postedAt: -1
    }
  }
}));
```

#### Default View

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