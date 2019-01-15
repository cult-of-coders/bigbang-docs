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

For MacOS/Linux users:
```
curl https://install.meteor.com/ | sh
```

Create your project:

```
meteor create --bare graphql-baby
cd graphql-baby
```



Now we add [this package](https://github.com/cult-of-coders/apollo) which elegantly blends Apollo inside Meteor:

```
meteor add cultofcoders:apollo
```

We continue as we are instructed on the main page of the package where it tells us what npm dependencies to install

Now we install our npm dependencies for running the server & the client.

```
meteor npm i -S graphql graphql-load apollo-server-express uuid graphql-tools graphql-type-json apollo-live-server
meteor npm i -S react-apollo apollo-live-client apollo-client apollo-cache-inmemory apollo-link apollo-link-http apollo-link-ws apollo-morpher subscriptions-transport-ws apollo-upload-client
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

# Meteor's Eager Loading

All the files inside server/ are loaded on the server-side eagerly.

However, this is bad because ultimately you'll lose control and regret it. It is a nice feature to quickly start, but we won't use it. So let's setup an entry point. Inside `package.json` add this config:

```json
{
  "meteor": {
    "mainModule": {
      "server": "server/main.js"
    }
  }
}
```

Now from this `main.js` everything is imported and no files gets automatically loaded without our specific approval.

If you want to load different things you could do it easily like this:


```
// file: server/sayDifferentThings.js
import { load } from 'graphql-load';

load({
  typeDefs: `
    type Query {
      saySomethingElse: String!
    }
  `,
  resolvers: {
    Query: {
      saySomethingElse: () => 'Hello, you!'
    }
  }
})
```

Now, inside your `main.js` put the import on top:
```js
import './sayDifferentThings';
```

Now it's time to learn a bit about Types, Queries and Mutations. Then we'll learn how to work with the database and user authentication. Then we dive into how to properly organise our code. I'm excited to show you the sheer elegance and beauty of all this. Bare with me.

