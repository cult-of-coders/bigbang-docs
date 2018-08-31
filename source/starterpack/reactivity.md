---
title: Live Queries
description: How to have reactive/live queries with ease
disqusPage: 'Starterpack:Reactivity'
---

```js
meteor npm i -S apollo-live-client apollo-live-server
```

This package comes already with the [apollo-live-server](https://www.npmjs.com/package/apollo-live-server) npm package already set-up, but for client you will also need: [apollo-live-client](https://www.npmjs.com/package/apollo-live-client), so make sure it is installed.

```js
```

```js
// our Subscription resolver
import { asyncIterator } from 'apollo-live-server';

export default {
  typeDefs: `
    type Subscription {
      users: SubscriptionEvent
    }
  `
  resolvers: {
    Subscription: {
      users: {
        resolve: payload => payload,
        subscribe(_, args, { db }) {
          const observer = db.users.find();

          return asyncIterator(observer);
        }
      }
    }
  }
}
```

Find out more: https://www.npmjs.com/package/apollo-live-server

## Client usage with React

Please refer to the documentation here: https://github.com/cult-of-coders/apollo-live-client

## Simulate reactivity

```js
// Meteor.users acts as any other Mongo.Collection you may have
// Simulate some reactivity ...
import { Accounts } from 'meteor/accounts-base';

Meteor.setInterval(function() {
  const userId = Accounts.createUser({
    username: 'Apollo is Live!',
  });

  Meteor.setTimeout(function() {
    Meteor.users.remove({ _id: userId });
  }, 500);
}, 2000);
```

You can now test your query inside GraphQL Playground, to see how easily it reacts to changes:

```js
subscription {
  users {
    event
    doc
  }
}
```

---

### [Table of Contents](index.md)
