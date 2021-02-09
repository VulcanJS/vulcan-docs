---
title: Datatable
---

The `Datatable` component is used to show a dynamic datatable for a given collection. 

## Datatable Props

### Required Props

It takes the following props:

- `collection` or `collectionName`: the collection object to load data from (or its name as a string).
- `data`: alternatively, you can pass a raw JSON data array as the `data` prop. But note that in that case, many features (such as sorting, filtering, etc.) will not be available.

### Optional Props

- `columns`: an array containing a list of columns (see below). If not `column` prop is passed, the datatable will use the keys of the first item as column headings. 
- `options`: the `options` object passed to `withMulti` in order to load the data.
- `showNew`: if `true`, will add a New Document button above the table (defaults to `true`).
- `showEdit`: if `true`, will add a column with an Edit Document button at the end of each row (defaults to `true`).
- `showSearch`: if `true`, will add a search field above the datatable (defaults to `true`).
- `newFormProps`: options passed to the “New Document” form.
- `editFormProps`: options passed to the Edit Document” forms.
- `Components`: the `Components` object lets you override child components of a specific datatable.
- `rowClass`: a string containing a CSS class applied to all rows; or a function that takes in the document corresponding to a table row and returns a string.
- `initialState`: an object containing the initial filtering/sorting state of the datatable.
- `useUrlState`: whether to store the datatable state in the URL. Defaults to `true`, but can be set to `false` when e.g. you need to handle multiple datatables on the same page.

## Column Items

The `columns` array is an array that describes which collection fields should be included in the datatable. It can contain either column names as strings (which should match the names of the document fields), objects, or both. 

Note that columns do not necessarily have to correspond to schema fields. For example, you could add an `actions` column to each rows even though there is no `actions` field in your collection schema.

### Required Props

- `name`: the name of the field to display in the column.

### Optional Props

- `order`: the order of the column within the table.
- `component`: a custom component used to render this column's table cells (see below).
- `label`: a label for the column (if not provided, the formatted `name` will be used).
- `sortable`: if `true`, sorting options will be shown next to the column header.
- `filterable`: if `true`, filtering options will be shown next to the column header.
- `filterComponent`: a component used to customize the contents of a column filter. See below.

### Custom Components

Custom cell components receive the following props:

- `column`: the current column.
- `document`: the current document.
- `collection`: the current collection.
- `Components`: the datatable's `Components` prop.

You can use custom components to control how the inside of a table cell is rendered. This can have many uses, such as displaying field `Bar` in field `Foo`'s column:

```
columns=[{
  name: 'startAt',
  component: ({ document }) => <span>{document.startAtFormattedShort}</span>,
}]
```

## Filtering

If a column is marked as `filterable: true`, a filter icon will automatically appear next to its header. The filter type depends on the column:

1. If a custom filter component is passed, this is what will be used.
2. Else, if the corresponding schema field has an `options` property, those will be used to populate a `CheckboxGroup` filter. 
3. If there are no `options` and the field is a `Date` or `Number` field, corresponding greater than/lesser than filters will be shown. 
4. If field is a string, a search field will be shown [TODO].

### Filter Query/Options

Filters will use a field's `options` property to populate a list of checkboxes. Alternatively, if a GraphQL `query` is defined on the field (in which case `options` should be a function that takes in the resulting `data` and returns a set of `options`), the filter will load the list of options from the database on the fly. 

The `options` property should be an array of `{ label, value }` objects (although when using custom filter components, you can add more properties as well if you need to).

### Filter Components

Filter custom components receive the following props:

- `name`: the field/column name.
- `options`: the corresponding field's `options` (used to populate checkboxes, selects, etc.).
- `filters`: active filters for the column. Of the form `{ operator: filterArray}`, e.g. `{ _in: ['public', 'archived']}`.
- `setFilters`: a function called to set the local state of the filter. Takes a single object as argument in the same format as `filters. 
- `submitFilters`: a function called to submit the local state of the filter back to the datatable. Takes a single object as argument in the same format as `filters` (generally not used). 

Note that the `filterComponent` only replaces the “inside” of the filter. The reset and submit buttons are not replaced, meaning your custom filter should generally only need to deal with setting the filter's internal state with `setFilters`, and not need to use `submitFilters`.

### Example

```js
const RoomIdFilter = ({ field, options, filters = { [checkboxOperator]: [] }, setFilters }) => {
  let value = filters[checkboxOperator];

  const formattedOptions = options.map(({ value, label, image }) => ({
    value,
    label: (
      <span booking-dashboard-roomid-filter-item>
        <img className="booking-dashboard-roomid-filter-image" src={image} />
        {label}
      </span>
    ),
  }));

  return (
    <div className="booking-dashboard-roomid-filter">
      <Components.FormComponentCheckboxGroup
        path="filter"
        itemProperties={{ layout: 'inputOnly' }}
        inputProperties={{ options: formattedOptions }}
        value={value}
        updateCurrentValues={({ filter: newValues }) => {
          setFilters({ [checkboxOperator]: newValues });
        }}
      />
    </div>
  );
};
```

## Examples

### Minimal Example

```js
<Components.Datatable 
  collectionName="Posts" 
/>
```

### Basic Example

```js
<Components.Datatable 
  collectionName="Posts"
  columns={[
    'title',
    { name: 'createdAt', sortable: true },
    'contents',
    { name: 'status', filterable: true }
  ]}
  showNew={true}
  showEdit={true}
  showSearch={true}
/>
```

### Complex Example

```js
<Components.Datatable
  collectionName="Bookings"
  options={{
    fragmentName: 'BookingFragment',
  }}
  initialState={{
    sort: {
      createdAt: 'desc',
    },
    filters: {
      status: {
        _in: [1, 3, 5],
      },
    },
  }}
  columns={[
    {
      name: 'numberOfNights',
      label: 'Infos',
      component: BookingInfos,
    },
    {
      name: 'createdAt',
      component: BookingCreatedAt,
      sortable: true,
      filterable: true,
    },
    {
      name: 'startAt',
      sortable: true,
      filterable: true,
      component: ({ document }) => <span>{document.startAtFormattedShort}</span>,
    },
    {
      name: 'endAt',
      sortable: true,
      filterable: true,
      component: ({ document }) => <span>{document.endAtFormattedShort}</span>,
    },
    {
      name: 'status',
      component: BookingStatus,
      filterable: true,
      filterComponent: RoomStatusFilter,
    },
  ]}
  showNew={Users.isAdmin(currentUser)}
  showEdit={Users.isAdmin(currentUser)}
  newFormOptions={{
    removeFields: [
      'source',
      'status',
      'numberOfGuests',
    ],
  }}
  rowClass={document => `status-${getBookingStatusLabel(document.status)}`}
/>
```
