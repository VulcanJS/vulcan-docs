---
title: Understanding the Vulcan Framework
---

Out of the box, Vulcan comes with many features such as handling posts and comments, generating a newsletter, and much more. 

But it's important to understand that you can also use the underlying framework powering these features directly and build a completely different type of app. 

So in this tutorial, we'll focus on understanding this framework and seeing how to build a simple paginated list of movies. 

The completed code for this tutorial can be found in the `framework-demo` package.

## What We're Building

<iframe width="560" height="315" src="https://www.youtube.com/embed/RzoYVqsD9WI" frameborder="0" allowfullscreen></iframe>

## Set Up

First, we'll make sure we're only including the Vulcan features we actually need. We can do that by removing any non-core packages in the `.meteor/packages` file (don't worry, this will only make sure these packages are not loaded, and won't actually remove them from your codebase). 

Delete the contents of your `packages` file, and replace them with this:

```
############ Core Packages ############

vulcan:core                       # core components and wrappers
vulcan:forms                      # auto-generated forms
vulcan:routing                    # routing and server-side rendering
vulcan:users                      # user management and permissions

vulcan:base-styles        # default styling
vulcan:i18n-en-us         # default language translation

accounts-password@1.3.3
```

Here's a quick overview of the main packages:

- `vulcan:core`: Vulcan's core libraries. 
- `vulcan:forms`: the library used for generating and submitting forms. 
- `vulcan:routing`: sets up and initializes routing and server-side rendering.
- `vulcan:users`: user management (groups, permissions, etc.).

We'll also keep `base-styles` to provide some basic Bootstrap styles for forms and menus, as well as `i18n-en-us` since it contains strings for core UI features. 

We'll also include Meteor's `accounts-password` package to activate password log in.

## Creating a Package

The next step will be creating a Meteor package to hold our code. This will be a *local* package, meaning it will live inside your repo, but we'll still have all the advantages of regular, remote packages (easy to enable/disable, restricted scope, ability to specify dependencies, etc.).

Create a new `my-package` directory under `/packages`. We'll use the following file structure:

```
my-package
  lib
    client
      - main.js
    components
    modules
      - index.js
    server
      - main.js
    - style.css
  - package.js
```

