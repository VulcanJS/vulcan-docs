---
title: Packages
---

Vulcan uses a package-based architecture, meaning that the entirety of its codebase resides in packages.

## Package Types

Vulcan makes use of many different packages types, so it might be a good idea to get them straight before we move on.

### NPM Packages

These are listed in your `package.json` file, and they are generic Node JavaScript packages that don't contain any Vulcan-specific code. 

Example: `react`

### Meteor Core Packages

In addition to NPM, Meteor also features its own package management system used in parallel. Meteor Core Packages are the packages that make up the Meteor framework itself. You can recognize them because they don't follow the usual `username:package` format.

Example: `meteor-base`

### Meteor Third-Party Packages

These are Meteor packages loaded from Meteor's package server. 

Example: `aldeed:collection2-core`

### Meteor Local Packages

These are Meteor packages loaded from the local filesystem. Most of Vulcan's packages, as well as your own customizations, belong to this category.

Example: `vulcan:posts`

Note that if both a local and remote Meteor package use the same name, Meteor will use the local copy. 

## Vulcan Packages

Here's a quick overview of the different Vulcan package categories you'll come across. Note that all packages are listed in your `.meteor/packages` file. 

### Core Packages

These packages make up the heart of Vulcan. You'll need **all of them** enabled. 

```
vulcan:core                       # core components and wrappers
vulcan:forms                      # auto-generated forms
vulcan:routing                    # routing and server-side rendering
vulcan:users                      # user management and permissions
```

### Features Packages

These optional packages provide specific features made to work together, most of the time using the `vulcan:posts` and `vulcan:comments` packages. These are all **disabled by default** (commented out with a `#`). 

```
# vulcan:email
# vulcan:posts
# vulcan:comments
# vulcan:newsletter
# vulcan:notifications
# vulcan:getting-started
# vulcan:categories
# vulcan:voting
# vulcan:events
# vulcan:embedly
# vulcan:api
# vulcan:rss
# vulcan:subscribe
```

As an example, the `vulcan:newsletter` package can be used to send an email newsletter of your most recent posts and comments. But of course all three packages can also be disabled. 

You can learn more about these in the [Features Packages](features-packages.html) section. 

### Theme Packages

These packages contain components that work with the default features packages (posts, comments, categories, etc.).

```
# vulcan:base-components        # default ui components
# vulcan:base-styles            # default styling
# vulcan:email-templates        # default email templates for notifications
```

These packages can contain React components, CSS styles, or Handlebars email templates. Since they work with features packages, they're also **disabled by default**. 

### Language Packages

These contain the language strings used throughout Vulcan. You need **at least one of them** enabled. 

```
vulcan:i18n-en-us               # default language translation
```

### Accounts Package

Accounts package contain the authentication logic used by Meteor's account system. You need **at least one of them** enabled.

```
accounts-password@1.3.4
# accounts-twitter
# accounts-facebook
```

### Custom Packages

Any other package that contains your own logic. 

Out of the box, three example custom packages are provided:

- `example-movies` contains a bare-bones forms and data loading example. 
- `example-movies-full` goes a little bit further.
- `example-customization` shows how to customize and extend features packages without having to modify their code directly. 

```
example-movies
# example-movies-full
# example-customization
```

Note that custom packages may be added even if they are not located within the Vulcan root directory, by setting the `METEOR_PACKAGE_DIRS` environment variable. For example, the following will allow you to add any package located in the `packages` subdirectory of your home directory:

```
cat >> $HOME/.profile
export METEOR_PACKAGE_DIRS=$HOME/packages
```
