---
title: Setting Up
description: 'Run a GraphQL end-point in minutes'
disqusPage: 'Starterpack:Setting-Up'
---

Make sure you have Meteor installed: https://www.meteor.com/install

# Create a fresh project

If you are in a terminal you can just copy/paste all at once. We start with an empty application.

```
meteor create --bare my-app
cd my-app

# Dependencies for the server
meteor npm i -S graphql graphql-load apollo-server-express uuid graphql-tools graphql-type-json apollo-live-server

# Dependencies for the client
meteor npm i -S react-apollo apollo-live-client apollo-client apollo-cache-inmemory apollo-link apollo-link-http apollo-link-ws apollo-morpher subscriptions-transport-ws apollo-upload-client

# If you're looking into Server Side Rendering with React
meteor npm i -S react-dom react-router apollo-link-schema

# Now we add the package
meteor add cultofcoders:apollo

# Optional but highly recommended (so you can import .gql/.graphql files)
meteor add swydo:graphql
```

Now let's initialize our GraphQL server:

Add the following to `package.json`, it will allow us to specify the entry points for our app:

```
  "meteor": {
    "mainModule": {
      "client": "src/startup/client/index.js",
      "server": "src/startup/server/index.js"
    },
    "testModule": "tests/main.js"
  }
```

```
mkdir -p src/startup/client src/startup/server src/api
touch src/startup/client/index.js src/startup/server/index.js src/api/index.js
```

Now go to `src/startup/server/index.js` and add this:

```js
// src/startup/server/apollo.js
import { initialize } from 'meteor/cultofcoders:apollo';

const { server } = initialize();

export { server };
```

```js
// src/api/index.js
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
```

```js
import '../../api';
import './apollo';
```

Note: the initialize() function needs to run after everything has been loaded, loading anything after `initialize()` will have no effect on your Schema.

Now you can start Meteor via:

```
meteor npm start
```

Now get on your browser and go to: http://localhost:3000/graphql and give your first query a spin:

```gql
query {
  sayHello
}
```

## Module Aliases

In addition to what we just described, you can also create module aliases so:

`npm i --save-dev babel-plugin-module-resolver`

In `.babelrc`

```
{
  "presets": ["meteor"],
  "plugins": [
    [
      "module-resolver",
      {
        "root": ["."],
        "alias": {
          "api": "./src/api",
          "ui": "./src/ui",
          "db": "./src/db"
        }
      }
    ]
  ]
}
```

This allows you to do nice things such as `import Page from 'ui/pages/Page';`, for example in the `startup/server` you could have done `import 'api'` instead of `import '../../api` or worse `import '/src/api`

Congratulations, you have succesfully setup a GraphQL Server that is extremely powerful and feature-rich.
