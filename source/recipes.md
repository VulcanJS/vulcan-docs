---
title: Recipes
---

Short code snippets teaching you how to do various things in Vulcan.

## Using Router Hooks

The router exposes an `onUpdate` hook that triggers every time the route changes (including when the initial route is triggered).

Here's how you'd use it in conjunction with Apollo's [imperative API](http://dev.apollodata.com/core/read-and-write.html) to check if the current user has completed their profile:

```js
import { addCallback, getFragment } from 'meteor/vulcan:core';
import Users from 'meteor/vulcan:users';
import gql from 'graphql-tag';

function checkProfileOnUpdate (unusedItem, store, apolloClient) {
  const query = gql`
    query getCurrentUser {
      currentUser {
        ...UsersCurrent
      }
    }
    ${getFragment('UsersCurrent')}
  `

  const currentUser = apolloClient.readQuery({query}).currentUser;

  if (currentUser && !Users.hasCompletedProfile(currentUser)) {
    alert(`Current user hasn't completed their profile!`)
  }
}

addCallback('router.onUpdate', checkProfileOnUpdate);
```