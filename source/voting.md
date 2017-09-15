---
title: Voting
---

This package adds voting features to a collection.

## Make Voteable

```js
import { makeVoteable } from 'meteor/vulcan:voting';
import Posts from './posts/index.js';

makeVoteable(Posts);
```

Calling `makeVoteable` on a collection does the following things:

- Add the collection to a `VoteableCollections` array.
- Add a set of custom fields to the collection schema (see below).

## Custom Fields

- `upvotes`
- `upvoters`
- `downvotes`
- `downvoters`
- `score`
- `baseScore`
- `inactive`

## Mutations

Every collection added to the `VoteableCollections` array gets added to a `Voteable` union type, which in turn gets returned by the `vote` mutation:

```
vote(documentId: String, voteType: String, collectionName: String) : Voteable
```

## Container

The Voting package exports the `withVote` container that gives access to the `vote` mutation. You can then call the mutation like this (note that this example only implements upvoting, not downvoting):

```js
import { Components, registerComponent, withMessages } from 'meteor/vulcan:core';
import React, { PureComponent } from 'react';
import PropTypes from 'prop-types';
import classNames from 'classnames';
import { withVote, hasUpvoted, hasDownvoted } from 'meteor/vulcan:voting';

class Vote extends PureComponent {

  constructor() {
    super();
    this.upvote = this.upvote.bind(this);
    this.state = {
      loading: false
    }
  }

  upvote(e) {
    e.preventDefault();

    const document = this.props.document;
    const collection = this.props.collection;
    const user = this.props.currentUser;

    if(!user){
      this.props.flash('Please log in first!');
    } else {
      const voteType = hasUpvoted(user, document) ? 'cancelUpvote' : 'upvote';
      this.props.vote({document, voteType, collection, currentUser: this.props.currentUser});
    } 
  }

  render() {
    return (
      <div>
        <a className="upvote-button" onClick={this.upvote}>
          {this.state.loading ? <Components.Icon name="spinner" /> : <Components.Icon name="upvote" /> }
          <div className="sr-only">Upvote</div>
          <div className="vote-count">{this.props.document.baseScore || 0}</div>
        </a>
      </div>
    )
  }

}

registerComponent('Vote', Vote, withMessages, withVote);
```

## Callbacks

You can make use of the voting package through callbacks as well. For example, here's how to make every new post be upvoted by its user on creation: 

```
import Posts from 'meteor/example-forum';
import Users from 'meteor/vulcan:users';
import { addCallback } from 'meteor/vulcan:core';
import { operateOnItem } from 'meteor/vulcan:voting';

function PostsNewUpvoteOwnPost(post) {
  var postAuthor = Users.findOne(post.userId);
  return {...post, ...operateOnItem(Posts, post, postAuthor, 'upvote', false)};
}

addCallback('posts.new.sync', PostsNewUpvoteOwnPost);
```