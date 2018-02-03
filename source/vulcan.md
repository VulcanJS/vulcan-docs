---
title: From Telescope to Vulcan
---

**Vulcan** is the name of the next big evolution of the [Telescope](http://www.telescopeapp.org/) project.

Over the years, we've realized that Telescope could be used to do a lot more than simply build Product Hunt clones. That's why we've decided to focus our energy on building a more flexible and more generic platform, and we thought this new approach deserved a new name.

In the near future, this will mean two big changes for Telescope:

First, all various assets (repo/documentation/homepage/etc.) will progressively transition to the new VulcanJS name and branding. This might be a bit confusing at first, but we'll get there eventually!

Second, Vulcan will adopt a new “framework-first” approach. Practically speaking, this means you'll start new projects with the `framework-demo` package, and not with the optional `vulcan:posts`, `vulcan:comments`, etc. feature packages. Of course, these packages will still be available if you choose to use them. We just think it's important for new users to understand how the underlying framework works before they start using its more advanced capabilities.

### Telescope, Nova, & Vulcan

Just to clear things up, here's a short timeline of Telescope's various versions:

#### Telescope Legacy (2012-2016)

The original Telescope, built with Meteor and Blaze. Focused on being usable out of the box as a Product Hunt/Hacker News clone.

#### Telescope Nova (2016-2017)

The Meteor/React refactor of Telescope. Contained fewer features than Legacy, and was focused mainly on extensibility and flexibility. Nova originally used the Meteor pub/sub data layer (“Nova Classic”).

Later in 2016, Nova was ported to the Apollo data layer while still using Meteor (“Nova Apollo”), and all internal APIs were refactored to eliminate global variables.

#### Vulcan (2017-???)

The new name for the Apollo version of Telescope Nova.

As of March 2017, Vulcan is merely a rebranding of Telescope Nova 1.2, and does not feature any code changes.

### Why “Vulcan”?

[Vulcan](https://en.wikipedia.org/wiki/Vulcan_(mythology) is the Roman god of fire and volcanoes. Since Vulcan uses React and Apollo, we thought this was an appropriate name. After all Apollo is a Roman god too, and the React team has used [volcanoes](https://facebook.github.io/react/blog/2016/09/28/our-first-50000-stars.html) in their imagery before.

Also, the logo has a hidden reference to Telescope's original space-themed roots. Can you find it?
