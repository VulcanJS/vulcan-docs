---
title: Datatable
---

### Datatable

The `Datatable` component is used to show a dynamic datatable for a given collection. It takes the following props:

- `collection`: the collection to load data from.
- `columns`: an array containing a list of columns (see below).
- `options`: the `options` object passed to `withMulti` in order to load the data (optional).
- `showEdit`: if `true`, will add a column with an Edit button at the end of each row (defaults to `false`).

The `columns` array is an array of either column names, objects, or both. If you're passing objects, each one should contain the following properties:

- `name`: the name of the column.
- `order`: the order of the column within the table (optional).
- `component`: a custom component used to render this column's table cells (optional).

For example:

```js
<Components.Datatable 
  collection={Rooms} 
  columns={['name', 'description']} 
/>
```

Or:

```js
const columns = [
  'name',
  {
    name: 'email',
    order: 10,
    component: AdminUsersEmail
  },
  'createdAt',
  {
    name: 'actions',
    order: 100,
    component: AdminUsersActions
  },
];

<Components.Datatable 
  collection={Users} 
  columns={columns} 
  options={{
    fragmentName: 'UsersAdmin',
    terms: {view: 'usersAdmin'},
    limit: 20
  }}
/>
```

Note that columns do not necessarily have to correspond to schema fields. For example, you could add an `actions` column to each rows even though there is no `actions` field in your collection schema.
