---
title: Introduction
---

## What's Vulcan?

Vulcan is a framework that gives you a set of tools for quickly building React-based web applications. Out of the box, it can handle data loading, automatically generate forms, handle email notifications, and much more. 

You can get a good overview of what Vulcan can do in the [Features](features.html) section. 

**Note: Vulcan is the new name of Telescope Nova. [Learn more](vulcan.html)**

## Install

Install the latest version of Node and NPM. We recommend the usage of [NVM](http://nvm.sh).

You can then install [Meteor](https://www.meteor.com/install), which is used as the Vulcan build tool.

Clone the [Vulcan repo](https://github.com/VulcanJS/Vulcan/) locally, then:

```sh
npm install
npm start
```

And open `http://localhost:3000/` in your browser.

Note that you can also start the app with:

```sh
# default setting file: sample_settings.json
meteor --settings sample_settings.json 

# 'npm start' will create a copy of the sample settings, 'settings.json', and run `meteor` with this file for you.
```

## Getting Started

When you first run VulcanJS, you'll see the contents of the `example-simple` package. It is recommended you go through the [Simple Example tutorial](example-simple.html) tutorial to get a grasp of how Vulcan's building blocks (data loading, forms, etc.) work. 

You can then take the more in-depth [Movies Example tutorial](example-movies.html) to get a better understanding of VulcanJS's internals. You can enable the Movies Example package by uncommenting it in `.meteor/packages`, and commenting out `example-simple` in its place)

Once you've gone through both tutorials, you can enable the `example-instagram` package, which takes the same basic example but goes a little further, as well as take a look at its code. 

At this stage, you can either continue using Vulcan's basic building blocks, or enable the more advanced forum features by checking out the [`example-forum`](example-forum.html) package. 

And if you'd like to use and customize the forum packages, you can then follow up with the [customization example tutorial](example-customization.html) tutorial, which will take you through the code of the included `example-customization` package and show you how to adapt Vulcan packages to your needs without modifying their code directly, by tweaking styles, overriding components, and inserting your own logic in Vulcan's back-end. 

You can also check out [Vulcan's YouTube channel](https://www.youtube.com/channel/UCGIvQQ6zw7ov2cHgD70HFlA) to learn more about the framework. 

## Technology

Vulcan is built on top of three main open-source technologies: React, GraphQL, and Meteor. 

- [React](https://facebook.github.io/react/) is Vulcan's front-end library.
- [GraphQL](http://graphql.org) is Vulcan's API layer, used to load data from the server with [Apollo](http://apollostack.com). 
- [Meteor](http://meteor.com) is Vulcan's back-end layer, used to handle the database as well as server and bundle the app. 

Each layer is independent from the other two: you can take your React components and use them with a completely different stack, and – although we're not quite there yet – ultimately we also hope to make it easy to migrate out of Meteor.

The goal is to avoid getting stuck: if at any point you outgrow Vulcan, it should be possible to leave it with minimal refactoring. 

## Roadmap

You can find the [Vulcan roadmap on Trello](https://trello.com/b/dwPR0LTz/nova-roadmap).

## Contribute

We're looking for contributors! If you're interested in this project, come say hello in the [Vulcan Slack chatroom](http://slack.telescopeapp.org).
