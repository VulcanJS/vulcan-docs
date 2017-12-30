---
title: Routing
---

## Linking to a Route

To create a link to a route, use React Router's `<Link>` (and not the usual `<a>`):

```js
import { Link } from 'react-router';

<Link to={`/post/${post._id}`}>{post.title}</Link>
```

Note that you have to specify the route's full path as the `to` prop, not its name. 

## Adding Routes

Here's how you can add routes to your app (using React Router):

```js
import Foo from './foo.jsx';

addRoute({
  name: 'foo',
  path: '/foo',
  component: Foo
});
```

If on the other hand you've previously registered a component with `registerComponent`, you can simply specify the `componentName` property instead:

```js
addRoute({
  name: 'foo',
  path: '/',
  componentName: 'Foo'
});
```

Finally, to change the index (`/`) route, you can just do:

```js
addRoute({
  name: 'foo',
  path: '/',
  component: Foo
});
```

## Child Routes

You can declare a new route as a child of an existing one using the following syntax:

```js
addRoute({
  name: 'foo',
  path: '/',
  component: Foo
}, 'nameOfParentRoute');
```

## Custom Layouts

By default, Vulcan will use the `Components.Layout` component as a layout. However, you can also specify a different layout on a per-route basis: 

```js
addRoute({
  name: 'foo',
  path: '/',
  component: Foo,
  layoutName: 'AdminLayout'
});
```

Note that this supposes you've previously registered the `AdminLayout` component using `registerComponent`.

## Dynamic Imports

Vulcan supports dynamically loading specific routes, meaning their content will not be included in the original JavaScript bundle but instead will only be loaded when the user actually visits that route.

The following static route:

```js
import Admin from './Admin.jsx';

addRoute({
  name: 'admin',
  path: '/admin',
  component: Admin
});
```

Can be changed to a dynamic route using `getDynamicComponent` and `import(...)`:

```js
import { getDynamicComponent } from 'meteor/vulcan:core';

addRoute({
  name: 'admin',
  path: '/admin',
  component: () => getDynamicComponent(import('./Admin.jsx'))
});
```

Note that components are imported as soon as the `import()` statement runs. This is why it's important to wrap the `getDynamicComponent()` block in a function call (`() => ...`) to delay its execution and ensure dynamic components are not loaded prematurely. 

Learn more about [dynamic imports here](https://blog.meteor.com/announcing-meteor-1-5-b82be66571bb).

## Overriding Routes

You might sometimes need to override an existing route from another package. To do so, you can simply redeclare a route of the same name. If this is not possible for some reason (for example, it's a dynamically loaded route that uses a local path), you can also override specific route properties like this:

```js
import { RoutesTable } from 'meteor/vulcan:core';

RoutesTable.admin.layoutName = 'AdminLayout';
```

## Callbacks

Vulcan uses server-side and client-side hooks to allow for manipulation of server rendered markup and state, both before and after rendering. You can access these hooks by calling the [`addCallback`](callbacks.html#Adding-Callback-Functions) function. Some examples:

#### Wrap the app in a custom container
```js
function addWrapper(app, { req, res, store, apolloClient }) {
  // Wrap the app with a custom container
  const state = store.getState();
  return (
    <CustomContainer theme={state.theme}>
      {app}
    </CustomContainer>
  );
}

if (Meteor.isServer) {
  addCallback('router.server.wrapper', addWrapper);
}
```
 
#### Initialise a custom state
```js
function dehydrate(context, { req }) {
  const host = req.headers.host;
  context.apolloClient.store.dispatch({
    type: 'SUBSITEINIT',
    payload: Subsites.findOne({ host }),
  });
  return context;
}

function rehydrate(obj) {
  const { store, initialState } = obj;
  store.dispatch({
    type: 'SUBSITEINIT',
    payload: (initialState && initialState.subsite) ?  initialState.subsite.config : null,
  });
  return obj;
}

if (Meteor.isServer) {
  addCallback('router.server.dehydrate', dehydrate);
} else {
  addCallback('router.client.rehydrate', rehydrate);
}
```

Be sure you're receiving and returning the expected values by checking the [server](https://github.com/VulcanJS/Vulcan/blob/master/packages/vulcan-routing/lib/server/routing.jsx) and [client](https://github.com/VulcanJS/Vulcan/blob/master/packages/vulcan-routing/lib/client/routing.jsx) `vulcan:routing` source files (where you can also find the full list of callback strings available).

### Update callbacks

A common pattern with React Router is running callbacks after the route update (to enable things like custom scrolling behavior, for example). You can use the `router.onUpdate` callback hook to add your own callbacks:

```js
addCallback('router.onUpdate', sendGoogleAnalyticsRequest);
```

## Accessing React Router

If you need to access router properties (such as the current route, path, or query parameters) inside a component, you'll need to wrap that component with the `withRouter` HoC (higher-order component):

```js
import React, { PropTypes, Component } from 'react';
import { withRouter } from 'react-router'

class SearchForm extends Component{

  render() {
    // this.props.router is accessible
  }
}

export default withRouter(SearchForm);
```

#### Alternative Approach

React Router is initialized in the `vulcan:routing` package, and the routing API lets you add routes without having to modify the package's code. However, for more complex router customizations you can also disable the `vulcan:routing` package altogether and replace it with your own React Router code. 

## Authentication & Redirection

Authentication and redirection rely on having access to the current user object; and since data loading is handled at the component level in Vulcan, so are they.

To make things easier, Vulcan provides a `withAccess` HoC:

```js
import { withAccess } from 'meteor/vulcan:core';

const Dashboard = () => <div className="dashboard">...</div>

const accessOptions = {
  groups: ['admins'],
  redirect: '/sign-up'
}

registerComponent('Dashboard', Dashboard, [withAccess, accessOptions]);
```

The HoC takes two options: 

- `groups`: an array of group names to limit who can access the component. If not specified, will default to requiring any logged-in user. 
- `redirect`: an optional path to redirect the user to if the `groups` check fails. 

### Manual Redirects

Behind the scenes, this is equivalent to using `withCurrentUser` and `withRouter` to access `currentUser` and `router` inside a component's `constructor`. For example, here is how you would redirect non-logged-in users to the `/sign-up` route when they try to access the `/dashboard` component: 

```js
import React, { PureComponent } from 'react';

import { Components, registerComponent, withCurrentUser } from 'meteor/vulcan:core';
import { withRouter } from 'react-router';

class Dashboard extends PureComponent {

  constructor(props) {
    super(props);
    if(!props.currentUser) {
      props.router.push('/sign-up');
    }
  }

  render() {
    return (
      <div className="dashboard">
        ...
      </div>
    )
  }
}

registerComponent('Dashboard', Dashboard, withCurrentUser, withRouter);
```