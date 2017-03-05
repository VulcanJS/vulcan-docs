## Intro

Vulcan is a new kind of toolkit that lets you build single-page web apps a lot faster. You'll get ready-made components to take care of common tasks such as data loading, user accounts, form handling, and much more. 

## 3 principles

### No Dumb Work

Vulcan's main goal is to take care of all the repetitive tasks involved in building a web app: hooking up user accounts, coding up forms, setting up CRUD operations… if it's something you've done before, Vulcan should do it for you!

### Flexible

Typically, the more features a framework has, the more rigid it gets. But Vulcan was built with extensibility and flexibility in mind from the start, meaning you can override and extend its features without having to ever touch the core. 

### Eject Anytime

There's nothing worse than realizing that the technology you've sunk hundreds of hours into isn't right for your needs. To avoid this, Vulcan is architectured to make it easy to take your React components or GraphQL resolvers, and migrate to a different stack altogether. Outgrowing Vulcan shouldn't be a big deal. 

## Technology

Vulcan is based on three open-source technologies that work hand-in-hand:

### Front-End: React

React has rapidly become one of the best ways to build robuts web app UIs. Its component-based approach and vast ecosystem make it ideal for rapid development. 

### Data Layer: GraphQL

GraphQL (using the [Apollo](http://apollostack.com) implementation) is an ideal way of getting your data from the server to the client. It's extremely flexible, used by some of the biggest companies in the world, and best of all lets you request exactly the data you need. 

### Server & Build Tool: Meteor

Meteor is the glue that binds everything else together. Its ready-made built tool, database, and user accounts make creating new projects a snap. 

## Features

### Data Layer

Vulcan makes leveraging GraphQL easy by letting you skip all the boilerplate. Just specify the data you need, and let Vulcan's data layer do the rest. 

### Forms

What if you never had to code a form ever again? Vulcan generates forms based on your collections schemas, and also handles the submission and validation process for you. 

### Users

Every app needs user accounts, and Vulcan gives you easy sign-up and log-in forms, as well as a simple groups and permissions system. 

### Theming

Vulcan's theming system lets you take a third-party theme built with React components, and then pick and choose which components you want to replace or extend. 

### Callback Hooks

Every server-side operation exposes synchronous and asynchronous hooks so you can extend or modify the server's behavior without having to modify any core files. 

### Internationalization

Vulcan comes internationalized out of the box using [react-i18n](#). Add or change strings with a few lines of code. 

## Packages

In addition to its core features, Vulcan also comes with a few optional packages that cover a web app's most common requirements. 

### Posts & Comments

Many apps are organized around a central item feed, and the Posts package gives you just that. Additionally, the Comments package lets users react to each post. 

### Categories

The Categories package enables regular or nested categories to help organize your content. 

### Newsletter

The Newsletter package works in combination with the Posts and Commments packages to automatically compile and send daily or weekly recaps of your app's activity. 

### Notifications

You can keep users notified of any activity in your app by using the Notifications package to send out email notifications whenever something important happens. 

### RSS & API

Automatically generate RSS feeds and a JSON API for your posts and comments. 

### Voting

Add upvotes and downvotes to posts and comments. 

## Team

### Sacha Greif

I learned a lot about making development more approachable by writing [Discover Meteor](http://discovermeteor.com). But you can only do so much with a book, so creating something like Vulcan was a natural next step.

### Xavier Cazalot

***

### You?

Vulcan is open source and we're always looking for new contributors, no matter if you're a JavaScript guru or coding newbie. Come say hello in [our Slack channel](http://slack.vulcanjs.org)!

### "Hack Learn Make"