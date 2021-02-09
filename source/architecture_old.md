---
title: Architecture
---

Here's what you need to know about Vulcan's architecture. 


## Package-Based Architecture

In keeping with this idea of flexibility and modularity, Vulcan's application structure is a bit different than most other Meteor apps. 

In Vulcan, each feature has its own distinct **Meteor package** which can be freely added or removed. [Learn more about packages here](packages.html).

## Extend, Don't Edit

Another key point of Vulcan's philosophy is to never edit the core codebase (i.e. what you get from the Vulcan repo) directly. 

Editing Vulcan's code makes it harder to keep it up to date when things change in the main Vulcan repo, as your modifications might conflict with a new updated version. 

Instead, Vulcan includes many hooks and patterns that enable you to tweak and extend core features from your *own* packages without having to actually modify their code. 

## Register & Execute

Many Vulcan objects (such as fragments, routes, components, etc.) follow a “register and execute” pattern, in which:

1. All items are first registered in a centralized array.
2. The items are then all executed at runtime. 

This two-tiered approach has two benefits. First, it means items can be overriden in between registration and execution. For example if a theme registers a component, your custom code can then extend it and the final component object that is created at runtime will be the extended version.

Second, things like React Router or the app's GraphQL schema can only be initialized once. By having you register items first, Vulcan is able to centralize objects from various sources, and then combine them all in a single initialization call. 

## The Vulcan Core Package

Unless mentioned otherwise, all Vulcan utilities function are imported from the `vulcan:core` Meteor package:

```js
import { createCollection } from 'meteor/vulcan:core';

const Posts = createCollection({...});
```

Note that a lot of these utilities actually live in the `vulcan:lib` package, but they're re-exported from `vulcan:core` for convenience's sake. 

## Alternative Approaches

Throughout the documentation, you'll find “Alternative Approach” sections that explain what a feature does, and how to achieve the same results without using Vulcan (typically, with standard React and/or Apollo code). This is useful if you've hit the limits of what Vulcan offers, and want to refactor parts of your app to use lower-level APIs. 
