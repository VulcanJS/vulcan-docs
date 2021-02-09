---
title: Unit Testing
---

Currently we are using [Mocha](https://mochajs.org/) for unit and integration testing. 
At some point soon we will be migrating to [Jest](https://jestjs.io), but we will make the migration process as smooth as possible.
This is the reason we are using the Jest Expect library instead of Chai. 

You'll find documentation about Meteor testing at the following places:
 
 * [Meteor Guide: Testing](https://guide.meteor.com/testing.html)
 * [MeteorTesting:Mocha](<https://github.com/meteortesting/meteor-mocha/blob/master/README.md>)
 * [Mocha Getting Started](https://mochajs.org/#getting-started)
 * [Jest Expect API Reference](https://jestjs.io/docs/en/expect)
 * [Jest Extended Expect API Reference](https://jestjs.io/docs/en/expect)
 * [Chai BDD Expect API Reference](https://www.chaijs.com/api/bdd/)

## Getting started

Vulcan comes pre-configured for unit testing, but for your own project you'll have to install it.

If you are NOT using a [two repo install](http://docs.vulcanjs.org/index.html#Two-Repo-Install-Optional) copy vulcan/packages/meteor-mocha to your project's packages directory.

In your app's root directory run these terminal commands:

```
meteor add meteortesting:mocha
meteor npm install --save-dev chai
meteor npm install --save-dev chai-as-promised
meteor npm install --save-dev selenium-webdriver@3.6.0
meteor npm install --save-dev chromedriver@2.41.0
meteor npm install --save-dev enzyme
meteor npm install --save-dev enzyme-adapter-react-16
meteor npm install --save-dev jsdom
meteor npm install --save-dev jsdom-global
```

Open `.meteor/versions` and make sure the meteortesting packages are at their latest versions, 
and if not, update them individually using `meteor update meteortesting:browser-tests` etc.:

```
meteortesting:browser-tests@1.2.0
meteortesting:mocha@1.1.3
meteortesting:mocha-core@6.1.2
```

Open `package.json` in your app root and add the following line to the scripts section:

```json
{
  "name": "MyApp",
   . . .
  "scripts": {
     . . .
    "unit-test": "TEST_BROWSER_DRIVER=chrome ROOT_URL=http://localhost:60859 METEOR_PACKAGE_DIRS=~/Dev/Erik-Vulcan/packages meteor test-packages ./packages/* --port 60859 --settings settings-dev.json --driver-package meteortesting:mocha --raw-logs"
  }
}  
```

Make sure to replace `~/Dev/Erik-Vulcan` with the actual path to you local copy of Vuclan in your two-repo install.
Before `meteor` you can define additional environment variables that you may need, such as `MAIL_URL` or `MONGO_URL`.

## Running the tests

To start testing initiate unit-test, for example type in your console:

```
npm run test-unit
```

The unit test suite will then run and later re-run when you make code changes. You can terminate it with **CTRL-C**.

## Adding code coverage

Add the following packages in your terminal:

```
meteor add lmieulet:meteor-coverage meteortesting:mocha
meteor npm install --save-dev babel-plugin-istanbul
```

In order to instrument your code, you need to add the babel-plugin-istanbul the .babelrc file in the root of your app:

```json
{
  "env": {
    "COVERAGE": {
      "plugins": [
        "istanbul"
      ]
    }
  },
  "presets": [
    . . .
  ]
}
```

You must wrap the istanbul plugin with the env setting to disable the file-instrumentation of your project when you are not running the test coverage script. Just keep in mind that if you follow the here under script, babel will use the istanbul package only when BABEL_ENV=COVERAGE.

Now, to run the coverage process, just add these new scripts inside your package.json in the root folder of your app:

```json
{
  "name": "MyApp",
   . . .
  "scripts": {
     . . .
    "unit-test": "TEST_BROWSER_DRIVER=chrome ROOT_URL=http://localhost:60859 METEOR_PACKAGE_DIRS=~/Dev/Erik-Vulcan/packages meteor test-packages ./packages/* --port 60859 --driver-package meteortesting:mocha --raw-logs",
    "unit-test-coverage": "BABEL_ENV=COVERAGE COVERAGE=1 COVERAGE_VERBOSE=1 COVERAGE_APP_FOLDER=$PWD/ TEST_BROWSER_DRIVER=chrome ROOT_URL=http://localhost:60859 METEOR_PACKAGE_DIRS=~/Dev/Erik-Vulcan/packages meteor test-packages ./packages/* --port 60859 --driver-package meteortesting:mocha --raw-logs"
  }
}  
```
