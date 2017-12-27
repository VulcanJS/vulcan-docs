---
title: Packages Types
---

Vulcan uses a package-based architecture, meaning that the entirety of its codebase resides in packages.

## Package Types

Vulcan makes use of many different packages types, so it might be a good idea to get them straight before we move on.

### NPM Packages

These are listed in your `package.json` file, and they are generic Node JavaScript packages that don't contain any Vulcan-specific code. 

Example: `react`

### Meteor Core Packages

In addition to NPM, Meteor also features its own package management system used in parallel. Meteor Core Packages are the packages that make up the Meteor framework itself. 

Example: `meteor-base`

### Meteor Remote Packages

Meteor can also load packages from its package server. Remote packages must always follow the `username:package` naming convention.

Example: `aldeed:collection2-core`

### Meteor Local Packages

Finally, Meteor can also load packages from the local file system. These packages can be named whatever you want, and don't need to follow the `username:package` convention. 

Example: `example-movies`, `vulcan:forms`

Note that if both a local and remote Meteor package use the same name, Meteor will use the local copy.

## Vulcan Packages

Different Vulcan packages can play different roles. So here's a quick overview of the different Vulcan package categories you'll come across. 

### Core Packages

The `vulcan:core` package contains the heart of Vulcan, and itself depends on a set of core packages:

```
vulcan:lib
vulcan:routing
vulcan:users
```

Note that since `vulcan:core` already includes these dependencies, you only need to make your own package depend on `vulcan:core`. 

### Features Packages

These optional packages provide additional features for your Vulcan app.

```
vulcan:accounts
vulcan:forms
vulcan:email
vulcan:events
```

### Forum Packages

Because Vulcan started its life as Telescope, a Hacker News-like forum app, it still includes quite a few packages that are made specifically to help you build a forum-type community app (enabled through [the `example-forum` package](example-forum.html)). 

```
vulcan:posts
vulcan:comments
vulcan:newsletter
vulcan:notifications
vulcan:getting-started
vulcan:categories
vulcan:voting
vulcan:embedly
vulcan:api
vulcan:rss
vulcan:subscribe
```

### Language Packages

These contain the language strings used throughout Vulcan (for the app core, features packages, and forum packages). You need **at least one of them** enabled. 

```
vulcan:i18n-en-us
```

### Accounts Package

Accounts package contain the back-end authentication logic used by Meteor's account system. You need **at least one of them** enabled.

```
accounts-password@1.3.4
# accounts-twitter
# accounts-facebook
```

### Custom Packages

Any other package that contains your own logic. 

Out of the box, four example custom packages are provided:

- `example-movies` contains a bare-bones forms and data loading example. 
- `example-instagram` goes a little bit further with a simple Instagram clone.
- `example-forum` includes the original Telescope forum app.
- `example-customization` shows how to customize and extend the forum example packages without having to modify their code directly. 

```
example-movies
# example-movies-full
# example-forum
# example-customization
```

## Controlling Dependencies

Generally speaking, there are two ways a package can be added to your codebase. It can be a top-level dependency, or one of your packages can itself depend on it. 

For example, the `example-movies` package depends on `vulcan:core`, so adding `example-movies` will automatically add `vulcan:core` as well. This is why `vulcan:core` is not listed in your `.meteor/packages` file. 

But although `example-movies` can use the English text strings stored in `vulcan:i18n-en-us`, it doesn't *depend* on it. This means you can (for example) swap out `vulcan:i18n-en-us` for `vulcan:i18n-fr-fr` in `.meteor/packages` to get French strings. 

A complete list of all Meteor packages currently loaded is stored in the `.meteor/versions` file. 

Note that Meteor packages may be added even if they are not located within the Vulcan root directory, by setting the `METEOR_PACKAGE_DIRS` environment variable. For example, the following commands will allow you to modify your `.profile` file to add any package located in the `packages` subdirectory of your home directory:

```
cat >> $HOME/.profile
export METEOR_PACKAGE_DIRS=$HOME/packages
```
