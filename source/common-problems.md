---
title: Common Problems
---

Sometimes, things just go wrong. Here are a few tips to debug your Vulcan app.

## Common Problems

### “Load More” Not Working

If your app initially loads fine but “Load More” doesn't work, this most likely indicates that while server-side rendering is working, data loading isn't.

The most common cause is a mismatch between your domain and the URL currently being used as your GraphQL endpoint, which is based on the `ROOT_URL` environment variable.

To double-check which GraphQL endpoint URL is being used, just use your Devtools Network tab in your browser and inspect any failed requests.

## Debugging

### Finding Errors

Locating the source of an issue can be trickier than you think.

First, you'll find most client-side errors through the browser console (`cmd/ctrl + opt + j` in Chrome).

If nothing is showing up in the browser console or maybe your app isn't loading at all, make sure you also check the server logs. You'll find them in the terminal window currently running your app locally, or with `mup logs -f` if you've deployed using Meteor Up.

It's also possible that you've encountered a GraphQL error, which won't be shown in either places. One way to track this down is to open your browser's Devtools Network tab and look for any red `graphql` requests. If you see any failed requests, check their Response tab to view the error message.

You can also access a query's error message through the Redux or Apollo devtools. In the Redux devtools, click the State tab, drill down to the affected query, and check its `networkError` and `graphQLErrors` properties.

### GraphQL Schema Issues

Although Vulcan includes features for generating your GraphQL schema automatically from your JSON schema, sometimes you'll run into issues and need to take a look under the hood.

You can review your final generated schema by opening the Meteor shell with command `meteor shell` and then type:

```
Vulcan.getGraphQLSchema()
```

You'll probably want to paste the output in a text editor and replace `\n` with actual line breaks for better formatting.

### SSR Warnings

Another common type of problem is server-side rendering (SSR) warnings:

> React attempted to reuse markup in a container but the checksum was invalid. This generally means that you are using server rendering and the markup generated on the server was not what the client was expecting. React injected new markup to compensate which works but you have lost many of the benefits of server rendering. Instead, figure out why the markup being generated is different on the client or server:

This means that the HTML content you've rendered on the server doesn't match up with the markup React has generated on the client. For example, maybe the server thinks the current user is logged out, while the client knows they're logged in. Or maybe the server thinks it's Tuesday, while the client thinks it's Wednesday.

Whatever the cause, these warnings should not break your app, but they might slow it down a bit.

### Data Loading

If your data doesn't seem to be loading, this could be due to a few reasons:

* **Component issue:** the client is loading your data, but your component is not displaying it. Check that the data is available in the Redux store, and check the value of your component's `props` using the React devtools.
* **Resolver issue:** if your client isn't loading any data at all, it might be because the resolver isn't returning any. In this scenario, `console.log` will be your best friend.
* **Arguments issue:** maybe the resolver itself is fine, but it's just not receiving the right arguments from the client. Again, adding `console.log` statements to the resolver will usually point you in the right direction.
* **Database issue:** sometimes your code works, but the data you expect just isn't present in your database. You can check this by taking the query being run by the resolver and running it manually in Mongo directly (using `meteor mongo` command).

### Mutations

Mutations can fail for a number of reasons:

* **Component issue:** is your mutation even getting called? Make sure you double-check those event handlers.
* **Resolver issue:** just like queries, mutations can also have resolver problems.
* **Permissions issue:** it's also possible that the mutation is failing because you don't have the proper rights.

### Data Updating

If your data isn't updating properly after a mutation, this can be due to a fragment mismatch. Make sure that `create` mutations use the same fragment as the query they're updating (`update` mutations on the other hand can safely return partial fragments).
