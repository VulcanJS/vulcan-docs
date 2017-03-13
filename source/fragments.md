---
title: Fragments
---


A fragment is a piece of schema, usually used to define what data you want to query for. 

### Registering Fragments

You can register fragments by passing the fragment string to `registerFragment`:

```js
import { registerFragment } from 'meteor/nova:core';

registerFragment(`
  fragment PostsList on Post {
    _id
    title
    url
    slug
  }
`);
```

### Using Fragments

You can get a fragment with:

```js
import { getFragment } from 'meteor/nova:lib';

getFragment('PostsList');
```

### Sub-Fragments

When registering a fragment, you'll often want to use sub-fragments to avoid repeating frequently used properties. For example, the `PostsList` fragment uses the `UserMinimumInfo` fragment:

```js
registerFragment(`
  fragment PostsList on Post {
    # nova:posts
    _id
    title
    url
    slug
    user {
      ...UsersMinimumInfo
    }
  }
`);
```

Note that in “regular” Apollo code, you need to include any sub-fragment used by a fragment [as tagged template literal](http://dev.apollodata.com/react/fragments.html#reusing-fragments), but Nova takes care of this for you (provided you've previously registered any sub-fragment using `registerFragment`).

### Extending Fragments

You often need to add one or more properties to a fragment without modifying its existing properties. You can do this with `extendFragment`:

```js
import { extendFragment } from 'meteor/nova:lib';

extendFragment('PostsList', `
  color # new custom property!
`);
```

This is the same as registering the entire `PostsList` fragment with `color` tacked on at the end. 

### Replacing Fragments

To replace a fragment completely, you can just register it again under the same name. 

```js
import { registerFragment } from 'meteor/nova:lib';

registerFragment(`
  fragment UsersMinimumInfo on User {
    _id
    slug
    username
    # displayName # remove this
    # emailHash # and this
    mySuperCustomProperty # but add this
  }
`);
```

Note that you can replace both “regular” fragments and sub-fragments.

#### Alternative Approach

You can use standard Apollo fragments at any point in your Nova app (passing them as `fragment` instead of `fragmentName`), but be aware that you will lose the ability to extend and replace fragments. You will also need to  [manually specify](http://dev.apollodata.com/react/fragments.html#reusing-fragments) sub-fragments. 