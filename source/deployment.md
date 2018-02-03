---
title: Deployment Guide
---

## Meteor Up

The recommended way to deploy Vulcan is by using [Meteor Up](https://github.com/zodern/meteor-up).

### Configuration

You should have a Linux server online, for instance [a Digital Ocean droplet running with Ubuntu](https://www.digitalocean.com).

On your local development machine, install globally the latest `kadirahq/meteor-up`.

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

### Setup your server

From this folder, you can now setup Docker & Mongo your server with:
```
mup setup
```

### Deploy your app to your server

Still in the same folder, to deploy your app with your settings file:

```
mup deploy --settings settings.json
```

### Troubleshooting

- `meteor npm install --save bcrypt`.
- Increase `deployCheckWaitTime`.



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

Change `mydomain` with your custom domain name.

You can also add `-e MONGO_URL=mongodb://<username>:<pass>@....` for persistance storage of data. We recommend you using a service like [mlab](https://mlab.com/).

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
