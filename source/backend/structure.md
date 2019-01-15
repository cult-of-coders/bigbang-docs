---
title: Structure
description: 'Setup your GraphQL Server easily'
disqusPage: 'Backend:Setting-Up'
---

# Thinking Modular

Clearly we won't be storing all of our code inside `server/file` and our code is going to evolve. We may need to store our type definitions into a nicely formatted `.gql` / `.graphql` file. 

So let's start thinking modular.

```sh
.
└── src
    └── modules
        ├── index.js # Imports all the modules (import ./{moduleName})
        └── {moduleName}
            ├── __tests__ # Your unit/module integration tests
            ├── db # Store our collections
            ├── entities # Store the principal types, like "User", "Post", etc
            ├── index.js # Here is the main module aggregator that loads different things
            ├── main.js # Here lies the main module that contains Query types and resolvers
            └── services # Here you store the logic into propper classes

```

So basically our `mainModule` inside `package.json` should just point to `modules/index.js`. However, we recommend a slightly different approach. Make your main module point to `src/startup/server/index.js` and from then on import what you wish. Because we want to keep our modules clean, without configurations related to server setups.

# Working with `.gql` files

```
meteor add swydo:graphql
```

In theory you want a GraphQLModule which is composed of `typeDefs` and `resolvers`. To create such a module here is a strategy. Let's say you want to create some queries and mutations related to posts.

```js
# file: posts/types.gql
type Query {
    posts(limit: Int!): [Post]
}

type Mutation {
    addPost(post: PostInput!): Boolean
}

input PostInput {
    title: String!
}
```

```js
// file: posts/resolvers.js
export default {
    Query: {
        posts: (_, { limit }) => ...
    },
    Mutation: {
        addPost(_, { post }) {
            // ...
        }
    }
}
```

```js
// file: posts/index.js
// just make sure you import this
import { load } from 'graphql-load';
import typeDefs from './types';
import resolvers from './resolvers';

load({
    typeDefs,
    resolvers
})
```

This is just an example of how you can separate your concerns into GraphQL Modules and benefit of `.gql` types.

And we would prefer to use the same approach for entities:

```bash
.
├── index.js # Loads everything
├── Post
│   ├── index.js # Similar approach to what we showed above
│   ├── resolvers.js
│   └── types.gql
└── User
    ├── index.js
    ├── resolvers.js
    └── types.gql
```

For collections (db):

```bash
.
├── index.js
└── posts
    ├── collection.js
    └── schema.js
```


