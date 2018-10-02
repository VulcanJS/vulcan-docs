---
title: Customization Example
---

Vulcan was made to be extensible from the ground up. This means that you can tweak and adapt many existing features to your needs without having to rewrite them from scratch. 

In this tutorial, we'll look at the `example-customization` code file by file to see how we can easily extend Vulcan with minimal work. 

Note that we're mainly talking about customizing [forum packages](packages.html) here.

## Intro

<iframe width="560" height="315" src="https://www.youtube.com/embed/PymWJnXOYMY" frameborder="0" allowfullscreen></iframe>

## Custom Packages

In Vulcan, everything is a package, so the first thing you'll need to do is to create a new directory in `/packages` to hold your code, and a `package.js` file inside that:

```js
Package.describe({
  name: "my-custom-package"
});

Package.onUse( function(api) {

  api.versionsFrom("METEOR@1.0");

  api.use([
    'fourseven:scss',

    'vulcan:core',
    'vulcan:',
    'vulcan:posts',
    'vulcan:users'
  ]);

  api.mainModule('server.js', 'server');
  api.mainModule('client.js', 'client');

  api.addFiles([
    'lib/stylesheets/custom.scss'
  ], ['client']);
  
  api.addAssets([
    'lib/server/emails/customNewPost.handlebars',
    'lib/server/emails/customEmail.handlebars'
  ], ['server']);

});
```

Let's go through this file block by block.

