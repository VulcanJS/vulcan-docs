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

Clone the [Vulcan repo](https://github.com/TelescopeJS/Telescope/) locally, then:

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

You should start by reading the [Architecture](architecture.html) section to understand how Vulcan's codebase is organized. 

Then, when you first run Vulcan, you'll see the contents of the `movies-example` package. It is recommended you go through the [Understanding the Vulcan Framework](tutorial-framework.html) tutorial to get a better grasp of how Vulcan's building blocks (data loading, forms, etc.) work. 

You can then also take a look at the code for the `movies-example-full` package, which takes the same basic example but goes a little further.

At this stage, you can either continue using Vulcan's basic building blocks, or enable the more advanced [Features Packages](packages.html) built around the `vulcan:posts` package. 

If you'd like to use and customize these packages, you can then follow the [Customizing & Extending Vulcan](tutorial-customizing.html) tutorial, which will take you through the code of the included `example-customization` package and show you how to adapt Vulcan to your needs by tweaking styles, overriding components, and inserting your own logic in Vulcan's back-end. 

## Alternative Approaches

Throughout the documentation, you'll find “Alternative Approach” sections that explain what a feature does, and how to achieve the same results without using Vulcan (typically, with standard React and/or Apollo code). This is useful if you've hit the limits of what Vulcan offers, and want to refactor parts of your app to use lower-level APIs. 

## Roadmap

You can find the [Vulcan roadmap on Trello](https://trello.com/b/dwPR0LTz/nova-roadmap).

## Contribute

We're looking for contributors! If you're interested in this project, come say hello in the [Vulcan Slack chatroom](http://slack.telescopeapp.org).
