---
title: Form Components
---

## General Components

The following components are used to display Vulcan forms.

#### `SmartForm`

This component (`FormWrapper.jsx`) wraps the entire form and handles all data loading.

#### `Form`

This is the main component responsible for generating and displaying the form.

#### `FormErrors`

Used to display errors at the top of the form.

#### `FormGroup`

Used to display form groups.

#### `FormSubmit`

Used to display the form's submit and cancel buttons.

#### `FieldErrors`

Used to display errors beneath a form field.

#### `FormComponent`

Used to display an individual form field.

#### `FormNested` & `FormNestedItem`

Used to display nested form items along with add/remove controls.

## Field Components

In addition to the components used to display the form's structure, an additional set of components is used to display each individual field, according to its type.

To select a field, use its lowercase name as the `control` property on your schema (e.g. `control: 'select'`).

#### `Default`

Default form input, for text strings.

#### `Textarea`

Textarea form input. 

#### `Email`

Email form input.

#### `Number`

Number form input. Will be used automatically for `Number` fields.

#### `Url`

URL form input. 

#### `Checkbox`

Checkbox form input. Will be used automatically for `Boolean` fields.

#### `Checkboxgroup`

Checkbox form input.

#### `Select`

Select form input.

#### `Radiogroup`

Radio group form input

#### `Date`, `Time`, `Datetime`

Used to select a date, a time, or both. `Date` will be used automatically for `Date` fields.