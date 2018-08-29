---
title: Read Me
---

## What's Vulcan?

Vulcan is a framework that gives you a set of tools for quickly building React & GraphQL-based web applications. Out of the box, it can handle data loading, automatically generate forms, handle email notifications, and much more. 

## Why Vulcan?

Vulcan is unique in that it spans the full stack, from the database to the browser. While there exists many excellent back-end frameworks or backend-as-a-service providers, these all still require you to handle your front-end code yourself, including building a set of components to read, write, and display data. 

But because Vulcan is full-stack, it's able to speed up these repetitive tasks by providing a whole range of helper components that are fine-tuned to the Vulcan back-end, and as a result just work out of the box. 

You can get a good overview of what Vulcan can do in the [Features](features.html) section. 

## Install

- [Full video tutorial](https://www.youtube.com/watch?v=aCjR9UrNqVk)

In order to run a Vulcan project, we'll need to first install [Node](https://nodejs.org/en/) and [Meteor](https://www.meteor.com/):

1. Install the latest version of Node and NPM. We recommend the usage of [NVM](http://nvm.sh).

2. You can then install [Meteor](https://www.meteor.com/install), which is used as the Vulcan build tool.

3. Clone the [Vulcan Starter repo](https://github.com/VulcanJS/Vulcan-Starter) locally.

4. Run:

```sh
npm install
npm start
```

And open `http://localhost:3000/` in your browser. You'll then be presented with the first step of the “Getting Started” tutorial. 

Note that you can also start the app with:

```sh
meteor --settings sample_settings.json 
```

All `npm start` does is run the above command, while also checking for the presence of `settings.json` first and creating it for you if missing. 

(A note for `windows` user: While running `npm install` you might get error regarding `node-gyp` and `bcrypt` package installation. This is because you need [windows-build-tool](https://github.com/nodejs/node-gyp/blob/master/README.md#on-windows) for `node-gyp` installation which is [required](https://github.com/kelektiv/node.bcrypt.js#dependencies) for `bcrypt` installation.)

## Two-Repo Install (Optional)

- [Full video tutorial](https://www.youtube.com/watch?v=mEemOReAw5Y)

In order to make working with Vulcan easier (especially when it comes to updating the core packages), you can also follow the more complicated "two-repo" pattern, with the two repositories being: 

1. Your own Vulcan project (for example, a clone of the Vulcan Starter project as outlined above).
2. A clone of [the main Vulcan repo](https://github.com/VulcanJS/Vulcan/), which the first repo will use as the source for the Vulcan core code. 

Follow these steps to get set up:

1. Clone the main [VulcanJS repo](https://github.com/VulcanJS/Vulcan) locally (for example, to `~/Vulcan`).
2. Create a new Meteor project or clone the [Vulcan starter repo](https://github.com/VulcanJS/Vulcan-Starter) (this will become your project's repo).
3. Inside your project repo, launch Vulcan with:

```
METEOR_PACKAGE_DIRS="/Users/sacha/Vulcan/packages" meteor --port 3000 --settings settings.json
```

(Taking care to adapt the `/Users/sacha/Vulcan/packages` path to point to your Vulcan core repo's `/packages` directory)

If you'd like, you can also export `METEOR_PACKAGE_DIRS` as a global variable for all your Meteor apps. In your `.bash_profile` file or equivalent, just add:

```
export METEOR_PACKAGE_DIRS="/Users/sacha/Vulcan/packages"
```

If you'd like to add more than one source directory (for example, one for Vulcan packages, and one for other Meteor packages), you can separate them with a `:`:

```
export METEOR_PACKAGE_DIRS="/Users/sacha/Vulcan/packages:/Users/sacha/meteor-packages"
```

### Git-Based Deploys

If you follow this install method, your Vulcan core files will *not* be included in the same Git repository as your "main" app. This can cause issues when using Git-based deploy methods such as Heroku. Either use deploy methods that bundle the whole app first (such as Meteor Up or Galaxy), or else add your core files as a Git submodule instead.

## Getting Started

When you first run the Vulcan starter repo, you'll see the “Getting Started” tutorial, and you should be able to follow along from there. 

After that, you can also go through the [Simple Example tutorial](example-simple.html) to get a grasp of how Vulcan's building blocks (data loading, forms, etc.) work. 

You can then take the more in-depth [Movies Example tutorial](example-movies.html) to get a better understanding of VulcanJS's internals. You can enable the Movies Example package by uncommenting it in `.meteor/packages`, and commenting out `example-simple` in its place)

Once you've gone through these tutorials, you can enable the `example-instagram` package, which takes the same basic example but goes a little further, as well as take a look at its code. 

At this stage, you can either continue using Vulcan's basic building blocks, or enable the more advanced forum features by checking out the [`example-forum`](example-forum.html) package. 

And if you'd like to use and customize the forum packages, you can then follow up with the [customization example tutorial](example-customization.html) tutorial, which will take you through the code of the included `example-customization` package and show you how to adapt Vulcan packages to your needs without modifying their code directly, by tweaking styles, overriding components, and inserting your own logic in Vulcan's back-end. 

You can also check out [Vulcan's YouTube channel](https://www.youtube.com/channel/UCGIvQQ6zw7ov2cHgD70HFlA) to learn more about the framework. 

## Slack Channel

Vulcan has a pretty active [Slack channel](http://slack.vulcanjs.org/) where you can ask for support, share ideas, and get tips. 

## Contribute

We're looking for contributors! If you're interested in this project, come say hello in the [Vulcan Slack chatroom](http://slack.vulcanjs.org/).
