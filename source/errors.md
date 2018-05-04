---
title: Errors & Messages
---

## Error Properties

An error object can have the following properties (all optional):

- `id`: a string identifier, for example `app.not_found`. Can be used as an internationalization key.
- `path`: for field-specific errors inside forms, the path of the field with the issue.
- `properties`: additional data. Will be passed to vulcan-i18n as values.
- `message`: if `id` cannot be used as an i81n key, `message` will be used instead.
- `type`: one of `error`, `warning`, `success`. Can be used to visually differentiate between message types. 
- `break`: a Boolean, whether to interrupt the current operation or try to keep going.
- `errors`: if the error object itself contains an `errors` array, it will be interpreted as a "multi-error" that contains multiple related sub-errors. 

## GraphQL Errors

When an error is thrown during a GraphQL query or mutation, it can be a good idea to use [Apollo-Errors](https://github.com/thebigredgeek/apollo-errors) instead of "regular" Node errors. This lets you build more complex errors and transmit more data back to the client. 

```js
import { createError } from 'apollo-errors';

const FooError = createError('FooError', { message: 'A foo error has occurred' });

throw new FooError({ data: { something: 'important' } });
```

For GraphQL errors, all the error properties listed above should be assigned on the `data` object (including `message`).

For example, here is how you could throw an error when trying to geocode an address inside a mutation callback:

```js
const message = `Could not geocode address “${address}”`;
const GeoDataError = createError('app.geoData_error', { message });
throw new GeoDataError({
  data: {
    break: false,
    id: 'app.geoData_error',
    path: 'address',
    properties: { address },
    message,
  },
});
```

Note that if `app.geoData_error` has a corresponding internationalization string, the string will be used (and `{ address }` passed as `values` usable inside the string); and if it does not `message` will be used instead. 

## Parsing Errors

When catching a server error as a result of an operation, you can call `getErrors()` to retrieve a properly formatted array of GraphQL (or regular) errors. 

## Displaying Errors

Inside your Vulcan app, errors can be displayed using the `withMessages` HoC, which gives a component access to the `flash()` function to show a message. 

## Form Errors

Form errors are a special use case, and are thrown from inside a form component using `this.props.throwError()`. 

Note that when throwing a GraphQL error meant to be displayed inside a form, you can give it a `path` property to relate it to a specific form field. 