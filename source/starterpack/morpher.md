---
title: Morpher
description: 'Prototype fast and make the client-server abstraction vanish'
disqusPage: 'Starterpack:Morpher'
---

# Morpher

Gives you a way to quickly prototype and make the client-server abstraction disappear. To allow this easy prototyping it communicates with the server through EJSON sending the payload data as a `String` which is deserialised on the server.

This may be a bit confusing, when you want to use morpher without the npm library `apollo-morpher`, but there shouldn't be scenarios for that.

## Install

```
meteor npm i -S apollo-morpher
```

### Server Usage

```js
// src/api/modules/user/expose.js
import { expose, db } from 'meteor/cultofcoders:apollo';

expose({
  users: {
    type: 'User'
    collection: () => db.users,
    // In the mutation methods you can perform propper checks, you get access to the context
    // You have ability to extract user from ctx `ctx.userId` or `ctx.user`
    update: (ctx, {selector, modifier, modifiedFields, modifiedTopLevelFields}) => true,
    insert: (ctx, {document}) => true,
    remove: (ctx, {selector}) => true,
    find(ctx, params) {
      // params is an object
      // by default filters, options are always empty objects, if they were not passed
      // if you pass other params filters and options will still be empty objects

      // You have two options here:
      // 1. Modify params.filters and params.options and don't return anything
      params.filters.userId = ctx.userId

      // 2. Modify filters options based on other parameters sent out
      if (params.accepted) {
        params.filters.accepted = true;
      }

      // 3. Return astToQueryOptions from Grapher for custom query support
      // https://github.com/cult-of-coders/grapher/blob/master/docs/graphql.md
      return {
        $filters: params.filters,
        $options: params.options
      }
    }
  }
})
```

Create a `exposures.js` inside `src/startup/server` in which you import all exposures before running `initialize()` on the GraphQL server.

### Client Usage

```js
// Then on the client
import db, { setClient } from 'apollo-morpher';

// Set your instantiated Apollo Client
setClient(apolloClient);

// Built-in mutations
db.users.insert(document).then(({ _id }) => {});
db.users.update(selector, modifier).then(response => {});
db.users.remove(selector).then(response => {});

// Or define the in object style:
const fields = {
  firstName: 1,
  lastName: 1,
  // You can also query the defined links!
  lastInvoices: {
    total: 1,
  },
};

// Or you could also define the fields in GraphQL style `firstName`

db.users
  .find(fields, {
    filters: {},
    options: {},
  })
  .then(users => {});

// find equivallent .findOne()
db.users
  .findOne(fields, {
    filters: { _id: 'XXX' },
  })
  .then(user => {});

// and for pagination purposes to retrieve the count
db.users
  .count({
    filters,
    options,
  })
  .then(count => {});
```
