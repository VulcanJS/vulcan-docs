---
title: Users
---

Vulcan uses the standard [Meteor user accounts system](https://guide.meteor.com/accounts.html), along with the [`std:accounts-ui`](https://github.com/studiointeract/accounts-ui) package for the front-end. 

### Preloaded Properties

A number of properties (such as a user's `_id`, `email`, etc.) are preloaded for the current user when the app loads. These properties are controlled by the `UsersCurrent` fragment:

```
fragment UsersCurrent on User {
  _id
  username
  createdAt
  isAdmin
  displayName
  email
  emailHash
  slug
  groups
}
```

You can extend or replace it just like any other [fragment](fragments.html).