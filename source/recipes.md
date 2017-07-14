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

## Load a Random Post

On the server:

```js
const randomResolver = {
  Query: {
    postsRandom(root, args, context) {
      const postCount = context.Posts.find({status: 2}).count();
      return context.Posts.find({status: 2}, {limit: 1, skip: _.random(postCount)}).fetch()[0];
    }
  }
};

addGraphQLResolvers(randomResolver);
addGraphQLQuery(`postsRandom: Post`);
```

On the client:

```js
const withRandomPost = graphql(gql`
  query postsRandom {
    postsRandom {
      ...PostsPage
    }
  }
  ${getFragment('PostsPage')}
`);

export default withRandomPost(PostsItem);
```