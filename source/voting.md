---
title: Voting
---

This package adds voting features to a collection.

## Vote Schema

An individual vote document is defined according to the following schema:

```js
const schema = {

  _id: {
    type: String,
    canRead: ['guests'],
  },

  /**
    The id of the document that was voted on
  */
  documentId: {
    type: String
  },

  /**
    The name of the collection the document belongs to
  */
  collectionName: {
    type: String
  },

  /**
    The id of the user that voted
  */
  userId: {
    type: String
  },

  /**
    The vote type (upvote, downvote, Facebook-style reactions, etc.)
  */
  voteType: {
    type: String,
    optional: true
  },

  /**
    The vote power (e.g. 1 = upvote, -1 = downvote, or any other value)
  */
  power: {
    type: Number,
    optional: true
  },
  
  /**
    The vote timestamp
  */
  votedAt: {
    type: Date,
    optional: true
  }

};
```

Two important properties are:

- `voteType`: a name such as `upvote` or `downvote`. It can also be used for Facebook-style reactions such as `happy`, `sad`, or even for 5-star ratings (`1stars`, `2stars`, etc.).
- `power`: the power of a vote, in other words how many points it should increment the document's score by. 

## Defining Vote Types

You can define new vote types using the `addVoteType` function:

```js
import { addVoteType } from 'meteor/vulcan:voting';

addVoteType('upvote', {power: 1, exclusive: true});
addVoteType('downvote', {power: -1, exclusive: true});
```

Setting `exclusive` to `true` means that adding a vote of this type will clear our any votes of any *other* types (in other words, you can't have more than one vote at any given time).

Or: 

```js
addVoteType('1stars', {power: 1, exclusive: true});
addVoteType('2stars', {power: 2, exclusive: true});
addVoteType('3stars', {power: 3, exclusive: true});
addVoteType('4stars', {power: 4, exclusive: true});
addVoteType('5stars', {power: 5, exclusive: true});
```

Optionally, you can also define `power` as a function that takes the current user and document being upvoted as argument:

```js
import Users from 'meteor/vulcan:users';

addVoteType('downvote', {power: (currentUser, document) => Users.isAdmin(currentUser) ? 5 : 1});
```

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

- `currentUserVotes`
- `score`
- `baseScore`

Do not forget to add these fields to your fragments for any collection you're making voteable:

```js
registerFragment(`
  fragment PostsList on Post {

    # posts
    _id
    title
    url
    slug
    postedAt

    # users
    userId
    user {
      ...UsersMinimumInfo
    }

    # voting
    currentUserVotes{
      ...VoteFragment
    }
    baseScore
    score
  }
`);
```

## The Vote Mutation

Every collection added to the `VoteableCollections` array gets added to a `Voteable` union type, which in turn gets returned by the `vote` mutation:

```
vote(documentId: String, voteType: String, collectionName: String, voteId: String) : Voteable
```

Calling the mutation a second time with the same vote type will simply cancel the vote (for example, calling `vote` with vote type `upvote` a second time cancels out the first upvote and removes its power from the document's score). 

Note that the mutation supports passing a `voteId` to the server. This is in order to pre-generate an `_id` on the client to make sure that the vote object generated as part of the optimistic response and the “real” vote object returned by the server have the same `_id` and can be matched. 

## The withVote Container

The Voting package exports the `withVote` container that gives access to the `vote` mutation. You can then call the mutation like this (note that this example only implements upvoting, not downvoting):

```js
import { Components, registerComponent, withMessages, withCurrentUser } from 'meteor/vulcan:core';
import React, { PureComponent } from 'react';
import { withVote } from 'meteor/vulcan:voting';

class Vote extends PureComponent {

  constructor() {
    super();
    this.upvote = this.upvote.bind(this);
  }

  upvote(e) {
    e.preventDefault();

    const document = this.props.document;
    const collection = this.props.collection;
    const user = this.props.currentUser;

    if(!user){
      this.props.flash('Please log in first!');
    } else {
      this.props.vote({document, voteType: 'upvote', collection, currentUser: this.props.currentUser});
    } 
  }

  render() {
    return (
      <div>
        <a className="upvote-button" onClick={this.upvote}>
          <div className="vote-count">{this.props.document.baseScore || 0}</div>
        </a>
      </div>
    )
  }

}

registerComponent('Vote', Vote, withMessages, withVote, withCurrentUser);
```

## Callbacks

You can make use of the voting package through callbacks as well. For example, here's how to make every new post be upvoted by its user on creation: 

```
import Posts from 'meteor/example-forum';
import Users from 'meteor/vulcan:users';
import { addCallback } from 'meteor/vulcan:core';
import { performVoteServer } from 'meteor/vulcan:voting';

function PostsNewUpvoteOwnPost(post) {
  const postAuthor = Users.findOne(post.userId);
  return performVoteServer({documentId: post._id, voteType: 'upvote', collection: Posts, user: postAuthor});
}

addCallback('posts.new.sync', PostsNewUpvoteOwnPost);
```
