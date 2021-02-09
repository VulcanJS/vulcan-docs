---
title: Resolvers
---

A resolver is the function on the server that receives a GraphQL query, decides what to do with it (how to _resolve_ it), and then returns some data.

Each level of a GraphQL query (in other words, each field) can have its own custom resolver; or if no resolver exists the field will default to look for a value on its parent object.

### Implicit Resolvers

Technically speaking, every single field in the fragment has to resolve to _something_. But if for example you've specified a `Post` resolver and you're asking for a post's `title` and `createdAt` properties, GraphQL will be smart enough to look for `post.title` and `post.createdAt` without you needing to define additional resolvers for these two fields.

That being said if you wanted `createdAt` to resolve to, say, `formatDate(post.createdAt)` instead of just `post.createdAt`, then you could write a custom resolver [just for that specific field](/field-resolvers.html).

### Data Updating

As long as you use `withMulti` or `useMulti` in conjunction with `withCreate`, `withUpdate`, and `withDelete`, your lists will automatically be updated after any mutation. This includes:

* Inserting new items in lists after they're inserted.
* Removing items when they're removed.
* Reordering lists when an item is edited in a way that changes a sort.
* Removing an item after it's been edited when it doesn't match a list's filters anymore.

At this time, it's only possible to benefit from this auto-updating behavior if you're using the three built-in mutation HoCs, although making `withMulti` more flexible towards custom mutations is on the roadmap (PRs welcome!).

#### Alternative Approach

You can replace any of Vulcan's generic HoCs with your own tailor-made HoCs (whether it is for queries or mutations) using the `graphql` utility. Note that if you do so, you will need to manually [update your queries](http://dev.apollodata.com/react/cache-updates.html) after each mutation.