- `package.js` is your [package manifest](http://docs.meteor.com/api/packagejs.html), and it tells Meteor which files to load.
- `client/main.js` and `server/main.js` are the client and server entry points.
- `modules/index.js` lists all the various modules that make up our package. 
- `style.css` contains all our styles.

Set up all six directories along with the five blank files. Once this is done, let's start with the `package.js` file:

```js
Package.describe({
  name: 'my-package',
});

Package.onUse(function (api) {

  api.use([
    'vulcan:core',
    'vulcan:forms',

    'std:accounts-ui@1.2.19',
  ]);

  api.addFiles('lib/style.css', 'client');

  api.mainModule('lib/server/main.js', 'server');
  api.mainModule('lib/client/main.js', 'client');

  api.export([
    'Movies',
  ], ['client', 'server']);

});
```

The `api.use` block defines our package's dependencies: `vulcan:core`, `vulcan:forms`, and `std:accounts-ui`, a package that provides a set of React components for managing log in and sign up. 

We then define our two client and server endpoints using `api.mainModule`. Create `lib/client/main.js` and `lib/server/main.js`, and paste in the following line in both:

```js
import '../modules/index.js';
```

Finally, let's paste in a few simple styles to make our little demo look better. Create `style.css` inside `lib`:

```
.accounts-ui, .document-new {
  padding: 20px;
  border: 1px solid #ccc;
  width: 400px;
  margin-bottom: 20px;
}
.accounts-ui label{
  margin-right: 5px;
}
.accounts-ui .buttons a, .accounts-ui button{
  display: block;
  width: 100%;
  border: 1px #ccc solid;
  border-radius: 3px;
  padding: 5px;
  margin-bottom: 5px;
}
.accounts-ui button{
  background: #0072DB;
  color: white;
  border: none;
}
```

The last step to activate your package is enabling it using the `meteor add` command to add it to your `.meteor/packages` file:

```
meteor add my-package
```

It may seem like not much happened, but once Meteor restart our custom package will now be active and loaded. 

## Our First Component

We now have the basic structure of our package, so let's get to work. We'll create a new component and a new route to display it. 

First, create a new `components` directory inside `lib` if you haven't done so yet, and inside it a component named `MoviesWrapper.jsx`:

```js
import React, { PropTypes, Component } from 'react';
import { Components, registerComponent } from 'meteor/vulcan:core';

const MoviesWrapper = () => 
  <div className="wrapper framework-demo">

    <div className="header">
      Sign up/log in form will go here
    </div>        
    
    <div className="main">
      Movie list will go here
    </div>

  </div>

registerComponent('MoviesWrapper', MoviesWrapper);
```

We're using Vulcan's `registerComponent` function to make the component available globally. You can learn more about this in the [Components & Theming](/theming.html) section, but in a nutshell registering components this way is what makes it possible for other themes or plugins to modify or extend them. 

This means we don't actually need to `export` the component, but we do need to import the actual file. Let's create a new `component.js` file inside `modules`:

```js
import '../components/MoviesWrapper.jsx';
```

And import it from `index.js`:

```js
import './components.js';
```

## Routing

Let's create a [route](/routing.html) to display this component. Create a new `routes.js` file inside `modules`:

```js
import { addRoute, getComponent } from 'meteor/vulcan:core';

addRoute({ name: 'movies', path: 'movies', componentName: 'MoviesWrapper' });
```

In case you're wondering, `addRoute` is a very thin wrapper over [React Router](https://github.com/ReactTraining/react-router). And since we've already registered our `MoviesWrapper` component using `registerComponent`, we can pass a string as the `componentName` property and let `addRouter` fetch the actual component object.

Note: we're putting our new route at `/movies` to avoid any conflicts in case the index route is already defined. But if you'd rather have your new page at `/`, just write `path: '/'` instead. 

Make sure to also import `routes.js` inside `modules/index.js`:

```js
import './components.js';
import './routes.js';
```

If everything worked properly, you should now be able to head to `http://localhost:3000/movies` and see your `MoviesWrapper` component show up. 

## The Schema

We want to display a list of movies, which means querying for data as well as setting up basic insert, edit, and remove operations. But before we can do any of that, we need to define what a “movie” is. In other words, we need a schema. 

Vulcan uses JSON schemas based on the [SimpleSchema](https://github.com/aldeed/meteor-simple-schema) package. You can also check out the [Collections & Schemas](/schemas.html) section if you want to learn more. 

Create `schema.js` inside `modules`:

```js
const schema = {
  _id: {
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  name: {
    label: 'Name',
    type: String,
    viewableBy: ['guests'],
  },
  createdAt: {
    type: Date,
    viewableBy: ['guests'],
    autoValue: (documentOrModifier) => {
      if (documentOrModifier && !documentOrModifier.$set) return new Date() // if this is an insert, set createdAt to current timestamp  
    },
  },
  year: {
    label: 'Year',
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  review: {
    label: 'Review',
    type: String,
    viewableBy: ['guests'],
  },
  userId: {
    type: String,
    optional: true,
    resolveAs: 'user: User',
    viewableBy: ['guests'],
  }
};

export default schema;
```

Note that the `_id` and `_userId` fields will be set automatically on the server, so we set them to `optional: true` so that new documents are able to be submitted without them.

We're also setting up an `autoValue` function on the `createdAt` field to initialize it to the current timestamp whenever a new document is inserted. 

Finally, we're setting `viewableBy: ['guests']` on every field to make sure they're visible to non-logged-in users (who belong to the default `guests` group). By default, any schema field is kept private, so we need to make sure we don't forget this step if we want our data to be publicly accessible.

## Setting Up a Collection

We're now ready to set up our `Movies` collection. Create a new `collection.js` file inside `modules`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';

const Movies = createCollection({

  collectionName: 'movies',

  typeName: 'Movie',

  schema,

});

export default Movies;
```

`createCollection` takes in a collection name, type name (in other words, the GraphQL type of documents belonging to this collection), a schema (as well as a few other objecs we'll learn about later) and sets up a fully working GraphQL data layer for you!

As we did previously, reference `collection.js` from `index.js`:

```js
import MoviesImport from './collection.js';

import './components.js';
import './routes.js';

Movies = MoviesImport;
```

Although this is completely optional, we're defining `Movies` as a global variable in order to export it to Meteor's global scope. While global variables are frowned upon in most contexts, exporting `Movies` makes debugging a lot easier as it makes it accessible inside the browser console and Meteor shell. 

At this point it might not look like much has changed, but we now have a functional GraphQL schema! You can see it by opening up the Meteor shell in a terminal window (by typing `meteor shell` from within your app directory) and typing:

```
import {GraphQLSchema} from 'meteor/vulcan:lib'
GraphQLSchema.finalSchema
```

## Query Resolvers

Even though we have a schema, we can't actually query for a document yet because we don't have any **query resolvers**. In a nutshell, a resolver tells the server how to respond to a specific GraphQL query, and you can learn more about them in the [Data Loading](/data-loading.html) section. 

Let's start simple. Create a new `resolvers.js` file inside `modules` and add:

```js
const resolvers = {

  list: {

    name: 'moviesList',

    resolver(root, args, context) {
      return context.Movies.find().fetch();
    },

  },

};

export default resolvers;
```

Then import this new file from `collection.js`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';

const Movies = createCollection({

  collectionName: 'movies',

  typeName: 'Movie',

  schema,
  
  resolvers,

});

export default Movies;
```

We can try out our new query resolver using [GraphiQL](https://github.com/graphql/graphiql), but first we need some data. Create a new `seed.js` file inside `server`:

```js
import Movies from '../modules/collection.js';
import Users from 'meteor/vulcan:users';
import { newMutation } from 'meteor/vulcan:core';

const seedData = [
  {
    name: 'Star Wars',
    year: '1973',
    review: `A classic.`,
    privateComments: `Actually, I don't really like Star Wars…`
  },
  {
    name: 'Die Hard',
    year: '1987',
    review: `A must-see if you like action movies.`,
    privateComments: `I love Bruce Willis so much!`
  },
  {
    name: 'Terminator',
    year: '1983',
    review: `Once again, Schwarzenegger shows why he's the boss.`,
    privateComments: `Terminator is my favorite movie ever. `
  },
  {
    name: 'Jaws',
    year: '1971',
    review: 'The original blockbuster.',
    privateComments: `I'm scared of sharks…`
  },
  {
    name: 'Die Hard II',
    year: '1991',
    review: `Another classic.`
  },
  {
    name: 'Rush Hour',
    year: '1993',
    review: `Jackie Chan at his best.`,
  },
  {
    name: 'Citizen Kane',
    year: '1943',
    review: `A disappointing lack of action sequences.`,
  },
  {
    name: 'Commando',
    year: '1983',
    review: 'A good contender for highest kill count ever.',
  },
];

Meteor.startup(function () {
  if (Users.find().fetch().length === 0) {
    Accounts.createUser({
      username: 'DemoUser',
      email: 'dummyuser@telescopeapp.org',
      profile: {
        isDummy: true
      }
    });
  }
  const currentUser = Users.findOne();
  if (Movies.find().fetch().length === 0) {
    seedData.forEach(document => {
      newMutation({
        action: 'movies.new',
        collection: Movies,
        document: document, 
        currentUser: currentUser,
        validate: false
      });
    });
  }
});
```

This time, we don't want to import this on the client, so we'll import it directly from our `server.main.js` endpoint:

```js
import '../modules/index.js';
import './seed.js';
```

Head to [http://localhost:3000/graphiql](http://localhost:3000/graphiql) and type:

```js
query moviesQuery{
  moviesList{
    createdAt
    name
  }
}
```

You should get a list of movie names and creation dates on the right. If you'd like, try requesting more fields:

```js
query moviesQuery{
  moviesList{
    createdAt
    name
    review
    year
  }
}
```

As you can see, the great thing about GraphQL is that you can specify exactly which piece of data you need!

## More Advanced Resolvers

Although our resolver works, it's fairly limited. As it stands, we can't filter, sort, or paginate our data. Even though we might not need these features right away, now is a good time to set things up in a more future-proof way.

Our resolver will accept a [terms](terms-parameters.html) object that can specify filtering and sorting options, which is then transformed into a MongoDB-compatible object by the `Movies.getParameters()` function. Additionally, we'll add a second resolver that takes in the same `terms`, but returns the *number* of documents matching these terms:

```js
const resolvers = {

  list: {

    name: 'moviesList',

    resolver(root, {terms = {}}, context) {
      let {selector, options} = context.Movies.getParameters(terms);
      options.limit = (terms.limit < 1 || terms.limit > 100) ? 100 : terms.limit;
      options.fields = context.getViewableFields(context.currentUser, context.Movies);
      return context.Movies.find(selector, options).fetch();
    },

  },

  total: {
    
    name: 'moviesTotal',
    
    resolver(root, {terms = {}}, context) {
      let {selector, options} = context.Movies.getParameters(terms);
      return context.Movies.find(selector, options).count();
    },
  
  }
};

export default resolvers;
```

Our resolvers take three arguments:

- `root`: a link back to the root of the current GraphQL node. You can safely ignore this for now. 
- `args`: an object containing the query arguments passed from the client.
- `context`: an object enabling access to our collections, as well as utilities such as `getViewableFields` (which is used to get a list of all fields viewable for the current user for a given collection). 

Once we've figured out the correct `selector` and `options` object from the `terms` and the current user, all that's left is to make the actual database query using `Movies.find()`. 

## Displaying Data

Now that we know we can access data from the client, let's see how to actually load and display it within our app. 

We'll need two pieces for this: a [**container** component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.uid3w3pk8) that loads the data, and a **presentational** component that displays it. Fortunately, Vulcan comes with [a set of built-in higher-order container components](data-loading.html) which we can use out of the box, so we can focus on the presentational components.

Create a new `MoviesItem` component inside `components`:

```js
import React, { PropTypes, Component } from 'react';
import { Components, registerComponent, withCurrentUser } from 'meteor/vulcan:core';
import Movies from '../modules/collection.js';

class MoviesItem extends Component {

  render() {
    const movie = this.props;
    return (
      <div>
        <h2>{movie.name} ({movie.year})</h2>
      </div>
    )
  }

}

registerComponent('MoviesItem', MoviesItem);
```

We'll also create a new `MoviesList` component, which will use a [GraphQL fragment](fragments.html) (which we'll define in the next section) to specify what data we want to load:

```js
import React, { PropTypes, Component } from 'react';
import Movies from '../modules/collection.js';
import { Components, registerComponent, withList, withCurrentUser } from 'meteor/vulcan:core';

const LoadMore = props => <a href="#" className="load-more button button--primary" onClick={e => {e.preventDefault(); props.loadMore();}}>Load More ({props.count}/{props.totalCount})</a>

class MoviesList extends Component {
  render() {
    if (this.props.loading) {
      return <Components.Loading />
    } else {
      const hasMore = this.props.totalCount > this.props.results.length;
      return (
        <div className="movies">
          {this.props.results.map(movie => <Components.MoviesItem key={movie._id} {...movie} currentUser={this.props.currentUser} refetch={this.props.refetch} />)}
          {hasMore ? <LoadMore {...this.props}/> : <p>No more movies</p>}
        </div>
      )
    }
  }

}

const options = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragmentName: 'MoviesItemFragment',
  limit: 5,
};

