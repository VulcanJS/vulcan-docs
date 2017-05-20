---
title: Routing
---

### Adding Routes

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

### Child Routes

You can declare a new route as a child of an existing one using the following syntax:

```js
addRoute({
  name: 'foo',
  path: '/',
  component: Foo
}, 'nameOfParentRoute');
```

### Update Callbacks

A common pattern with React Router is running callbacks after the route update (to enable things like custom scrolling behavior, for example).

You can use the `router.onUpdate` callback hook to add your own callbacks:

```js
addCallback('router.onUpdate', sendGoogleAnalyticsRequest);
```

#### Alternative Approach

React Router is initialized in the `vulcan:routing` package, and the routing API lets you add routes without having to modify the package's code. However, for more complex router customizations you can also disable the `vulcan:routing` package altogether and replace it with your own React Router code. 

### Using React Router In Your Components

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
