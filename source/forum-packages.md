---
title: Forum Packages
---

Here's an overview of the various forum packages included with Vulcan. 

Note that most of these packages currently require the `vulcan:posts` package. Eventually, the goal is to generalize them so they can work with any Vulcan collection. 

## Posts

- Dependencies: `vulcan:core`, `vulcan:events`

The Vulcan Posts package.

## Comments

- Dependencies: `vulcan:core`, `vulcan:posts`

The Vulcan Comments package.

## Notifications

- Dependencies: `vulcan:core`, `vulcan:email`

Email notifications. 

## Voting

- Dependencies: `vulcan:core`, `vulcan:posts`

Enables upvoting and downvoting for the Posts and Comments collections. 

## Embedly

- Dependencies: `vulcan:core`, `vulcan:posts`

Use [Embedly](http://embed.ly) to load extra metadata for posts. 

## Categories

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`

Posts categories. 

## API

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`

Create a JSON API for posts and comments. 

## RSS

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`

Create RSS feeds for posts and comments. 

## Newsletter

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:categories`, `vulcan:email`

Generate and schedule a MailChimp newsletter for posts and comments. 

## Email Templates

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:email`

Templates for email notifications and the newsletter.

## Getting Started

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:events`

Dummy content for posts, comments, and users. 