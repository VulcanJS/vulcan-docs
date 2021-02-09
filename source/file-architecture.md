---
title: File Architecture
---

## Package-Based Architecture

Vulcan's core code follows a **file-based architecture**, which means every feature lives inside its own package which can be enabled or disabled to add or remove said feature; or even over-ridden with your own custom package of the same name. 

## App Packages

When building your own Vulcan app, you will typically create a new package that contains the code for your app. Although you are of course free to follow your own structure, we recommend the following file architecture.

### Root Level

First, a Meteor package will contain a `package.js` package manifest that list its contents. On the same level, you will have a `lib` directory that contains the entirety of the package's code. This `lib` directory is then divided into three directories: 

- `/client`: code that will only run inside the browser.
- `/server`: code that will only run on the server.
- `/modules`: code that will run on both. 

Both `client` and `server` should contain a `main.js` entry point that will load client- and server- specific code as well a shared `modules` code. 

### Client Directory

Because Vulcan enables server-side rendering, very little code actually runs *only* on the client. This directory will thus be empty save for the `main.js` entry point that should point back to `modules/index.js`. 

### Modules Directory

We recommend splitting your code around your collection (a.k.a. models). For example, a forum app could have the following directories:

- `/modules/posts`
- `/modules/users`
- `/modules/comments`

Each directory will then contain all or some of the following files: 

- `collection.js`: the main collection/model declaration
- `schema.js`: the schema definition
- `fragments.js`: GraphQL fragments related to the collection
- `helpers.js`: any other code related to the collection

Additionally, we also recommend always including an `index.js` file to centralize all imports/exports in every directory. 

#### Other Code

You will usually also end up with some code that is not directly related to any collection, such as components imports, route declarations, or internationalization definitions. You can leave those files in `/modules`. 

### Server Directory

The server directory should more or less mirror the `modules` directory in terms of being organized by collection, except it will contain code that only runs on the server, such as mutation callbacks.