registerComponent('MoviesList', MoviesList, withList(options), withCurrentUser);
```

The `registerComponent` utility has a nitfy trick: any additional arguments you pass it after the component name and component will be treated as [higher-order components](https://facebook.github.io/react/docs/higher-order-components.html) and wrapped around the component.

Here, we want to provide `MoviesList` with the `results` document list and the `currentUser` property, so we wrap it with `withList` and `withCurrentUser`.

By now you know the drill. Add these two new components to `components.js`:

```js
import '../components/MoviesWrapper.jsx';
import '../components/MoviesItem.jsx';
import '../components/MoviesList.jsx';
```

Then call this component inside `MoviesWrapper`:

```js
import React, { PropTypes, Component } from 'react';
import { Components, registerComponent } from 'meteor/vulcan:core';

const MoviesWrapper = () => 
  <div className="wrapper framework-demo">

    <div className="header">
      Sign up/log in form will go here
    </div>        
    
    <div className="main">
      <Components.MoviesList />
    </div>

  </div>

registerComponent('MoviesWrapper', MoviesWrapper);
```

## Fragments

At this stage, we'll probably get an error message saying the fragment we're using is not yet defined. To remedy this, create a new `fragments.js` file inside `modules`:

```js
import { registerFragment } from 'meteor/vulcan:core';