- `Package.describe` is used to set your package's metadata. Only the `name` is required, but you can also add a version, Git repository, etc.
- `Package.onUse` contains the code used to load our package. 
- `api.use` declares the package's dependencies on other Meteor packages. These dependencies can be either local (living in `/packages` like `vulcan:*`) or remote (pulled from Meteor's package server like `fourseven:scss`).
- `api.mainModule` is used to declare your package's client and server entry points and should be used to load JavaScript files. 
- `api.addFiles` is used to add files one by one, usually HTML or CSS. 
- `api.addAssets` is used to add static assets to your package. Basically, anything that's not a JavaScript, HTML, or CSS file. Make sure to declare the files for client side `['client']` otherwise you might not see these files. images should come from `\packages\<package-name>\static\` the "static" folder name is where you store your images, you can rename this to whatever you want. Beware of storing static assets in git. More info can be found in Meteor's docs regarding [`Assets`](https://docs.meteor.com/api/assets.html).

## Custom Styles

The simplest and most common customization is probably tweaking your styles. Thanks to Meteor's automatic CSS packaging features, all you need to do is point your custom package to a style file.

```js
api.addFiles([
    'lib/stylesheets/custom.scss'
  ], ['client']);
```

And since we're using the `fourseven:scss` package, we can use a SCSS file:

```css
.logo-hello {
  display: flex;
  justify-content: center;
  color: #333;
}

.post-red{
  background-color: #FFD3D2;
}
.post-blue{
  background-color: #D2F5FF;
}
.post-yellow{
  background-color: #FBFFD2;
}
.post-green{
  background-color: #D2FFD2;
}
```

Protip: a good way to make sure that your CSS file is getting properly bundled up and applied is to include something very visible like `body{ background: red; }`.

## Replacing Components

A really cool feature of Vulcan's theming system is the ability to replace a specific component without having to overwrite the entire theme.

For example, let's consider the Forum example's `Logo` component:

```js
import React from 'react';
import { IndexLink } from 'react-router';

const Logo = ({logoUrl, siteTitle}) => {
  if (logoUrl) {
    return (
      <h1 className="logo-image ">
        <IndexLink to={ { pathname: "/" } }>
          <img src={logoUrl} alt={siteTitle} style={ { maxWidth: "100px", maxHeight: "100px" } } />
        </IndexLink>
      </h1>
    )
  } else {
    return (
      <h1 className="logo-text">
        <IndexLink to={ { pathname: "/" } }>{siteTitle}</IndexLink>
      </h1>
    )
  }
}

Logo.displayName = "Logo";

registerComponent('Logo', Logo);
```

We'll replace it with a new component that shows a welcome message next to the logo:

```js
import React from 'react';
import { IndexLink } from 'react-router';
import { withCurrentUser, replaceComponent } from 'meteor/vulcan:core';
import Users from 'meteor/vulcan:users';

const CustomLogo = ({logoUrl, siteTitle, currentUser}) => {
  return (
    <div>
      <h1 className="logo-text"><IndexLink to="/">⭐{siteTitle}⭐</IndexLink></h1>
      { currentUser ? <span className="logo-hello">Welcome {Users.getDisplayName(currentUser)} 👋</span> : null}
    </div>
  )
}

replaceComponent('Logo', CustomLogo, withCurrentUser);
```

Note how we're adding the `withCurrentUser` higher-order component to give our new `CustomLogo` component access to the `currentUser` prop.

All that's needed now is to import this file from `modules.js`:

```js
import "./components/CustomLogo.jsx";
```

### Extending Components

Replacing the entire component works for simple components, but sometimes you only want to replace one part of the component (typically its `render` method) while preserving others.

Take a look look at the `CustomNewsletter` component to see an example of doing just that:

```js
import { Components, replaceComponent, getRawComponent } from 'meteor/vulcan:core';
import React, { PropTypes, Component } from 'react';
import { FormattedMessage, intlShape } from 'react-intl';

class CustomNewsletter extends getRawComponent('Newsletter') {

  render() {
    // console.log(this.renderButton); <-- exists
    
    return this.state.showBanner
      ? (
        <div className="newsletter">
          <h4 className="newsletter-tagline">✉️<FormattedMessage id="newsletter.subscribe_prompt"/>✉️</h4>
          {this.props.currentUser ? this.renderButton() : this.renderForm()}
          <a onClick={this.dismissBanner} className="newsletter-close"><Components.Icon name="close"/></a>
        </div>
      ) : null;
  }

}

replaceComponent('Newsletter', CustomNewsletter);
```

It's interesting to note that we're using the `extend` syntax to extend the original component, in order to inherit from all its methods. We're extending `getRawComponent('Newsletter')` and not just `Components.Newsletter` to make sure we extend the *original* component before it gets wrapped with any HoCs such as `withCurrentUser`, `withRouter`, etc. 

What's more, the `replaceComponent` method will make sure to preserve the original component's HoCs and re-wrap our new custom component with them.

By extending the component, we're able to only redefine the `render` method while preserving all the others (in this case `constructor`, `componentWillReceiveProps`, etc.).

You can also look at the `CustomPostItem` component to see another example of extending components.

You can learn more about replacing and extending components in the [Components & Theming](theming.html) section. 

## Strings & Internationalization

Another common need is changing the wording of a specific string. Vulcan uses the [react-intl](https://github.com/yahoo/react-intl) library to make every string in the default components customizable and translatable.

```js
import { addStrings } from 'meteor/vulcan:core';

addStrings('en', {
  "posts.color": "Color" // add a new one (collection.field: "Label")
});
```

You can learn more about translating strings in the [Internationalization](internationalization.html) section. 

## Custom Fields

Custom fields let you add extra properties to predefined collections like `Posts` or `Comments`. In the `custom_fields.js` file, we're adding a new `color` property to posts, and making it insertable, editable, and viewable by any member of the `members` user group (in other words, regular users):

```js
import Posts from "meteor/vulcan:posts";

Posts.addField(
  {
    fieldName: 'color',
    fieldSchema: {
      type: String,
      control: 'select', // use a select form control
      optional: true, // this field is not required
      canCreate: ['members'],
      canUpdate: ['members'],
      canRead: ['members'],
      form: {
        options: function () { // options for the select form control
          return [
            {value: "white", label: "White"},
            {value: "yellow", label: "Yellow"},
            {value: "blue", label: "Blue"},
            {value: "red", label: "Red"},
            {value: "green", label: "Green"}
          ];
        }
      },
    }
  }
);
```

We'll then extend the `PostsItem` component to add a CSS class to the `posts-item` div based on this color field:

```js
import { replaceComponent, getRawComponent } from 'meteor/vulcan:core';

class CustomPostsItem extends getRawComponent('PostsItem') {

  render() {

    const post = this.props.post;

    let postClass = "posts-item"; 
    if (post.sticky) postClass += " posts-sticky";

    // ⭐ custom code starts here ⭐
    if (post.color) {
      postClass += " post-"+post.color;
    }
    // ⭐ custom code ends here ⭐

    return (
      <div className={postClass}>
        ...
      </div>
    )
  }
};
  
CustomPostsItem.propTypes = {
  currentUser: React.PropTypes.object,
  post: React.PropTypes.object.isRequired
};

CustomPostsItem.fragment = gql`
  fragment PostsItemFragment on Post {
    _id
    title
    url
    color // ⭐ our new color property ⭐
    ...
  }
`;

replaceComponent('PostsItem', CustomPostsItem);
```

Don't forget to also add the `color` property to the fragment that governs what data we query for. 

Learn more about adding your own custom fields in the [Custom Fields](custom-fields.html) section. 

## Callbacks

Callbacks let you insert your own server-side logic at key points in Vulcan's code. For example, the `callbacks.js` file shows you how you can insert random emojis in a post title whenever a new post is inserted:

```js
import { addCallback } from 'meteor/vulcan:core';

function PostsNewAddRandomEmoji (post, user) {

  post.title = post.title + " " +_.sample(["🎉", "💎", "☠", "⏱", "🎈", "⛱"])

  return post;
}
addCallback("posts.new.sync", PostsNewAddRandomEmoji);
```

You can learn more about callbacks in the [Callbacks](callbacks.html) section. 

## Groups

Out of the box, Vulcan considers three kind of users: guests (users without an account), members (users with a normal account), and admins. Let's add a fourth kind, mods, in `groups.js`:

```js
import Users from 'meteor/vulcan:users';

Users.createGroup("mods");

Users.groups.mods.can("posts.edit.all"); // mods can edit anybody's posts
Users.groups.mods.can("posts.remove.all"); // mods can delete anybody's posts
```

We'll give mods two extra actions: editing and removing other user's posts. Note that like everybody else, mods are also considered part of the `default` group so they inherit all the default permissions too. 

You can learn more about user groups in the [Groups & Permissions](groups-permissions.html) section. 

## Routes

Adding a new route is easy enough. After creating a new `MyCustomPage` component we  just add the following code in `routes.js`: 

```js
import { addRoute } from 'meteor/vulcan:core';
import MyCustomPage from './components/MyCustomPage.jsx';

addRoute({name:"myCustomRoute", path:"/my-custom-route", component:MyCustomPage});
```

You can learn more about routes in the [Routing](routing.html) section. 

## Emails & Email Templates

Finally, we'll also customize the email notification that gets sent out when a user creates a new post, as well as add a new email template. This is a three-step process.

First, we need to create the new templates (`customEmail.handlebars` and `customNewPost.handlebars` in `lib/server/emails`) using the [Handlebars](http://handlebarsjs.com/) templating language.

Then, we register these two files as **assets** in `package.js`:

```js
api.addAssets([
  'lib/server/emails/customNewPost.handlebars',
  'lib/server/emails/customEmail.handlebars'
], ['server']);
```

Finally, we retrieve these two assets and assign them to email templates in `lib/server/templates.js`:

```js
import VulcanEmail from 'meteor/vulcan:email';

VulcanEmail.addTemplates({
  newPost: Assets.getText("lib/server/emails/customNewPost.handlebars"),
  customEmail: Assets.getText("lib/server/emails/customEmail.handlebars")
});
```

You can learn more about customizing emails in the [Email](email.html) section. 

## Conclusion

By going through this sample package's code, you've learned how to customize most aspects of Vulcan without having to modify any of Vulcan's original code. 

You can now use that newfound knowledge to build your app, or even create new themes and plugins to add to the Vulcan ecosystem. The rest is up to you!
