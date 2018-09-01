---
title: Folder Structure
description: 'Learn how to organize your GraphQL code'
disqusPage: 'Starterpack:Structure'
---

In this chapter we'll explore an opinionated way of organising our code. The milky way.

All of our source code lies in `./src` folder.

We have two folders `startup/client` and `startup/server` those are the places where we perform system level configurations for their respective environment, where you initialize maybe different packages and so on.

So, in there lies configuration, not implementation. For example, it would make sense in there, to initialize the GraphQL server with a certain config, but it would not make sense to add any Query or resolver logic.

We will introduce to following folders:

- src/db - Here is where we define database schema and logic
- src/api - Here is where we define our API implement any kind of logic and operations on the database
- src/ui - Here lie all of our UI components

## Entities

Our GraphQL schema will exist in: `src/api` and it is composed of `entities` and `modules`. An entity is a type, but since type is such a generic word in JS world, we are going to refer to them as entities.

```gql
# src/api/entities/User.gql
type User {
  _id: ID!
  firstName: String
  lastName: String
  fullName: String
}
```

```js
// src/api/entites/User.resolvers.js
export default {
  User: {
    fullName: user => `${user.firstName} ${user.lastName}`,
  },
};
```

And the place where we aggregate them all:

```js
// src/api/entities/index.js

import UserType from './User.gql';
import UserResolvers from './User.resolvers.js';
import PostType from './Post.gql';
import PostResolvers from './Post.resolvers.js';

export default {
  typeDefs: [UserType, PostType],
  UserResolvers: [UserResolvers, PostResolvers],
};
```

## GraphQL Modules

We regard as a GraphQL module an object that contains typeDefs and/or resolvers. Using the `graphql-load` package this allows us to work with these modules and allows us to think about them logically separated.

In our case inside `src/api/entities/index.js` what we export is a GraphQL Module.

So to aggregate and use that, we'll create our entry point for loading inside:

```js
// file: src/api/index.js
import { load } from 'graphql-load';

import EntitiesModule from './entities';
import UserModule from './modules/user';
import PostModule from './modules/post';

load([EntitiesModule, UserModule, PostModule]);
```

A sample implementation of the `post` module inside `src/api/modules/post`:

```
# src/api/modules/post/typeDefs.gql
type Query {
  posts: [Post]
}

type Mutation {
  postCreate(title: String!): Post
}
```

```js
// src/api/modules/post/resolvers.js
export default {
  Query: {
    posts: () => {
      return Posts.find().fetch();
    },
  },
  Mutation: {
    postCreate: (_, args, context) => {
      const title = args.title;
      // Logic to create the Post and you have to return the Post
    },
  },
};
```

```js
// ./index.js
import typeDefs from './typeDefs.gql';
import resolvers from './resolvers';

export default {
  typeDefs,
  resolvers,
};
```

Alternatively, for simplicity, you can also do something like this, without the need of `typeDefs.gql` or `resolvers.js`

```js
// src/api/modules/post/index.js
export default {
  typeDefs: `
    type Query {
      posts: [Post]
    }

    type Mutation {
      postCreate(title: String!): Post
    }
  `,
  resolvers: {
    Query: {
      posts: (_, args, context) => {
        return Posts.find().fetch();
      },
    },
    Mutation: {
      postCreate: (_, args, context) => {
        const title = args.title;
        // Logic to create the Post and you have to return the Post
      },
    },
  },
};
```

And ofcourse now what we have exported here is the `PostModule`.

## Services

You should not store logic inside your `resolvers`, even if that is the place you're gonna store it and it makes a lot of sense!

There is a lot to talk about here, so we're only going to scratch the surface, but keep in mind that you want your mutations and queries to be easily testable, and you may not want to rely on the full GraphQL API to test them.

Proposal, decouple logic into services like this:

```js
// src/api/modules/post/services/PostService.js

export default class PostService {
  static createPost(title) {
    // ... logic here ...
  }
}
```

```js
// src/api/modules/post/resolvers.js
import PostService from './services/PostService';

export default {
  Mutation: {
    createPost(_, args, context) {
      PostService.createPost(args.title);
    },
  },
};
```

The objective of mutation resolver is not just to delegate, it acts as `Controller` (contains the flow control logic) so it can also perform security checks (eg: if the user is logged in) or consistency checks (if the data passed is aligned to what the service needs) and handle exceptions correctly.

## Database

The database is not your API, you should treat them completely separated. However, in the beginning it's very likely that you would want to just prototype quickly without thinking too much about it.

This is why you can define the database collections inside the type:

```js
type Post @mongo(name: "posts") {
  _id: ID!
  createdAt: Date
  title: String
}
```

And you can access `posts` collection from `context.db.posts` inside your resolvers. This is how you should prototype, below is how you should grow:

We recommend the following structure inside `src/db`:

```js
// src/db/posts/collection.js
import { Mongo } from 'meteor/mongo';

const Posts = new Mongo.Collection('posts');

export default Posts;
```

The API for Mongo.Collection can be found here:
https://docs.meteor.com/api/collections.html

If you want to enforce a schema:
https://github.com/aldeed/meteor-collection2

```js
// src/db/posts/schema.js
import SimpleSchema from 'simpl-schema';

export default new SimpleSchema({
  title: {
    type: String,
  },
});
```

```js
// src/db/posts/collection.js
import PostSchema from './schema';

// ...
Posts.attachSchema(PostSchema);
```

```js
// src/db/index.js
import Posts from './posts/collection';

export { Posts };
```
