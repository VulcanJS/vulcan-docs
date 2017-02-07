---
title: Understanding the Nova Framework
---

Out of the box, Nova comes with many features such as handling posts and comments, generating a newsletter, and much more. 

But it's important to understand that you can also use the underlying framework powering these features directly and build a completely different type of app. 

So in this tutorial, we'll focus on understanding this framework and seeing how to build a simple paginated list of movies. 

The completed code for this tutorial can be found in the `framework-demo` package.

## Intro

<iframe width="560" height="315" src="https://www.youtube.com/embed/RzoYVqsD9WI" frameborder="0" allowfullscreen></iframe>

## Set Up

First, we'll make sure we're only including the Nova features we actually need. We can do that by removing any non-core packages in the `.meteor/packages` file (don't worry, this will only make sure these packages are not loaded, and won't actually remove them from your codebase). 

It should now look like this:

```
############ Nova Core ############

nova:core                       # core components and wrappers
nova:forms                      # auto-generated forms
nova:routing                    # routing and server-side rendering
nova:email                      # email
nova:users                      # user management and permissions

nova:base-styles        # default styling
nova:i18n-en-us         # default language translation

accounts-password@1.3.3
```

We'll keep `base-styles` to provide some basic Bootstrap styles for forms and menus, as well as `i18n-en-us` since it contains strings for core UI features. 

We'll also include the official `accounts-password` package to activate password log in.

## Creating a Package

The next step will be creating a Meteor package to hold our code. Create a new `my-package` directory under `/packages`, and then a `package.js` file in `my-package`:

```js
Package.describe({
  name: 'my-package'
});

Package.onUse(function (api) {

  api.use([
    'nova:core@1.0.0-nova',
    'nova:forms@1.0.0-nova',
    'std:accounts-ui@1.2.17',
  ]);

  api.mainModule('server.js', 'server');
  api.mainModule('client.js', 'client');
  
  api.addFiles('lib/style.css', 'client');

});
```

The `api.use` block defines our package's dependencies: `nova:core`, `nova:forms`, and `std:accounts-ui`, a package that provides a set of React components for managing log in and sign up. 

We then define our two client and server endpoints using `api.mainModule`. Create these two files in `my-package`, and paste in the following line in both:

```js
import './lib/modules.js';
```

Next, create a `lib` directory inside `my-package` to hold all your package's code, and create a blank `modules.js` file inside it. 

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

## Routing

We now have the basic structure of our package, so let's get to work. We'll create a new component and a new route to display it. 

First, create a new `components` directory inside `lib`, and inside it a `MoviesWrapper` component:

```js
import React, { PropTypes, Component } from 'react';

class MoviesWrapper extends Component {
  render() {
    return (
      <div className="wrapper">

        <div className="header">
          *accounts UI will go here*
        </div>        
        
        <div className="main">
          *movies list will go here*
        </div>

      </div>
    )
  }
}

export default MoviesWrapper;
```

Then, create a new `routes.js` file inside `lib`:

```js
import { addRoute } from 'meteor/nova:core';
import MoviesWrapper from './components/MoviesWrapper.jsx';

addRoute({ name: 'movies', path: '/', component: MoviesWrapper });
```

Make sure to also import `routes.js` inside `modules.js`:

```js
import './routes.js';
```

If everything worked properly, you should now be able to head to `http://localhost:3000/movies` and see your `MoviesWrapper` component show up. 

## The Schema

We want to display a list of movies, which means querying for data as well as setting up basic insert, edit, and remove operations. But before we can do any of that, we need to define what a “movie” is. In other words, we need a schema. 

Create `schema.js` inside `lib`:

```js
const schema = {
  _id: {
    type: String,
    optional: true,
  },
  name: {
    label: 'Name',
    type: String,
  },
  createdAt: {
    type: Date,
    autoValue: (documentOrModifier) => {
      if (documentOrModifier && !documentOrModifier.$set) return new Date()
    }
  },
  year: {
    label: 'Year',
    type: String,
    optional: true,
  },
  review: {
    label: 'Review',
    type: String,
  },
  userId: {
    type: String,
    optional: true,
  }
};

export default schema;
```

Note that the `_id` and `_userId` fields will be set automatically, so we set them to `optional: true` so that new documents are able to be submitted without them.

We're also setting up an `autoValue` function on the `createdAt` field to initialize it to the current timestamp whenever a new document is inserted. 

## Settings Up a Collection

We're now ready to set up our `Movies` collection. Create a new `collection.js` file inside `lib`:

```js
import { createCollection } from 'meteor/nova:core';
import schema from './schema.js';

const Movies = createCollection({

  collectionName: 'movies',

  typeName: 'Movie',

  schema,

});

export default Movies;
```

And, as we did previously, reference it from `modules.js`:

```js
import './collection.js';
import './routes.js';
```

It might not look like much has changed, but we now have a functional GraphQL schema! You can see it by opening up the Meteor shell in a terminal window (by typing `meteor shell` from within your app directory) and typing:

