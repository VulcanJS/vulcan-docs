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


----
# Setting up redux:vulcan with the 1.13.0 update from  [Laurent](https://github.com/VulcanJS/Vulcan/pull/2350)

There are lots of ways to manage the state of a project, but for some reason you might want to use Redux. Here's the steps I took to get it working with the help of Laurent and his amazing work on the update.

1. Add redux:vulcan
2. Set up redux.js
3. Import redux.js 
4. Create a Higher Order Component
5. Register the HOC with your component

### 1. Add it to package.js

---

First, add the vulcan:redux v1.13.0 package to the `package.js` of your project. It sits in the `api.use` function inside `Package.onUse` . Here is an example - don't forget to add @1.13.0:

    Package.describe({
      name: 'my-app',
    });
    
    Package.onUse(function (api) {
    
      api.use([
        'promise',
        'fourseven:scss',
    
        // vulcan core
        'vulcan:core@1.13.0',
    
        // vulcan packages
        'vulcan:forms@1.13.0',
        'vulcan:accounts@1.13.0',
        'vulcan:ui-bootstrap@1.13.0',
        
        //vulcan redux 👇👇👇
        'vulcan:redux@1.13.0'
        //over here! 👆👆👆
    
      ]);
    
      api.addFiles([
        'lib/stylesheets/styles.scss'
      ], ['client']);
    
      api.mainModule('lib/server/main.js', 'server');
      api.mainModule('lib/client/main.js', 'client');
    
    });

If you can't find this file, it'll be somewhere like: `Vulcan/packages/your-project/package.js`

### 2. Set up a redux.js file in your project 📁

---

In your project, create a folder called redux, and inside it, a file called redux.js: `redux/redux.js`. This is how I have it structured:

![directory](https://tinify-bucket.s3-us-west-1.amazonaws.com/directory.png)

In this redux.js file, the first line should import 3 things from vulcan:redux:

    import {addAction, addReducer, setupRedux} from 'meteor/vulcan:redux';

Add you actions and reducers first, and **only call** `setupRedux()` **at the very end, or it will not work**. Here is a full example based off the [previous docs](http://docs.vulcanjs.org/redux.html):

    import {addAction, addReducer, setupRedux} from 'meteor/vulcan:redux';
    
      addAction({
        ui: {
          toggleSidebar(payload) {
            return {
              type: 'TOGGLESIDEBAR',
              payload
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
                showSidebar: action.payload
              };
            default:
              return state;
          }
        },
      });
    
    	//set up redux
      setupRedux();

### 3. Import Redux.js

---

This step is also important to get right. Redux.js must be imported before your components. In `/lib/modules/index.js` (using the Vulcan Starter structure), import your redux file first:

    //Redux 👇👇👇
    import '../redux/redux.js';
    
    // The Movies collection
    import './movies/collection.js';
    
    // Components
    import './components.js';
    
    // Routes
    import './routes.js';

### 4. Create a Higher Order Component

---

Following on from the original example again, a HoC lets you inject the state into any component. For this toggle example, I've created a file named `withUI.js` within a hoc folder:

    import { bindActionCreators } from 'redux';
    import { connect } from 'react-redux';
    import {getActions, getStore} from 'meteor/vulcan:redux';
    
    
    const mapStateToProps = state => (getStore().getState());
    
    const mapDispatchToProps = dispatch => bindActionCreators(getActions().ui, dispatch);
    
    const withUI = (component) => connect(mapStateToProps, mapDispatchToProps)(component);
    
    export default withUI;

- Lines 1-2 import the regular redux methods from `redux` and `react-redux` packages.
- Line 3 imports `getActions` and `getStore` from `vulcan:redux`
    - **getActions** will grab the actions previously defined in your redux.js
    - **getStore** lets you access the state and map them to your components in  `mapStateToProps`.
- Line 4: here we expose the entire state with `getStore().getState()`.

### 5. Register the HOC with your component

---

The last step is to use your HoC as an argument when registering your component. First, import the HOC file at the top of your component. This example uses the withUI HOC made in step 4:

    import withUI from '../../../hocs/withUI.js';

Now add `withUI` into the `hocs` in the `registerComponent()` call:

    registerComponent({
      name: "MenuComponent",
      component: MenuComponent,
      hocs: [withUI]
    });

That should be it - you'll be able to see the state immiediately in redux. 

*In this example, the state was defined in the addReducer, and not passed in with setupRedux (see step 2).*

![state example](https://tinify-bucket.s3-us-west-1.amazonaws.com/state.png)

# Using setupRedux Parameter

`setupRedux` also comes with an `initialState` parameter. You can pass the entire redux state you want to start with as an object. Here is my own example:

![example](https://tinify-bucket.s3-us-west-1.amazonaws.com/example.png)

So I have passed in:

    {canvasActions:{addingBlock:true}}

That is because my reducer is called 'canvasActions', so is required for the initialState parameter of setupRedux. 

[setupRedux/InitialState issues](https://www.notion.so/6a3c4735a5f14f1faebbe47af0935808)
