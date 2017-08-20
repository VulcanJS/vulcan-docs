---
title: Admin Dashboard
---

The `vulcan:admin` package provides a user moderation dashboard accessible to admin users at `/admin`.

### Adding Columns

You can extend the admin dashboard with your own custom columns. First, you'll need to make the relevent data available to the client by adding it to the `UsersAdmin` fragment:

```js
import { extendFragment } from 'meteor/vulcan:core';

extendFragment('UsersAdmin', `
  posts{
    ...PostsPage
  }
`);
```

Then, you can create a component for the dashboard cell item:

```js
import React from 'react';
import Posts from 'meteor/vulcan:posts';
import { Link } from 'react-router';

const AdminUsersPosts = ({ user }) => 
  <ul>
    {user.posts && user.posts.map(post => 
      <li key={post._id}><Link to={Posts.getLink(post)}>{post.title}</Link></li>
    )}
  </ul>

export default AdminUsersPosts;
```

And finally add the component to the dashboard as a new column using the `addAdminColumn` function:

```js
import { addAdminColumn } from 'meteor/vulcan:core';
import AdminUsersPosts from './components/AdminUsersPosts';

addAdminColumn({
  name: 'users.posts',
  order: 50,
  component: AdminUsersPosts
});
```