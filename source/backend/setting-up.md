---
title: Setting Up
description: 'Setup your GraphQL Server easily'
disqusPage: 'Backend:Setting-Up'
---

# Setting Up

We are going to work with Meteor for the following reasons:

- It is an extremely efficient and easy to use build-tool
- It is easy for us to opt-out of it in the future (we don't depend on it)

You can install [Meteor here](https://www.meteor.com/install).

Create your project:

```
meteor create --bare graphql-baby
cd graphql-baby
```

Now we install our npm dependencies for server:

```
meteor npm i -S graphql graphql-load apollo-server-express uuid graphql-tools graphql-type-json apollo-live-server
```

Dependencies for the client:

```
meteor npm i -S react-apollo apollo-live-client apollo-client apollo-cache-inmemory apollo-link apollo-link-http apollo-link-ws apollo-morpher subscriptions-transport-ws apollo-upload-client
```

Now we add [this package](https://github.com/cult-of-coders/apollo) which elegantly blends Apollo inside Meteor:

```
meteor add cultofcoders:apollo
```

Now open the project in your favorite IDE, and inside server/main.js write out:

```js
// file: server/main.js
import { initialize } from 'meteor/cultofcoders:apollo';
import { load } from 'graphql-load';

load({
  typeDefs: `
    type Query {
      sayHello: String
    }
  `,
  resolvers: {
    Query: {
      sayHello: () => 'Hello world!',
    },
  },
});

initialize();
```

Now you have a fully working extremely potent GraphQL server:

```
meteor run
```

Enjoy the start of a beautiful journey and explore your playground, we'll be playing a lot in it.

http://localhost:3000/graphql

Give this query a spin:

```
query {
  sayHello
}
```

Woop Woop! That's it. You are now ready young skywalker. We'll be exploring lots of great stuff together, we will show you how awesome Web Development has become. The sheer elegance. The beauty. Oh!
