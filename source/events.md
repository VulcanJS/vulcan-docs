---
title: Analytics & Event Tracking
---

Vulcan provides an API to add support for multiple analytics and event tracking providers. 

## Tracking Events

You can track events using the `track` function:

```js
import { track } from 'meteor/vulcan:events';

track('clickedSignUp', {time: new Date(), foo: 'bar'});
```

`track` takes two argument: the name of the event (as a string), and an optional properties object. 

## Providers

- `vulcan:events-ga`: Google Analytics
- `vulcan:events-segment`: Segment
- `vulcan:events-mongo`: Internal MongoDB event tracking

## Adding Providers

To add a provider, you can use the following methods (applied here to [Segment](https://segment.com/)'s client-side JavaScript API): 

#### `addInitFunction`

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

#### `addPageFunction`

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

#### `addIdentifyFunction`

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

#### `addTrackFunction`

Pass a function that takes an event name and event properties:

```js
import { addTrackFunction } from 'meteor/vulcan:events';

function segmentTrack(eventName, eventProperties) {
  analytics.track(eventName, eventProperties);
}
addTrackFunction(segmentTrack);
```
