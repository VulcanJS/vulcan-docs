---
title: Introduction
---

## What's Nova?

Nova is a framework that gives you a set of tools for quickly building React-based web applications. Out of the box, it can handle data loading, automatically generate forms, handle email notifications, and much more. 

You can get a good overview of what Nova can do in the [Features](features.html) section. 

## Install

Clone the [Nova repo](https://github.com/TelescopeJS/Telescope/) locally, then:

```
git checkout devel
npm install
meteor
```

And open `http://localhost:3000/` in your browser.

## Getting Started

There are two main ways of using Nova: you can use the features it provides out of the box such as posts, comments, a newsletter, and so on. Or, you can use the underlying *framework* that powers all these features. 

In either case, you should start by reading the [Architecture](architecture.html) section to understand how Nova's codebase is organized. 

Once you've done that, depending on which path you're interested in you can refer to either one of these two tutorials:

1. [Customizing & Extending Nova](tutorial-customizing.html) will take you through the code of the included `customization-demo` package and show you how to adapt Nova to your needs by tweaking styles, overriding components, and inserting your own logic in Nova's back-end. 
2. [Understanding the Nova Framework](tutorial-framework.html) on the other hand will teach you how to use Nova's core data management features to display a list of movies, complete with pagination and a data insertion form. 

If you're not sure which one is for you, I recommend starting with the first tutorial, as the second one covers more advanced concepts.

## Contribute

We're looking for contributors! If you're interested in this project, come say hello in the [Nova Slack chatroom](http://slack.telescopeapp.org).