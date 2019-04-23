---
title: Storybook
---

# Storybook

If you don't know Storybook yet, please read [the official documentation](<https://storybook.js.org/>).

## Under the hood

Storybook, as many modern JavaScript development tools, relies on Webpack to build the application. However, it is not currently possible to load Meteor packages using Webpack. See [this discussion on GitHub](<https://github.com/storybooks/storybook/issues/3329>) for more insights.

To overcome this difficulty, we have created custom Webpack loaders that are able to load Vulcan packages, even if they are based on Meteor. Thanks to those Webpack loaders, we are able to import, build and display Vulcan components. 

The [Vulcan Starter](<https://github.com/VulcanJS/Vulcan-Starter>) provides a fully working example with multiple stories.

## Contribute

This feature is still experimental. We'd be glad to hear about your feedback on [Slack](<http://slack.telescopeapp.org/>) or even receive your contributions on [GitHub](<https://github.com/VulcanJS/Vulcan-Starter>)!

## Prerequisites

- You application must be setup using a [2-repo install](<http://docs.vulcanjs.org/index.html#Two-Repo-Install-Optional>). This is a mandatory step because Webpack will load the Vulcan packages based on your local installation.
- The default config will look for your Vulcan local installation based on environment variable. 
  - **If your install folder is named exactly "Vulcan"**, setup your `METEOR_PACKAGE_DIRS`  as usual. Vulcan Loader will automatically detect your install.
  - **If your Vulcan install folder have a specific name**, setup a `VULCAN_DIR` environment variable to point to the **root** path of your Vulcan install.

## Install

### Install in an application based on Vulcan Starter

Good news, the current version of the Vulcan Starter already includes Storybook. You can jump to the setup section.

### Install in an application not based on a recent version of Vulcan Starter

- Copy-paste the `.storybook` and `stories` folder from [Vulcan Starter](https://github.com/VulcanJS/Vulcan-Starter) in your app
- Copy-paste `.babelrc` in your app if it does not exist already
- Add "stories" to your `.meteorignore` file. This excludes stories from your application build.
```sh
echo "stories" >> .meteorignore
```
- Enhance your `package.json` with a storybook script:

```json
	"storybook": "start-storybook -p 6006",
    	"build-storybook": "build-storybook
```

- Install the following packages:

```sh
# Install Storybook and relevant addons
npm i --save-dev @storybook/addon-actions
npm i --save-dev @storybook/addon-links
npm i --save-dev @storybook/addons
npm i --save-dev @storybook/react
npm i --save-dev @storybook/theming
npm i --save-dev storybook-addon-intl
npm i --save-dev storybook-react-router

# Must explicitely install Webpack 4.28.4 because of https://github.com/webpack/webpack/issues/8656
npm i --save-exact --save-dev webpack@4.28.4

# Necessary to support Dynamic Imports
npm i --save-dev @babel/plugin-syntax-dynamic-import

# Load Vulcan packages from your local install
npm i --save-dev vulcan-loader

# Ditch other Meteor imports
npm i --save-dev scrap-meteor-loader
```

## Run

Just run this command:

```
npm run storybook
```

In case of error, please check that you followed the Prerequisites and Install steps correctly.

## Setup

### Add stories

#### Folder location
**All stories must be located in the `stories` folder at the root of your application**. As a default Storybook automatically finds stories based on filenames. However, you don't want stories to be included in your Meteor build, so you have to put them in a clearly separated `stories` folder, and list this folder in your `.meteorignore` file. The Vulcan Starter provides examples of stories.

#### Components initalization

When importing a Vulcan package in a story, you may then need to manually trigger startup actions that Vulcan usually handles for you during Meteor startup. For example, the `populateComponentsApp` function must be called after a new package is imported, so that the `<Components.XX />` is correctly defined in your stories based on registered components. You may find concrete examples in the Vulcan Starter and Vulcan core stories.

### Load your packages

Our Webpack loaders, `vulcan-loader` and `scrap-meteor-loader`, are only here to handle Vulcan related packages.

For your own packages, you will need to write a custom loader similar to `vulcan-loader`. We provide a complete example in the file `.storybook/loaders/starter-example-loader.js`. You can see how this loader is used in `.storybook/webpack.config.js` file.

**Note:** As a default, `vulcan-loader` and our example loader expects the client side entry point to be **`your-vulcan-package/lib/client/main.js`**.

In order to be able to write a similar package loader for your application, we advise to define a strict naming convention for your package. This will allow to differentiate them from usual Meteor packages automatically. This is exactly what we do in Vulcan, with the `vulcan:` prefixing convetion. If your app is named "My App", you can prefix all your packages with `ma:` or `ma-`. You can also write a simpler loader and list explicitly all your packages in it.

### Mocks

Since Meteor is not compatible with Webpack, we do not actually run the stories in a Meteor environment. Thus, global variables must be mocked.

We provide basic mocks for global variables like Meteor, Mongo or `_` (underscore) in the `.storybook/mocks` folder. You can improve them if necessary.

### Decorators

Decorators are a way to wrap all your stories with a component. The main use case is loading your UI library and .css files. We provide two examples, with Bootstrap and Material UI. You can reuse or improve them based on your needs.

## Caveats

Meteor packages and Meteor itself are either mocked or simply removed. Thus, some components or packages may fail either at build time or runtime. Hopefully, this can be fixed by enhancing the mocks.

