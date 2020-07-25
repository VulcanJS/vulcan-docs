---
title: Relations
---

A common use case for field resolvers is fetching one or more item associated with the current document, such as the user corresponding to a post's `userId` field; or the array of categories correponding to its `categoriesIds` field. 

While you can achieve this by explicitly adding an API schema field, Vulcan also offers a shortcut syntax using the `relation` property:

```
userId: {
  type: String,
  optional: true,
  canRead: ['guests'],
  relation: {
    fieldName: 'user',
    typeName: 'User',
    kind: 'hasOne', // one of 'hasOne' or 'hasMany'
  }
},
```

Since this is a `hasOne` relation, Vulcan will assume that the current field stores the `_id` of the related document to look up; and because we know that the type of the returned item should be `User` we can also easily figure out that we need to search the `Users` collection.

Similarly, you can also define `hasMany` relations:

```
categoriesIds: {
  type: Array,
  arrayItem: {
    type: String,
    optional: true,
    canRead: ['guests'],
  }
  optional: true,
  canRead: ['guests'],
  relation: {
    fieldName: 'categories',
    typeName: '[Categories]',
    kind: 'hasMany'
  }
},
```

## Relations, Cards, and Datatables

As an added bonus, if you define the `userId` field as having a relation to the `User` type, Cards and Datatables will be able to understand this relation and use smarter formatting when displaying this data. 
