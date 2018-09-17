---
title: Deployment Guide
---

This guide is written to get your Vulcan app deployed and running on a remote server.
Deploying a Vulcan app is much like deploying a [Meteor](https://guide.meteor.com/deployment) app. If something isn't addressed here, it might be in the official documentation.
## Meteor Up

The recommended way to deploy Vulcan is by using [Meteor Up](http://meteor-up.com/).

### Configuration

You should have a Linux server online, for instance [a Digital Ocean droplet running with Ubuntu](https://www.digitalocean.com).
> Here's a guide to a [good initial server setup](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04).

_____

On your local development machine, install the latest [Meteor Up release](https://www.npmjs.com/package/mup).

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

This will create two files:

```
mup.js - Meteor Up configuration file
settings.json - Settings for Meteor's settings API
```

Then, replace the content of the newly created `settings.json` with your own settings (you can use the content of `sample_settings.json` as a starter).


To quickly get up and running, copy/paste the following configuration and edit it to reflect your environment:


Please refer to the official [Meteor Up starting guide](http://meteor-up.com/getting-started.html) for more in-depth instructions.


> The official [mup.js configuration examples and the possible options are found here](http://meteor-up.com/docs.html#example-configs).

```javascript
module.exports = {
  servers: {
    one: {
      // TODO: set host address, username, and authentication method
      host: '123.123.321.321',
      username: 'server-username',
      // pem: '/home/user/.ssh/id_rsa',
      // password: 'server-password'
      // or neither for authenticate from ssh-agent

      // If I'm not using a default SSH port, declare which here:
      //   opts: {
      //     port: 2222,
      //   },
    },
  },

  app: {
    // TODO: change app name and path
    name: 'myAppName',
    path: '../',

    servers: {
      one: {},
    },

   // All options are optional.
    buildOptions: {
      // Set to true to skip building mobile apps
      // but still build the web.cordova architecture. (recommended)
      serverOnly: true,

      executable: 'meteor',
    },

    env: {
      // TODO: Change to your app's url
      // If you are using ssl, it needs to start with https://
      ROOT_URL: 'https://mydomain.example',
      MONGO_URL: 'mongodb://mongodb/meteor',
      MONGO_OPLOG_URL: 'mongodb://mongodb/local',
    },

    docker: {
      // change to 'abernix/meteord:base' if your app is using Meteor 1.4 - 1.5
      image: 'zodern/meteor:root',
    },

    // Show progress bar while uploading bundle to server
    // You might need to disable it on CI servers
    enableUploadProgressBar: true,
  },

  mongo: {
    servers: {
      one: {},
    },
  },

  proxy: {
    // comma-separated list of domains your website
    // will be accessed at.
    // You will need to configure your DNS for each one.
    domains: 'mydomain.example',
    ssl: {
      // TODO: disable if not using SSL
      forceSSL: true,
      // Enable let's encrypt to create free certificates
      letsEncryptEmail: 'my-active-contact@email.com',
    },
  },
};

```
> You do not have to edit the mongo settings unless you are using a custom database setup.
### Setup your server

From the `./deploy` folder, you can now set up the remote servers you have specified in your config.
It will take around 2-5 minutes depending on the server’s performance and network availability.
```
mup setup
```

### Deploy your app

Still in the `./deploy` folder, to deploy your app with your settings file:

```
mup deploy --settings settings.json
```

This will bundle the Meteor project locally and deploy it to the remote server(s). The bundling process is the same as what meteor deploy does.

If you are using the [two-repo install](http://docs.vulcanjs.org/#Two-Repo-Install-Optional) you must specify the path to your package directory:
```
METEOR_PACKAGE_DIRS="/Users/sacha/Vulcan/packages" mup deploy --settings settings.json
```

(Taking care to adapt the `/Users/sacha/Vulcan/packages` path to point to your Vulcan core repo's `/packages` directory)

### Utility Commands

- `mup reconfig` - reconfigures app with new environment variables, Meteor settings, and it updates the start script. This is also the last step of mup deploy.
- `mup stop` - stop the app
- `mup start` - start the app
- `mup restart` - restart the app
- `mup logs [-f --tail=50]` - view the app’s logs. Supports all of the flags from `docker logs`.

_____

### Troubleshooting

> Official docs on [deploying with Meteor Up](http://meteor-up.com/docs)

- [Troubleshooting docs](http://meteor-up.com/docs.html#troubleshooting)
- [Common problems](http://meteor-up.com/docs.html#common-problems)
- `meteor npm install --save bcrypt`.
- Increase `deployCheckWaitTime`.

_____

## Meteor Now

Another simple option to get your app live for no cost is by using [Meteor Now](https://github.com/jkrup/meteor-now).
Meteor-now is a tool to let you instantly deploy your Meteor apps with one command using ZEIT's [▲now](http://zeit.co/now) service.

### Configuration

Install the `now` and `meteor-now` packages:

```
npm install -g now meteor-now
```

Create now account

```
$ now --login
> Enter your email: <your email>
> Please follow the link sent to <your email> to log in.
> Verify that the provided security code in the email matches Pragmatic Manta Ray.

✔ Confirmed email address!

> Logged in successfully. Token saved in ~/.now.json
```

### Deploy

Now on your root folder, run the following command:

```
meteor-now -d -e ROOT_URL=https://mydomain.now.sh -e NODE_ENV=production`
```

Change `mydomain` with your custom domain name. `meteor-now` will automatically look for a file named `production.settings.json` at the root of your application. For a staging deployment, you can also set `NODE_ENV=development`, `meteor-now` will then look for `development.settings.json` instead.

You can also add `-e MONGO_URL=mongodb://<username>:<pass>@....` for persistance storage of data. We recommend you using a service like [mlab](https://mlab.com/).

If you are using the [two-repo install](http://docs.vulcanjs.org/#Two-Repo-Install-Optional) you must specify the path to your package directory:
```
METEOR_PACKAGE_DIRS="/Users/sacha/Vulcan/packages" meteor-now ...
```
(Taking care to adapt the `/Users/sacha/Vulcan/packages` path to point to your Vulcan core repo's `/packages` directory)

Note that the `METEOR_PACKAGE_DIRS` variable is only used locally by the machine that runs`meteor-now` during build, you **must not** send it to the server with the `-e` option, contrary to other variables.

After your deployment is done, create an alias to the domain you just specified. Just copy the hashed link provided by the now deployment. It should be copied to your clipboad. It usually follows this format: `https://xxxxxxxxx.now.sh`.

```
now alias set https://xxxxxxxxx.now.sh https://mydomain.now.sh
```

That's it! now visit `https://mydomain.now.sh`.

For more info, for how to get a custom domain name run `now -h`. You may need a premium [now](http://zeit.co/now) account.

## PM2

Contributed by [adalidda](https://github.com/VulcanJS/Vulcan/issues/1552#issuecomment-276948862).

Deploy Vulcan GraphQL/Apollo version with PM2

### Server Setup

On Ubuntu 14 or better.

#### Install nvm
Run the following command

```
sudo apt-get update
sudo apt-get install build-essential libssl-dev
curl -sL https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh -o install_nvm.sh
bash install_nvm.sh
source ~/.profile
nvm install 4.7.3
nvm alias default 4.7.3
```

Check that node 4.7.3 is your default node version, with the command `nvm ls`.

#### Install PM2

```
sudo npm i -g pm2
```

Check that your PM2 is working with the following commands:

- `pm2 list` => list the process under PM2
- `pm2 -v` => list PM2 version

#### Install nginx for the management of your host server

```
sudo apt-get install nginx
```

Note: I suggest to use the docker version of nginx

### Development Setup

#### Install pm2-meteor on your Development computer

```
npm i pm2-meteor -g
```

#### Create a deployment folder with mkdir deploy

```
Cd deploy
```

Type the following command to init the default deployment file: `pm2-meteor init`.

pm2-meteor will then create a file `pm2-meteor.json`.

Edit your `pm2-meteor.json` with your own parameters.

Below is my pm2-meteor.json given as reference.

You note that I use the `nvm` command in my `pm2-meteor.json`, because without using the nvm command, pm2 will use Node version on your server that may be not compatible with the one required by Vulcan GraphQL/Apollo (which is node version 4.7.0 or 4.7.3) and in this case pm2 will generate high use of your CPU around 100% !

```
{
  "appName": "myappname",
  "appLocation": {
    "local": "~/myappfolder",
    "branch": "myappbranch"
  },
  "meteorSettingsLocation": "~/myappfolder/mysettings.json",
  "meteorSettingsInRepo": false,
  "prebuildScript": "meteor add mycustompackage",
  "meteorBuildFlags": "--architecture os.linux.x86_64",
  "env": {
    "ROOT_URL": "https://myurl",
    "PORT": myport,
    "MONGO_URL": "mongodb://username:password@ip1:port1,ip2:port2/dbname?authSource=admin&replicaSet=replicasetname",
    "MONGO_OPLOG_URL": "mongodb://user:password@ip1:port1,ip2:port2/local?authSource=admin&replicaSet=replicasetname"
  },
  "server": {
    "host": "myserverip",
    "username": "username",
    "password": "mypassword",
    "deploymentDir": "/opt/pm2-meteor",
    "loadProfile": "",
    "nvm": {
      "bin": "~/.nvm/nvm.sh",
      "use": "4.7.2"
    },
    "exec_mode": "cluster_mode",
    "instances": 2
  }
}
```

#### Deploy your app to the host server

Type the following command

```
cd deploy
pm2-meteor deploy
```

### Useful Commands

Type the following command to check the status of your app on the server
`pm2 list`

To Stop your App, type
`pm2 stop myappname`

To restart your App, type
`pm2 restart myappname`

To delete your App, type
`pm2 delete myappname`

### References

- [How to deploy Meteor Apps with pm2-meteor](http://pm2-meteor.betawerk.co/)
- [pm2 2.3.1-next](https://libraries.io/npm/pm2)
- [How To Install Node.js on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-16-04)
- [Heroku deployment instructions](https://www.coshx.com/blog/2016/08/19/how-to-deploy-a-meteor-1-4-app-to-heroku/)

## Other Solutions

- [Galaxy](http://galaxy.meteor.com)
- [Scalingo](https://scalingo.com/)