registerFragment(`
  fragment MoviesItemFragment on Movie {
    _id
    name
    year
    createdAt
    userId
    user {
      displayName
    }
  }
`);
```

And –as usual— don't forget to import it within `index.js`, making sure that it comes *before* `components.js`:

```js
import MoviesImport from './collection.js';

import './fragments.js';
import './components.js';
import './routes.js';

Movies = MoviesImport;
```

Once you save, you should finally get your prize: a shiny new movie list displayed right therte on your screen!

## User Accounts

So far so good, but we can't yet do a lot with our app. In order to give it a little more potential, let's add user accounts.

Create a new `AccountsForm.jsx` component inside `components`:

```js
import React, { PropTypes, Component } from 'react';
import { Accounts } from 'meteor/std:accounts-ui';
import { withApollo } from 'react-apollo';
import { registerComponent } from 'meteor/vulcan:core';

Accounts.ui.config({
  passwordSignupFields: 'USERNAME_AND_EMAIL',
});

const AccountsForm = ({client}) => {
  return (
    <div>
      <Accounts.ui.LoginForm 
        onPostSignUpHook={() => client.resetStore()}
        onSignedInHook={() => client.resetStore()}
        onSignedOutHook={() => client.resetStore()}
      />
    </div>
  )
}

registerComponent('AccountsForm', AccountsForm, withApollo);
```

Add it to `components.js`:

```js
import '../components/MoviesWrapper.jsx';
import '../components/MoviesItem.jsx';
import '../components/MoviesList.jsx';
import '../components/AccountsForm.jsx';
```

And hook it up inside `MoviesWrapper`:

```js
import React, { PropTypes, Component } from 'react';
import { Components, registerComponent } from 'meteor/vulcan:core';