```
import Telescope from 'meteor/nova:lib'
GraphQLSchema.finalSchema
```

## Query Resolvers

Even though we have a schema, we can't actually query for a document yet because we don't have any **query resolvers**. Create a new `resolvers.js` file inside `lib` and add:

```js
const resolvers = {
  list: {
    name: 'moviesList',
    resolver(root, {terms, offset, limit}, context, info) {
      const options = {
        sort: {createdAt: -1},
        limit,
        skip: offset,
      };
      return context.Movies.find({}, options).fetch();
    },
  },
  total: {
    name: 'moviesTotal',
    resolver(root, {terms}, context) {
      return context.Movies.find().count();
    },
  }
};

export default resolvers;
```

And then import this new file from `collection.js`:

```js
import Telescope from 'meteor/nova:lib';
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

We can try out our new query resolver using [GraphiQL](https://github.com/graphql/graphiql), but first we need some data. Create a new `seed.js` file inside `lib`:

```js
import Movies from './collection.js';
import Users from 'meteor/nova:users';
import { newMutation } from 'meteor/nova:core';

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

This time, we don't want to import this on the client, so we'll import it directly from our `server.js` endpoint:

```js
import './lib/modules.js';
import './lib/seed.js';
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

## Displaying Data

Now that we know we can access data from the client, let's see how to actually load and display it within our app. 

We'll need two pieces for this: a **container** component that loads the data, and a **presentational** component that displays it. Fortunately, Nova comes with a set of built-in container components which we can use out of the box, so we can focus on the presentational components.

Create a new `MoviesItem` component inside `components`:

```js
import React, { PropTypes, Component } from 'react';
import gql from 'graphql-tag';

class MoviesItem extends Component {
  render() {
    const movie = this.props;
    return (
      <div>
        <h2>{movie.name} ({movie.year})</h2>
      </div>
    )
  }
};

export default MoviesItem;
```

We'll also create a new `MoviesList` component, which will use a GraphQL fragment to specify what data we want to load:

```js
import Telescope from 'meteor/nova:lib';
import React, { PropTypes, Component } from 'react';
import Movies from '../collection.js';
import MoviesItem from './MoviesItem.jsx';
import { withList } from 'meteor/nova:core';
import gql from 'graphql-tag';

const LoadMore = props => {
  return (
    <a href="#" className="load-more button button--primary" onClick={props.loadMore}>
      Load More ({props.count}/{props.totalCount})
    </a>
  )
}

class MoviesList extends Component {

  render() {
    
    if (this.props.loading) {
      return <Components.Loading />
    } else {
      const hasMore = this.props.totalCount > this.props.results.length;
      return (
        <div className="movies">
          {this.props.results.map(movie => <MoviesItem key={movie._id} {...movie} />)}
          {hasMore ? <LoadMore {...this.props}/> : <p>No more movies</p>}
        </div>
      )
    }
  }

};

export const MoviesListFragment = gql`
  fragment moviesItemFragment on Movie {
    _id
    name
    year
  }
`;

const listOptions = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragment: MoviesListFragment,
};

export default withList(listOptions)(MoviesList);
```

And import this component from within `MoviesWrapper`:

```js
import React, { PropTypes, Component } from 'react';
import MoviesList from './MoviesList.jsx';

class MoviesWrapper extends Component {
  render() {
    return (
      <div className="wrapper">

        <div className="header">
          *accounts UI will go here*
        </div>        
        
        <div className="main">
          <MoviesList/>
        </div>

      </div>
    )
  }
}

export default MoviesWrapper;
```

## User Accounts

So far so good, but we can't yet do a lot with our app. In order to give it a little more potential, let's add user accounts.

Create a new `Accounts.jsx` component inside `components`:

```js
import React, { PropTypes, Component } from 'react';
import { Accounts } from 'meteor/std:accounts-ui';
import { withApollo } from 'react-apollo';

Accounts.ui.config({
  passwordSignupFields: 'USERNAME_AND_EMAIL',
});

