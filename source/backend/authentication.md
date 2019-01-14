---
title: Authentication
description: 'Learn how to easily work with users & GraphQL'
disqusPage: 'Backend:Authentication'
---

# Authentication

In this chapter we'll learn registration process and login & authentication.

Meteor comes with a nice and simple authentication system. And we're gonna learn how to leverage it inside Apollo.

# Setting Up

Adding accounts to your GraphQL Schema:

```
meteor add accounts-password
meteor npm i -S bcrypt meteor-apollo-accounts
meteor add cultofcoders:apollo-accounts
```

Create a type called `User`. This is how Meteor's default user schema looks like:

```js
type User {
  _id: ID!
  username: String
  email: String
  profile: UserProfile
  emails: [UserEmail]
  roles: [String]
}

type UserProfile {
  firstName: String
  lastName: String
}

type UserEmail {
  address: String
  verified: Boolean
}
```

```js
// file: /server/accounts.js
import { initAccounts } from 'meteor/cultofcoders:apollo-accounts';
import { load } from 'graphql-load';

const AccountsModule = initAccounts({
  loginWithFacebook: false,
  loginWithGoogle: false,
  loginWithLinkedIn: false,
  loginWithPassword: true,
});

load(AccountsModule);
// Make sure initialize() runs after everything is loaded()
```

If you want to learn more about `initAccounts`, check the docs here:
https://github.com/cult-of-coders/meteor-apollo-accounts

# Usage

You can register an account like this:

```js
mutation {
  createUser(
    username: "cultofcoders",
    plainPassword: "12345",
  ) {
    token
  }
}
```

Now copy/paste that token, and inside `HTTP Headers` use:
`meteor-login-token: ${token}`, and that's how your resolvers know who is requesting data.

Your resolver function receives `root`, `args` and `context`. Inside `context` we store the current `userId` and `user`.

An example to illustrate the usage:

```js
// file: server/
export default {
  Query: {
    invoices(root, args, context) {
      const { userId, user } = context;

      return Invoices.find({
        userId,
      }).fetch();
    },
  },
};
```

You notice that we also inject the `user` object. We can customise which fields to get for the user:

```js
import { initialize } from 'meteor/cultofcoders:apollo';

initialize({}, {
  // You can configure your default fields to fetch on each GraphQL request
  userFields: {
    _id: 1,
    roles: 1,
  }
}),
```

Test the fact that you can query the logged in user:

```js
import { Meteor } from 'meteor/meteor';
import { load } from 'meteor/cultofcoders:apollo';

const typeDefs = `
  type Query {
    me: User
  }
`;

const resolvers = {
  Query: {
    me(_, args, context) {
      return Meteor.users.findOne(context.userId);
    },
  },
};

load({
  typeDefs,
  resolvers,
});
```

And try it out:

```js
query {
  me {
    _id
  }
}
```

---

### [Table of Contents](index.md)
