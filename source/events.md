---
title: Analytics & Event Tracking
---

Vulcan provides an API to add support for multiple analytics and event tracking providers.

## Automated Analytics

The following events will happen automatically for every analytics provider (assuming they are supported):

* `init` (client only): initialize the provider's code snippet.
* `identify` (client/server): identify the current user, if they exists.
* `page` (client only): track any page changes.

## Manual Event Tracking

Additionally, you can also manually track specific events using the `track` function:

```js
import { track } from 'meteor/vulcan:events';

track('clickedSignUp', { time: new Date(), foo: 'bar' });
```

`track` takes three argument: the name of the event (as a string), an optional properties object, and an optional `currentUser` object which will be used if the current user can't be inferred from the context (for example, you're calling `track` on the server).

You can call `track` on either client or server (assuming both versions have been implemented), but note that even though both versions have the same API, they might work differently behind the scenes (see below). 

## Built-in Providers

* `vulcan:events-ga`: Google Analytics (note: currently client-only)
* `vulcan:events-segment`: Segment
* `vulcan:events-internal`: Internal database-stored event tracking

## Adding Providers

To add a provider, you can use the following methods (applied here to [Segment](https://segment.com/)'s client-side JavaScript API):

#### `addInitFunction` (client only)

Pass a function that will be called immediately to initialize the provider. For example:

```js
import { addInitFunction, getSetting } from 'meteor/vulcan:events';

function segmentInit() {
  !(function() {
    var analytics = (window.analytics = window.analytics || []);
    /*
    snippet code
    */
    analytics.SNIPPET_VERSION = '4.0.0';
    analytics.load(getSetting('segment.clientKey'));
  })();
}
addInitFunction(segmentInit);
```

Note that on the server, initialization will typically happen inside the provider package's own code, which means you don't need to use `addInitFunction`. 

#### `addPageFunction`  (client only)

Pass a function taking the current `route` as argument that will be used to track navigation to a new page:

```js
import { addPageFunction } from 'meteor/vulcan:events';

function segmentTrackPage(route) {
  const { name, path } = route;
  const properties = {
    url: Utils.getSiteUrl().slice(0, -1) + path,
    path,
  };
  window.analytics.page(null, name, properties);
  return {};
}
addPageFunction(segmentTrackPage);
```

In a single-page web app, the concept of "page" only makes sense on the client which is why the `addPageFunction` function doesn't do anything on the server. 

#### `addIdentifyFunction` (client and server)

Pass a function taking the current `currentUser` as argument that will be used to identify users:

```js
import { addIdentifyFunction } from 'meteor/vulcan:events';

function segmentIdentify(currentUser) {
  window.analytics.identify(currentUser.userId, {
    email: currentUser.email,
    pageUrl: currentUser.pageUrl,
  });
}
addIdentifyFunction(segmentIdentify);
```

#### `addTrackFunction` (client and server)

Pass a function that takes an event name and event properties:

```js
import { addTrackFunction } from 'meteor/vulcan:events';

function segmentTrack(eventName, eventProperties) {
  analytics.track(eventName, eventProperties);
}
addTrackFunction(segmentTrack);
```

Note that `track` can have different implementations on the client and server. For example, the `vulcan:events-internal` package's client version of `track` will trigger a GraphQL mutation, while the server version will mutate the data directly. Or, the client version of `vulcan:events-segment` will use an in-browser JavaScript snippet while the server version will use Segment's Node API via an NPM package.

