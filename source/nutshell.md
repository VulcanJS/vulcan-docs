---
title: Vulcan in a Nutshell
---

The best way to understand how Vulcan works is to consider its three main aspects: the role of the schema, how Vulcan reads data, and how Vulcan writes data.

## Overview

[![/images/how-vulcan-works.svg](/images/how-vulcan-works.svg)](/images/how-vulcan-works.svg)

## The Schema

At its core, a Vulcan schema is just a JavaScript object containing a list of fields such as `name`, `_id`, `createdAt`, `description`, etc. describing a **type of document** (a movie, a post, a photo, a review, and so on).

The schema is what defines how a [Collection](/schemas.html) (you might also be more familiar with the equivalent term “model”) behaves, and it fulfills many important functions:

1. It's used to generate your GraphQL schema, which in turn controls your app's GraphQL API.
2. It's used to [control permissions](http://docs.vulcanjs.org/groups-permissions.html).
3. It's used to [generate forms](http://docs.vulcanjs.org/forms.html).

## Reading Data

Reading data basically means getting data from your database all the way to the user's browser.

Let's assume you have a `Movies.jsx` component ready to take a `results` prop and display its contents as a list of movies.

In order to pass that prop, you'll wrap your component with the [`withMulti` higher-order component](/resolvers.html#withMulti). You just need to specify the appropriate [Collection](/schemas.html), and optionally also specify a [fragment](/fragments.html) to define which document fields to load.

The `withMulti` HoC will then query your GraphQL endpoint's corresponding [`movies` resolver](/resolvers.html#List-Resolver), which will usually have been generated automatically using Vulcan's [default resolvers](/resolvers.html#Default-Resolvers).

The `movies` resolver will optionally check the query's `terms` argument and [feed it through a set of callbacks](/terms-parameters.html) in order to generate a valid MongoDB query, whose result it will return to `withMulti` and from there back to your `Movies` component.

## Writing Data

Now let's consider the opposite operation: writing data, such as editing a movie's description.

First, you should know that the movie update form can be [automatically generated from your schema](/forms.html), meaning you don't actually need to code it or worry about hooking it up to your GraphQL API.

That form is wrapped with the [`withUpdate` HoC](/mutations.html#Higher-Order-Components), which in turn will call the `updateMovie` mutation on the server (which again can be [automatically generated from default mutations](/mutations.html#Default-Mutations)).

`updateMovie` will then call a [boilerplate mutator](/mutations.html#Boilerplate-Mutations) which will perform **validation** based on your schema, and finally modify the document inside your database.
