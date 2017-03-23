---
title: Features Packages
---

Here's an overview of the various features packages included with Vulcan. 

## Events

- Dependencies: `vulcan:core`

Simple internal event tracking. 

#### `sendGoogleAnalyticsRequest()`

Log a Google Analytics request for the current page. 

#### `requestAnalyticsAsync(hook, document, user)`

Log a Segment request. 

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

## Debug

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:email`

Add some special debugging routes. 

## Base Components

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:voting`

A theme that includes a set of basic React components and routes for posts and comments. 

## Base Styles

- Dependencies: `vulcan:core`

Includes styles for the `vulcan:base-components` package as well as the main Bootstrap 4 stylesheet.

## Newsletter

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:categories`, `vulcan:email`

Generate and schedule a MailChimp newsletter for posts and comments. 

## Email Templates

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:email`

Templates for email notifications and the newsletter.

## Getting Started

- Dependencies: `vulcan:core`, `vulcan:posts`, `vulcan:comments`, `vulcan:events`

Dummy content for posts, comments, and users. 