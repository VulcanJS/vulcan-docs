---
title: Forum Example
---

This packages enables the original Telescope forum features and components. 

Note that it doesn't actually contain any code by itself, but instead activates the [forum packages](packages.html) package set.

## Settings

| Setting | Example | Description |
| --- | --- | --- |
| enableNotifications | true | Whether to enable email notifications |
| postInterval | 20 | How long, in seconds, to make users wait between submitting posts |
| commentInterval | 20 | How long, in seconds, to make users wait between commenting |
| maxPostsPerDay | 10 | How many posts a user can submit per day |
| RSSLinksPointTo | "link"/"page" | Whether to point RSS links to the linked site or back to your own site |
| postsPerPage | 10 | How many posts to display per page |
| thumbnailWidth | 800 | Posts thumbnails width |
| thumbnailHeight | 600 | Posts thumbnails height |
| scoreUpdateInterval | 30 | How often, in seconds, to update posts scores. Set to `0` to disable score updating |

## Categories

You can prefill categories from your `settings.json` file:

```js
"categories": [
  {
    "name": "Test Category 1",
    "description": "The first test category",
    "order": 4,
    "slug": "testcat1"
  },
  {
    "name": "Test Category 2",
    "description": "The second test category",
    "order": 7,
    "slug": "testcat2"
  },
  {
    "name": "Test Category 3",
    "description": "The third test category",
    "order": 10,
    "slug": "testcat3"
  }
],
```