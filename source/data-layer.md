---
title: Data Layer
---

Nova uses the [Apollo](http://apollostack.com) data layer to manage its [GraphQL API](http://graphql.org/). 

## Data Flow

When you connect to a GraphQL-powered site, the client makes an API request to load the data it needs. 

Unlike traditional REST APIs, the client doesn't just hit an API endpoint but actually sends a list of *what data it needs* to the server. The specific document properties needed by the client are defined in a **fragment**. 

Upon receiving that query, the server validates the fragment against the main GraphQL **schema**.

If that validation passes, the server then looks for a **query resolver** that will specify how to actually retrieve the data (presumably through some kind of database `find()` operation).

Data mutations (inserting, editing, or removing a document) function in a similar way.

The client sends a query that includes the name of the mutation, the mutation arguments, as well as specifies what data the client expects back once the mutation is complete. 

The server then looks for a **mutation resolver** and executes it.

## Nova Collections

Nova features a number of helpers to make settings up that data layer fast, most of which are initialized through the `createCollection` function:

```js
const Movies = createCollection({

  collectionName: 'movies',

  typeName: 'Movie',

  schema,
  
  resolvers,

  mutations,

});
```

The function takes the following arguments:

- `collectionName`: the name of the collection in your MongoDB database.
- `typeName`: the name of the GraphQL type that will be generated for the collection.
- `schema`, `resolvers`, `mutations`: see below.

### Schema

The first piece of any GraphQL API is the **schema**, which defines what data is made available to the client. 

In Nova, this GraphQL schema can be generated automatically from your collection's JSON schema, so you don't have to type things twice. Just pass a [SimpleSchema](https://github.com/aldeed/meteor-simple-schema)-compatible JSON schema as `createCollection`'s `schema` property.

You can learn more on defining your schema in the [Schema](schema.html) section.

#### Custom Schemas

If you need to manually add a schema, you can also do so using the `GraphQLSchema.addSchema` function:

```js
const customSchema = `
  input Custom {
    _id: String
    userId: String
    title: String
  }
`;
GraphQLSchema.addSchema(customSchema);
```

### Resolvers

You can pass a `resolver` object to `createCollection`. It should include three `list`, `single`, and `total` resolvers.

Each resolver should have the following properties:

- `name`: the resolver's name.
- `resolver`: the resolver function.

#### List Resolver

The `list` resolver is used to display lists of documents. It takes the following arguments:

- `terms`: a JSON object containing a list of terms used to filter and sort the list.
- `offset`: how many documents to offset the list by (used for paginating requests).
- `limit`: how many documents to return.

It should return an array of documents.

#### Single Resolver

The `single` resolver takes a `documentId` argument, and should return a single document.

#### List Total Resolver

The `total` resolver takes a `terms` argument and should return the total count of results matching these terms in the database. 

#### Custom Resolvers

Just like for the schema, you can also define resolvers manually using `GraphQLSchema.addResolvers`:

```js
const movieResolver = {
  Movie: {
    user(movie, args, context) {
      return context.Users.findOne({ _id: movie.userId }, { fields: context.getViewableFields(context.currentUser, context.Users) });
    },
  },
};
GraphQLSchema.addResolvers(movieResolver);
```

Resolvers can be defined on any new or existing type (e.g. `Movie`).

### Mutations

Finally, `createCollection` also accepts a `mutations` object. This object should include three mutations, `new`, `edit`, and `remove`, each of which has the following properties:

- `name`: the name of the mutation.
- `check`: a function that takes the current user and (optionally) the document being operated on, and return `true` or `false` based on whether the user can perform the operation.
- `mutation`: the mutation function.

#### New Mutation

Takes a single `document` argument.

#### Edit Mutation

Takes three arguments:

- `documentId`: the document to modify.
- `set`: the fields to modify (as a list of field name/value pairs, e.g.`{title: 'My New Title', body: 'My new body'}`).
- `unset`: the fields to remove (as a list of field names/booleans, e.g. `{title: true, body: true}`).

#### Remove Mutation

Takes a single `documentId` argument.

#### Custom Mutations

You can also add your own mutations using `GraphQLSchema.addMutation` and `GraphQLSchema.addResolvers`:

```js

GraphQLSchema.addMutation('postsVote(documentId: String, voteType: String) : Post');

const voteResolver = {
  Mutation: {
    postsVote(root, {documentId, voteType}, context) {
      // do mutation
    },
  },
};

GraphQLSchema.addResolvers(voteResolver);
```

#### Boilerplate Operations

Note that even when using the `new`, `edit`, and `remove` mutations, defining the actual operation is up to you.

But if you want to save time, you can import Telescope's own boilerplate mutations from `nova:lib`:

```js
import { newMutation, editMutation, removeMutation } from 'meteor/nova:lib';
```

They take the following arguments:

- `collection`: the collectin affected.
- `document` (new) or `documentId` (edit, remove): the document or document ID.
- `set`, `unset` (edit only): the `set` and `unset` objects. 
- `currentUser`: the user performing the operation.
- `validate`: whether to validate the operation based on the current user.
- `context`: the resolver context.

If `validate` is set to `true`, these boilerplate operations will: 

- Check that the current user has permission to insert/edit each field.
- Validate the document against collection schema.
- Add `userId` to document (insert only).
- Run any validation callbacks (e.g. `movies.new.validate`).

They will then run the mutation's document (or the `set` modifier) through the collection's sync callbacks (e.g. `movies.new.sync`), perform the operation, and finally run the async callbacks (e.g. `movies.new.async`).

Callback hooks run with the following arguments:

- `new`: `newDocument`, `currentUser`.
- `edit`: `modifier`, `oldDocument`, `currentUser`.
- `remove`: `document`, `currentUser`.

You can learn more about callbacks in the [Callbacks](callbacks.html) section. 

## Higher-Order Components

To make working with Apollo easier, Nova provides you with a set of higher-order components (HoC). 

An HoC is simply a function you can call on a React component to give it additional props.

### withList

The `withList` HoC is used to display lists of documents. It takes the following options:

- `queryName`: an arbitrary name for the query.
- `collection`: the collection on which to look for the `list` resolver.
- `fragment`: the fragment to use (see below).

For example:

```js
const listOptions = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragment: MoviesItem.fragment,
};

export default withList(listOptions)(MoviesList);
```

The resulting wrapped component also takes in the following options:

- `terms`: an object containing a list of quering, sorting, and filtering terms.

Note that `terms` needs to be passed not as an option, but as a prop from the *parent* component.

The HoC then passes on the following props:

- `loading`: `true` while the data is loading.
- `results`: the loaded array of documents.
- `totalCount`: the total count of results.
- `count`: the number of current results (i.e. `results.length`).
- `refetch`: a function that can be called to trigger a query refetch.
- `loadMore`: a function that can be called to load more data. 

### withDocument

The `withDocument` HoC displays a single document. It takes the same options as `withList`, but takes a `documentId` **prop**. 

Like `terms` for `withList`, `documentId` needs to be passed not as an option, but as a prop from the *parent* component.

This HoC then passes on the following child prop:

- `loading`: `true` while the data is loading.
- `document`: the loaded document.

### withNew

This HoC takes the following four options:

- `collection`: the collection to operate on.
- `queryName`: the name of the query to update on the client.
- `fragment`: specifies the data to ask for as a return value.

And passes on a `newMutation` function to the wrapped component, which takes a single `document` argument.

### withEdit

Same options as `withNew`. The returned `editMutation` mutation takes three arguments: `documentId`, `set`, and `unset`. 

### withRemove

Same options as `withNew`. The returned `removeMutation` mutation takes a single `documentId` argument. 

Note that when using the [Forms](forms.html) module, all three mutation HoCs are automatically added for you. 

## Fragments

A fragment is a piece of schema, usually used to define what data you want to query for. 

A good practice is defining the fragment wherever the data will actually be used. Note that this doesn't have to be the component that directly receives this data. 

For example, you could define a fragment in one component:

```js
class MoviesItem extends Component {
  //...
};

MoviesItem.fragment = gql`
  fragment moviesItemFragment on Movie {
    _id
    name
    year
    user {
      __displayName
    }
  }
`;

export default MoviesItem;
```

And reuse it in another:

```js
const listOptions = {
  collection: Movies,
  queryName: 'moviesListQuery',
  fragment: MoviesItem.fragment,
};

export default withList(listOptions)(MoviesList);
```

## Terms & Parameters

When you request data, you often want to do it according to specific criteria. For example, you might want to load all posts from a certain category, and order them by score. 

Nova accomplishes this through MongoDB operators such as `find`, `sort`, etc. but because it would be dangerous to let the client specify its own operators, the client instead defines a `terms` object which gets converted into MongoDB-compatible objects using the `collection.getParameters` function. 

For example, given the following `terms`:

```js
{
  view: 'top',
  cat: 'foo',
  after: '2016-12-01',
  before: '2016-12-31'
}
```

Calling `Posts.getParameters(terms)` would give you the following object:

```js
{
  selector: { 
    status: 2,
    isFuture: { '$ne': true },
    postedAt: { 
      '$gte': Thu Dec 01 2016 00:00:00 GMT+0900 (JST),
      '$lt': Sat Dec 31 2016 23:59:59 GMT+0900 (JST)
    },
    categories: { '$in': ['RmpzcMurbFysAAvhf'] } 
  },
  options: { 
    sort: { 
      sticky: -1, score: -1, _id: -1 
    }, 
    limit: 10 
  }
}
```

As you can see, the resulting object is a mix of default properties (`status`, `isFuture`) and properties calculated from the `terms` object (`postedAt.$gte`, `postedAt.$lt`, `categories`, etc.). 

Behind the scenes the `Posts.getParameters` function iterates through all the functions defined on the `posts.parameters` callback hook until it produces the final `{ selector, options}` object. 

This means you can handle additional `terms` parameters simply by adding your own callback. For example, this is how you would use a `cat` parameter accepting a category slug to fetch the corresponding category when querying the `Posts` collection:

```js
import { addCallback } from 'meteor/nova:core';

function addCategoryParameter (parameters, terms) {
  if (terms.cat) {
    const categoryId = Categories.findOne({slug: terms.cat})._id;
    parameters.selector.category = {$in: categoryId};
  }
  return parameters;
}
addCallback('posts.parameters', addCategoryParameter);
```

Note that in the default `PostsHome` component, the `terms` object is automatically built from the current URL query parameters:

```js
const PostsHome = (props, context) => {
  const terms = _.isEmpty(props.location && props.location.query) ? {view: 'top'}: props.location.query;
  return <Components.PostsList terms={terms}/>
};
```

### Posts Views

One of the parameters accepted by the `Posts` collection is `view`, which functions as a shorthand for defining multiple parameters at the same time. For example, this is how the `pending` view is defined in order to only show pending posts, while ordering them by their `createdAt` property:

```js
Posts.views.add("pending", function (terms) {
  return {
    selector: {
      status: Posts.config.STATUS_PENDING
    },
    options: {sort: {createdAt: -1}}
  };
});
```

Since a view is a function that takes the `terms` object as argument, you can make it dynamic. For example, here's how you would show all posts upvoted by a user:

```js
Posts.views.add("userUpvotedPosts", function (terms) {
  return {
    selector: {
      upvoters: {$in: [terms.userId]},
    },
    options: {
      sort: {
        postedAt: -1
      }
    }
  };
});
```

Note that views currently only work with the `Posts` collection, but this might change in the future. 

### On The Client

Although the primary function of the `terms` object is to figure out the `selector` and `options` to use on the server to fetch data from the database, it's also used on the client to make sure data is kept in a logical state as it gets updated. 

For example, let's imagine you're viewing posts filtered by a specific category. If you were to edit one of these posts to remove its category, we need to make sure the post is removed from the list since it doesn't match the inclusion criteria anymore. 

Instead of refetching the whole list from the server, we can do this transparently using [Mingo](https://github.com/kofrasa/mingo) to resort the data on the client using the same `selector` and `options` objects. 