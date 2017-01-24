---
title: Deployment Guide
---

The recommended way to deploy Nova is by using [Mup](https://github.com/kadirahq/meteor-up/), at least v1.0.3.

## Configuration

You should have a Linux server online, for instance [a Digital Ocean droplet running with Ubuntu](https://www.digitalocean.com).

Install globally the latest `kadirahq/meteor-up`.

```
npm install -g mup
```

Create Meteor Up configuration files in your project directory with `mup init`. In the example below, the configuration files are created in a `.deploy` directory at the root of your app.

```
cd my-app-folder
mkdir .deploy
cd .deploy
mup init
```

This will create two files :

```
mup.js - Meteor Up configuration file
settings.json - Settings for Meteor's settings API
```

Then, replace the content of the newly created `settings.json` with your own settings (you can use the content of `sample_settings.json` as a starter).

Fill `mup.js` with your credentials and optional settings (check the [Mup repo](https://github.com/kadirahq/meteor-up) for additional docs).

**Note:** the `ROOT_URL` field should be the absolute url of your deploy ; and you need to explicitly point out to use `abernix/meteord:base` docker image with a `docker` field within the `meteor` object.

```
...
meteor: {
  ...
  path: '../' // relative path of the app considering your mup config files
  env: {
        ROOT_URL: 'http://nova-app.com', // absolute url of your deploy
        ...
  },
  ...
  docker: {
        image:'abernix/meteord:base' // docker image working with meteor 1.4 & node 4
  },
  ...
},
...
```

You can take inspiration (or copy/paste) on this [`mup.js` example](https://gist.github.com/xavcz/6ddc2bb6f67fe0936c8328ab3314641d).

## Setup your server

From this folder, you can now setup Docker & Mongo your server with:
```
mup setup
```

## Deploy your app to your server

Still in the same folder, to deploy your app with your settings file:

```
mup deploy --settings settings.json
```
