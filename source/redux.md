---
title: Using Redux [REVIEW]
---

Redux is used transparently by Apollo to cache your data on the client, and it's perfectly fine to use Vulcan without ever worrying about it. 

That being said, if you need to manage state at an application-wide level (for example, you might want to track if a sidebar is open or closed), using Redux explicitly can be very useful. 

## A Redux Higher-Order Component

Vulcan doesn't come with any built-in Redux HoCs, but it's not too hard to build one. For example, here's how to create a HoC to manage a sidebar's shown/hidden state.

First, we'll create a new action and a new reducer in a new `redux.js` file. The action defines a specific type of event that can happen inside your app; while the reducer defines what to do when that action happens.

```js
import { addAction, addReducer } from 'meteor/vulcan:core';

// register messages actions
addAction({
  ui: {
    toggleSidebar() {
      return {
        type: 'TOGGLESIDEBAR',
      };
    },
  }
});

// register messages reducer
addReducer({
  ui: (state = {showSidebar: false}, action) => {
    switch(action.type) {
      case 'TOGGLESIDEBAR':
        return {
          ...state,
          showSidebar: !state.showSidebar
        };
      default:
        return state;
    }
  },
});
```

Next, let's create our higher-order component in a separate `withUI.js` file:

```js
import { getActions,} from 'meteor/vulcan:lib';
import { bindActionCreators } from 'redux';
import { connect } from 'react-redux';

const mapStateToProps = state => ({ ui: state.ui, });
const mapDispatchToProps = dispatch => bindActionCreators(getActions().ui, dispatch);

const withUI = (component) => connect(mapStateToProps, mapDispatchToProps)(component);

export default withUI;
```

Once we wrap a component with `withUI`, the following props will be available to it:

- `this.props.ui.showSidebar`: a boolean that indicates if the sidebar is shown or not.
- `this.props.toggleSidebar`: a function that toggles the sidebar on and off. 
