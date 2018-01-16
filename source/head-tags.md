---
title: Head Tags
---

You'll often need to add tags to your app's `<head>` section. There are three ways to do this: using the [Helmet](https://github.com/nfl/react-helmet) library, using the `<Components.HeadTags/>` component, or using the `Head` object. 

## Helmet

You can use Helmet inside a VulcanJS app just like inside any other React app:

```js
const Layout = ({children}) =>

  <div id='wrapper'>

    <Helmet>
      <link name='bootstrap' rel='stylesheet' type='text/css' href='https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.5/css/bootstrap.min.css'/>
      <link name='font-awesome' rel='stylesheet' type='text/css' href='https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css'/>
    </Helmet>
    
    <div className='main'>

      {children}

    </div>
  
  </div>

registerComponent('Layout', Layout);
```

## Components.HeadTags

### Using HeadTags

Out of the box, a VulcanJS app adds a few default tags such as `<title/>`, `<meta name='description'/>`, etc. These are all added using Helmet by `Components.HeadTags` (called from inside the `App` component), using values provided by your `settings.json` file. 

In some cases, you'll want to change these values for a given page. For example, a single post page will want to have a different title than your homepage. You can do that by calling `Components.HeadTags` directly anywhere inside your component tree  with one or more of the following props:

- `title`
- `url`
- `image`
- `description`

```js
<Components.HeadTags url={Posts.getPageUrl(post, true)} title={post.title} image={post.thumbnailUrl} description={post.excerpt} />
 ```

By calling `HeadTags` again, you'll be calling Helmet again, which in turn will override the head tags previously set by the `App` component.

### Overriding Headtags

Since `Components.HeadTags` is a component, you can also override it completely using `replaceComponent('HeadTags', MyComponent)`.

## The Head Object

Finally, there are also some cases where you want to add or remove a tag from *outside* your React components. You can do so using the `Head` object.

For example. here is how the `vulcan:rss` package adds an RSS feed link to the head:

```js
import { Head, Utils } from 'meteor/vulcan:core';

Head.link.push({
  name: 'rss',
  rel: 'alternate', 
  type: 'application/rss+xml',
  href: `${Utils.getSiteUrl()}feed.xml`
});
```

You can also remove a tag using `removeFromHeadTags`:

```js
import { removeFromHeadTags } from 'meteor/vulcan:core';

removeFromHeadTags('link', 'rss');
```

### Head Components

You can also use the `Head` object to add React components to every page of your app (this can be useful for adding analytics integrations wrapped inside React components, for example). Just push the components to `Head.components`. 

If you push an array (such as `[myComponent, withCurrentUser]`) the array will be interpreted as a component + HoC array similar to the arguments of `registerComponent`. 