const MoviesWrapper = () => 
  <div className="wrapper framework-demo">

    <div className="header">
      <Components.AccountsForm />
    </div>        
    
    <div className="main">
      <Components.MoviesList />
    </div>

  </div>

registerComponent('MoviesWrapper', MoviesWrapper);
```

Yay! You can now log in and sign up at your own leisure. And you can also refer to the [std:accounts-ui](https://github.com/studiointeract/accounts-ui/) package documentation to learn more about setting up accounts.

## Mutations

Now that we're logged in, we can start interacting with our data. Let's build a simple form for adding new movies.

Before we can build the user-facing part of this feature though, we need to think about how the insertion will be handled server-side, using [Mutations](mutations.html).

Create a new `mutations.js` file in `modules`:

```js
import { newMutation, Utils } from 'meteor/vulcan:core';
import Users from 'meteor/vulcan:users';

const mutations = {

  new: {
    
    name: 'moviesNew',
    
    check(user) {
      if (!user) return false;
      return Users.canDo(user, 'movies.new');
    },
    
    mutation(root, {document}, context) {
      
      if (!this.check(context.currentUser, document)) throw new Error('Mutation not allowed!');

      return newMutation({
        collection: context.Movies,
        document: document, 
        currentUser: context.currentUser,
        validate: true,
        context,
      });
    },

  }

};

export default mutations;
```

This mutation performs a simple check for the presence of a logged-in user and whether they can perform the action, and then passes on the `document` property to one of Vulcan's boilerplate mutations, `newMutation`. 

Let's pass it on to our `createCollection` function in `collection.js`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';
import mutations from './mutations.js';

const Movies = createCollection({

  collectionName: 'Movies',

  typeName: 'Movie',

  schema,
  
  resolvers,

  mutations,

});

export default Movies;
```

## Actions, Groups, & Permissions

The mutation's `check` function checks if the user can perform an action named `movies.new`. We want all logged-in users (known as the `members` group) to be able to perform this action, so let's take care of this by creating a new `permissions.js` file:

```js
import Users from 'meteor/vulcan:users';

Users.groups.members.can(['movies.new']);
```

And adding it to `index.js` as usual:

```js
import MoviesImport from './collection.js';

import './fragments.js';
import './components.js';
import './routes.js';
import './mutations.js';
import './permissions.js';

Movies = MoviesImport;
```

Note that in this specific case, creating an action and checking for it is a bit superfluous, as it boils down to checking if the user is logged in. But this is a good introduction to the permission patterns used in Vulcan, which you can learn more about in the [Groups & Permissions](/groups-permissions.html) section.

One more thing! By default, all schema fields are locked down, so we need to specify which ones the user should be able to insert as part of a “new document” operation. 

Once again, we do this through the schema. We'll add an `insertableBy` property to any “insertable” field and set it to `[members]` to indicate that a field should be insertable by any member of the `members` group (in other words, regular logged-in users):

```js
const schema = {
  _id: {
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  name: {
    label: 'Name',
    type: String,
    viewableBy: ['guests'],
    insertableBy: ['members'],
    editableBy: ['members'],
  },
  createdAt: {
    type: Date,
    viewableBy: ['guests'],
    optional: true,
    autoValue: (documentOrModifier) => {
      if (documentOrModifier && !documentOrModifier.$set) return new Date() // if this is an insert, set createdAt to current timestamp  
    }
  },
  year: {
    label: 'Year',
    type: String,
    optional: true,
    viewableBy: ['guests'],
    insertableBy: ['members'],
    editableBy: ['members'],
  },
  review: {
    label: 'Review',
    type: String,
    control: 'textarea',
    viewableBy: ['guests'],
    insertableBy: ['members'],
    editableBy: ['members']
  },
  userId: {
    type: String,
    optional: true,
    viewableBy: ['guests'],
    resolveAs: 'user: User',
  }
};

export default schema;
```

