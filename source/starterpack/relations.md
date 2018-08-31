---
title: Database Relations
description: 'How to define relationships between collections'
disqusPage: 'Starterpack:Database-Relations'
---

Relations are very important when you deal with your database. MongoDB has many advantages but it does not have full support
for relational data, however there is an extremely performant way of handling those relations leveraging the power of [Grapher](https://github.com/cult-of-coders/grapher)

We recommend you read the package documentation to fully understand what is going on and skip the following chapters:

- Named Queries
- Global Exposure

Grapher is meant to work with pure Meteor even without Apollo, thus having some additional functionalities we will not use in here, but the rest of the functionalities are a must to understand, especially [GraphQL Bridge](https://github.com/cult-of-coders/grapher/blob/master/docs/graphql.md)

We have 2 ways of defining relationships, first we'll explore the prototipish way of doing it via Schema Directives:

`@mongo` - Creates or re-uses an already existing `Mongo.Collection`
`@link` - Defines the links with other types
`@map` - Maps a value to the database value so Grapher can interogate properly

Let's say we have a Post and a Comment and each post and comment has an `userId` that refers to a certain User:

```gql
type Comment @mongo(name: "comments") {
  _id: ID!
  text: String!

  userId: String!
  user: User @link(field: "userId")

  postId: String!
  post: Post @link(field: "commentId")
}

type Post @mongo(name: "posts") {
  _id: ID!
  title: String!
  isPublished: Boolean!
  createdAt: Date

  authorId: String
  author: User @link(field: "authorId")

  comments: [Comment] @link(to="post")
}

type User @mongo(name: "users") {
  _id: ID!
  name: String
  comments: [Comment] @link(to: "user")
  posts: [Post] @link(to: "author")
}
```

Given this configuration, you can then define the following GraphQL Module:

```js
export default {
  typeDefs: `
    type Query {
      posts(filters: JSON, options: JSON): [Post]
    }
  `,
  resolvers: {
    Query: {
      posts(_, { filters, options }, { db }, ast) {
        const query = db.posts.astToQuery(ast, {
          $filters: filters
          $options: options
        })

        return query.fetch()
      }
    }
  }
}
```

Now if you go to your GraphQL playground you can do something like:

```
query {
  posts {
    title
    author {
      name
    }
    comment {
      text
      user {
        name
      }
    }
  }
}
```

And relations not only are done automatically, [they are extremely performant](https://github.com/cult-of-coders/grapher/blob/master/docs/hypernova.md), and it will only interogate the database for the fields you need, not all fields.

And for example you can even add extra logic via parameters, for example, you only want the published posts sorted by createdAt descending, and only want the first page, paginated by 20, you would use the following parameters:

```
{
  "filters": {
    "isPublished": true
  },
  "options": {
    "sort": {
      "createdAt": -1
    },
    "limit": 20,
    "skip": 0,
  }
}
```

With few lines of code you have an imense amount of power. However, as your app grows, and you need database consistency and maybe other extensions. It's a good idea to move the database definitions and links outside the types (No more schema directives)

To do so you define a collection as it was explained in structure and you create the equivallent of the above like this:

```js
// src/db/posts/links.js

import { Posts, Comments, Users } from '../db';

Posts.addLinks({
  author: {
    type: 'one',
    field: 'authorId',
    collection: Users,
    index: true,
  },
  comments: {
    collection: Comments,
    inversedBy: 'post',
  },
});
```

```js
// src/db/comments/links.js

import { Posts, Comments, Users } from '../db';

Comments.addLinks({
  user: {
    type: 'one',
    field: 'userId',
    collection: Users,
    index: true,
  },
  post: {
    type: 'one',
    index: true,
    field: 'postId',
    collection: Posts,
  },
});
```

```js
// src/db/users/links.js
import { Posts, Comments, Users } from '../db';

Users.addLinks({
  comments: {
    collection: Comments,
    inversedBy: 'user',
  },
  posts: {
    collection: Posts,
    inversedBy: 'author',
  },
});
```

```js
// src/db/links.js
import './posts/links';
import './users/links';
import './comments/links';
```

And don't forget to import `src/db/links.js` in your `src/startup/server/index.js`.

Read more about Grapher & GraphQL here:
https://github.com/cult-of-coders/grapher/blob/master/docs/graphql.md