const AccountsComponent = ({client}) => {
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

export default withApollo(AccountsComponent);
```

And hook it up inside `MoviesWrapper`:

```js
import React, { PropTypes, Component } from 'react';
import MoviesList from './MoviesList.jsx';
import Accounts from './Accounts.jsx';

class MoviesWrapper extends Component {
  render() {
    return (
      <div className="wrapper">

        <div className="header">
          <Accounts/>
        </div>        
        
        <div className="main">
          <MoviesList/>
        </div>

      </div>
    )
  }
}

export default MoviesWrapper;
```

You can refer to the [std:accounts-ui](https://github.com/studiointeract/accounts-ui/) package documentation to learn more about setting up accounts.

## Adding Movies

Now that we're logged in, we can start interacting with our data. Let's build a simple form for adding new movies.

Before we can build the user facing part of this feature though, we need to think about how the insertion will be handled server-side. 

Create a new `mutations.js` file in `lib`:

```js
import { newMutation } from 'meteor/nova:core';

const mutations = {
  new: {
    
    name: 'moviesNew',
    
    check(user) {
      return !!user;
    },
    
    mutation(root, {document}, context) {
      if (!this.check(context.currentUser, document)){
        throw new Error('Mutation error!');
      }
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

This mutation performs a simple check for the presence of a logged-in user, and then passes on the `document` property to one of Nova's boilerplate mutations, `newMutation`. 

Let's pass it on to our `createCollection` function in `collection.js`:

```js
import schema from './schema.js';
import resolvers from './resolvers.js';
import mutations from './mutations.js';

const Movies = createCollection({

  collectionName: 'movies',

  typeName: 'Movie',

  schema,

  resolvers,

  mutations,

});

export default Movies;
```

One last thing! By default, all schema fields are locked down, so we need to specify which ones the user should be able to insert as part of a “new document” operation. 

Once again, we do this through the schema. We'll add an `insertableBy` property to any “insertable” field and set it to `[default]` to indicate that a field should be insertable by any member of the `default` group (in other words, regular logged-in users):

```js
const schema = {
  _id: {
    type: String,
    optional: true,
  },
  name: {
    label: 'Name',
    type: String,
    insertableBy: ['members'],
  },
  createdAt: {
    type: Date,
    autoValue: (documentOrModifier) => {
      if (documentOrModifier && !documentOrModifier.$set) return new Date() // if this is an insert, set createdAt to current timestamp  
    }
  },
  year: {
    label: 'Year',
    type: String,
    optional: true,
    insertableBy: ['members'],
  },
  review: {
    label: 'Review',
    type: String,
    insertableBy: ['members'],
    control: 'textarea',
  },
  userId: {
    type: String,
    optional: true,
  }
};

export default schema;
```

In this case, the three fields we want users to be able to write to are `name`, `year`, and `review`. While we're at it we'll also give the `review` field a `textarea` form control. 

We now have everything we need to create a new `MoviesNewForm.jsx` component:

```js
import React, { PropTypes, Component } from 'react';
import NovaForm from "meteor/nova:forms";
import Movies from '../collection.js';
import { MoviesListFragment } from './MoviesList.jsx';

const MoviesNewForm = (props, context) => {
  return (
    <NovaForm 
      collection={Movies}
      fragment={MoviesListFragment}
    />
  )
}

export default MoviesNewForm;
```

We'll pass the `MoviesListFragment` to the form so that it knows what data to return from the server once the mutation is complete. 

Let's import the form from `MoviesList.jsx`:

~~~javascript
import React, { PropTypes, Component } from 'react';
import Movies from '../collection.js';
import MoviesItem from './MoviesItem.jsx';
import { withCurrentUser, withList } from 'meteor/nova:core';
import MoviesNewForm from './MoviesNewForm.jsx';
import { compose } from 'react-apollo';
import gql from 'graphql-tag';

const LoadMore = props => {
  return (
    <a href="#" className="load-more button button--primary" onClick={props.loadMore}>
      Load More ({props.count}/{props.totalCount})
    </a>
  )
}

class MoviesList extends Component {

  render() {

    const canCreateNewMovie = Movies.options.mutations.new.check(this.props.currentUser);

    if (this.props.loading) {
      return <Components.Loading />
    } else {
      const hasMore = this.props.totalCount > this.props.results.length;
      return (
        <div className="movies">
          {canCreateNewMovie ? <MoviesNewForm /> : null}
          {this.props.results.map(movie => <MoviesItem key={movie._id} {...movie} />)}
          {hasMore ? <LoadMore {...this.props}/> : <p>No more movies</p>}
        </div>
      )
    }
  }

};

export const MoviesListFragment = gql`
  fragment moviesItemFragment on Movie {
    _id
    name
    year
    user {
      __displayName
    }
  }
`;

const listOptions = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragment: MoviesListFragment,
};

export default compose(
  withList(listOptions), 
  withCurrentUser
)(MoviesList);
~~~

We only want to show the “New Movie” form when a user actually *can* submit a new movie, so we'll make use of the mutation's `check` function to figure this out.

We need to access the current user to perform this check, so we'll use the `withCurrentUser` higher-order component and use the `compose` utility to keep the syntax clean. 

Now fill out the form and submit it. The query will be updated and the new movie will appear right there in our list! 

## Going Further

This is probably a good place to stop, but you can go further simply by going through the code of the `framework-demo` package. In it, you'll see how to:

- Create a resolver for single documents so you can load more data for a specific movie.
- Add edit and remove mutations and forms so you can manage your list.
- Use React helpers for quickly creating modal pop-ups. 
- Use permission checks to enforce fine-grained security throughout your app.
- Define GraphQL “joins” in your schema to decorate objects with more data.

And this is just the start. You can do a lot more with Nova, Apollo, and React, as you'll soon see!
