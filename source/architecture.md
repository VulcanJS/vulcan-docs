---
title: Architecture
---

Nova's application structure is a bit different than most other Meteor apps. Generally speaking, we can distinguish between three ways of organizing code in a Meteor app: default, module-based, and package-based (which is what Nova uses).

The **default** app structure is what legacy Meteor apps such as [Microscope](https://github.com/DiscoverMeteor/Microscope) use. Files are stored in `/client`, `/server`, `/lib`, etc. directories and imported automatically by Meteor. This approach requires the least work, but also gives you less control over load order.

Starting with Meteor 1.3, the **module-based** approach is the pattern officially recommended by the [Meteor Guide](http://guide.meteor.com/structure.html). In it, all your files are stored in an `/imports` directory, with two `/client/main.js` and `/server/main.js` entry points that then import all other files. The main difference with the previous pattern is that files in `/imports` no longer run automatically.

Finally, with the **package-based** technique, all your code is stored in [Meteor packages](http://guide.meteor.com/using-packages.html). Packages can be loaded from Meteor's package server, or stored locally in your `/packages` directory. Note that it is recommended you use modules within your packages.

When customizing Nova, you can use any of these three approaches for your own custom code. But if you can, I would recommend sticking with Nova's **package-based** approach just to maintain consistency between Nova's codebase and yours.

Also, using packages for customization means you have an easy way to turn off any customization you've added if you need to track down the source of a problem.
