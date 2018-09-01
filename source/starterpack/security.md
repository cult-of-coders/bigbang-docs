---
title: Security
description: How to secure your Queries and Mutation resolvers
disqusPage: 'Starterpack:Security'
---

Security is a very important aspect of any application. When your code-base grows,
you need to more and more careful on how you handle this. We'll first show how we can
secure our `Query` and `Mutation` resolvers, then we will get into some tips and tricks meant to teach you how to handle
the security for an evergrowing code base.

## Securing Methods & Publicaions

Let's see a sample:

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
        if (!context.userId) {
          throw new Error(`Unauthorized`);
        }

        return Posts.find().fetch();
      },
    },
    Mutation: {
      postCreate: (_, args, context) => {
        if (!context.userId) {
          throw new Error(`Unauthorized`);
        }
        // Logic to create the Post and you have to return the Post
      },
    },
  },
};
```

## Managing Roles

Based on the userId you have the ability to check if the user is logged in, maybe you have multiple roles in the system,

We recommend installing the infamous package, [alanning:roles](https://atmospherejs.com/alanning/roles):

```
meteor add alanning:roles
```

## Security Service

Centralize security in a module:

```js
// file: /src/api/security/index.js
// example of a module for security
import { Roles } from 'meteor/alanning:roles';

export default class Security {
  static checkRole(userId, role) {
    if (!this.hasRole(userId, role)) {
      throw new Error('not-authorized');
    }
  }

  static hasRole(userId, role) {
    return Roles.userIsInRole(userId, role);
  }

  static checkLoggedIn(userId) {
    if (!userId) {
      throw new Error('not-authorized', 'You are not authorized');
    }
  }

  // add other business logic checks here that you use throughout the app
  // something like: isUserAllowedToSeeDocument()
  // always keep decoupling your code if this class gets huge.
}
```

```js
import Security from '/src/api/security';

export default {
  typeDefs: `...`
  resolvers: {
    Query: {
      posts: (_, args, { userId }) => {
        Security.checkLoggedIn(userId);

        return Posts.find().fetch();
      },
    },
    Mutation: {
      postCreate: (_, args, context) => {
        Security.checkLoggedIn(userId);
        // Logic to create the Post and you have to return the Post
      },
    },
  },
};
```

Pretty straight forward right? The reason we do it like this, by centralizing security in one place,
is to remove boilerplate code inside our methods and keep separation of concerns. You can do it however you want it, there is no right or wrong way to do it,
depends on your use-case, but we believe that it is easier to maintain, and newly onboarded developers were writing secure
code right from the start!

That's it. With this knowledge you can build more secured apps!
