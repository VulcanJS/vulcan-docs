---
title: Filtering, Sorting & Selecting
---

When loading data, you will usually want to apply some kind of filtering or sorting to your query. Vulcan offers the following tools to help you get the data you need. 

Note that in all following examples, you can only use **database** fields, not GraphQL-only fields. 

## Selecting

Vulcan queries and mutations take a `where` argument to help you target one or more specific documents.

The `where` argument can either accept a list of fields; or special `_and`, `_not`, and `_or` operators used to combine multiple `where` selectors. 

Each field can then in turn receive an operator such as `_eq` or `_in` depending on its type. Note that this API was heavily inspired by the [Hasura](https://docs.hasura.io/1.0/graphql/manual/queries/query-filters.html) API. 

Here is an example `MovieWhereInput` input type:

```
input MovieWhereInput {
  _and: [MovieWhereInput]
  _not: MovieWhereInput
  _or: [MovieWhereInput]
  _id: String_Selector
  createdAt: Date_Selector
  userId: String_Selector
  slug: String_Selector
  title: String_Selector
}
```

And here is an example query using `where`: 

```
query MyMovie {
  movie(input: { where: { _id: { _eq: "123foo" } } }) {
    result{
      _id
      title
      year
    }
  }
}
```

## Ordering

```
query RecentMovies {
  movies(input: { where: { year: { gte: 2010" } }, orderBy: { year: "desc" } }) {
    results{
      _id
      title
      year
    }
  }
}
```

## Limiting

```
query RecentMovies {
  movies(input: { where: { year: { gte: 2010" } }, orderBy: { year: "desc" }, limit: 10 }) {
    results{
      _id
      title
      year
    }
  }
}
```

## Searching

In some cases you'll want to select data based on a field value, but without knowing exactly which field to search in. While you can build a complex query that lists every field needing to be searched, Vulcan also offers a shortcut in the form of the `search` argument: 

```
query FightMovies {
  movies(input: { search: "fight" ) {
    results{
      _id
      title
      year
      description
    }
  }
}
```

On the server, Vulcan will search any field marked as `searchable: true` in its collection's schema for the string `fight` and will return the result. 

## Filters

There are also cases where your query is too complex to be able to define it through the GraphQL API. For example, you could imagine a scenario where you want to only show movies that have five-star reviews.

In this case, you would define a server-side “filter”, and then reference it in your query:

```
query FiveStarsdMovies {
  movies(input: { filter: "fiveStars" ) {
    results{
      _id
      title
      year
      description
    }
  }
}
```

The filter function takes the query `input` and `context` as arguments; and should return an object with `selector` and `options` properties. Here's how you would define it:

```
const Movies = createCollection({

  collectionName: 'Movies',

  typeName: 'Movie',

  schema,

  resolvers: getDefaultResolvers({ typeName: 'Movie' }),

  mutations: getDefaultMutations({ typeName: 'Movie' }),

  filters: {
    fiveStars: (input, context) => {
      const { Reviews } = context;
      const fiveStarReviews = Reviews.find({ stars: { $gte: 5 }}).fetch();
      const fiveStarReviewsMoviesIds = fiveStarReviews.map(review => review.movieId);
      return {
        selector: { _id: {$in: fiveStarReviewsMoviesIds } },
        options: {}
      }
    };
  },

});
```

## Default Input

The `defaultInput` property lets you control what happens when no argument at all is provided to a query:

```
const Movies = createCollection({
  collectionName: 'Movies',

  typeName: 'Movie',

  //...

  defaultInput: {
    orderBy: {
      createdAt: 'desc'
    },
  },

});
```