While we're at it we'll also specify which fields can be edited via `editableBy`. In this case, the three fields we want members to be able to write to are `name`, `year`, and `review`. Finally, we'll also give the `review` field a `textarea` form control. 

At this point it's worth pointing out that for mutations like inserting and editing a document, we have to distinct permission "checkpoints": first, the mutation's `check` function checks if the user can perform the mutation at all. Then, each of the mutated document's fields is checked individually to see if the user should be able to mutate it.

This makes it easy to set up collections with admin-only fields, for example. 

## Forms

We now have everything we need to create a new `MoviesNewForm.jsx` component using the [SmartForm](/forms.html) utility:

```js
import React, { PropTypes, Component } from 'react';
import Movies from '../modules/collection.js';
import { Components, registerComponent, withMessages, getFragment } from 'meteor/vulcan:core';

const MoviesNewForm = props =>
  <Components.SmartForm 
    collection={Movies}
    mutationFragment={getFragment('MoviesItemFragment')}
  />

registerComponent('MoviesNewForm', MoviesNewForm, withMessages);
```

We'll pass the `MoviesItemFragment` fragment to the form so that it knows what data to return from the server once the mutation is complete. 

Let's add the form to our component list:

```js
import '../components/MoviesWrapper.jsx';
import '../components/MoviesItem.jsx';
import '../components/MoviesList.jsx';
import '../components/AccountsForm.jsx';
import '../components/MoviesNewForm.jsx';
```

And then call it from `MoviesList.jsx`:

```js
import React, { PropTypes, Component } from 'react';
import Movies from '../modules/collection.js';
import { Components, registerComponent, withList, withCurrentUser } from 'meteor/vulcan:core';

const LoadMore = props => <a href="#" className="load-more button button--primary" onClick={e => {e.preventDefault(); props.loadMore();}}>Load More ({props.count}/{props.totalCount})</a>

class MoviesList extends Component {

  render() {

    const canCreateNewMovie = Movies.options.mutations.new.check(this.props.currentUser);
    
    if (this.props.loading) {
      return <Components.Loading />
    } else {
      const hasMore = this.props.totalCount > this.props.results.length;
      return (
        <div className="movies">
          {canCreateNewMovie ? <Components.MoviesNewForm/> : null}
          {this.props.results.map(movie => <Components.MoviesItem key={movie._id} {...movie} currentUser={this.props.currentUser} refetch={this.props.refetch} />)}
          {hasMore ? <LoadMore {...this.props}/> : <p>No more movies</p>}
        </div>
      )
    }
  }

}

const options = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragmentName: 'MoviesItemFragment',
  limit: 5,
};

registerComponent('MoviesList', MoviesList, withList(options), withCurrentUser);
```

We only want to show the “New Movie” form when a user actually *can* submit a new movie, so we'll make use of the `new` mutation's `check` function to figure this out.

We need to access the current user to perform this check, so we'll use the `withCurrentUser` higher-order component. 

Now fill out the form and submit it. The query will be updated and the new movie will appear right there in our list! 

## Sorting

As it stands, our movie list isn't really sorted. What if we wanted to sort it by movie year?

To do so, create a `parameters.js` file (learn more about parameters [here](terms-parameters.html)):

```js
import { addCallback } from 'meteor/vulcan:core';

function sortByCreatedAt (parameters, terms) {
  return {
    selector: parameters.selector, 
    options: {...parameters.options, sort: {createdAt: -1}}
  };
}

addCallback("movies.parameters", sortByCreatedAt);
```

And add it to `index.js`:

```js
import MoviesImport from './collection.js';

import './fragments.js';
import './components.js';
import './routes.js';
import './mutations.js';
import './permissions.js';
import './parameters.js';

Movies = MoviesImport;
```

## Going Further

This is probably a good place to stop, but you can go further simply by going through the full code of the `framework-demo` package. In it, you'll see how to:

- Create a resolver for single documents so you can load more data for a specific movie.
- Add edit and remove mutations (along with their forms) so you can manage your list.
- Use React helpers for quickly creating modal pop-ups. 
- Use permission checks to enforce fine-grained security throughout your app.
- Define GraphQL “joins” in your schema to decorate objects with more data.

And this is just the start. You can do a lot more with Vulcan, Apollo, and React, as you'll soon see!
