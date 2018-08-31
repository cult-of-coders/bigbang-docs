---
title: 'Testing'
description: Testing server-side logic
disqusPage: 'Starterpack:Testing'
---

# Testing

Testing your app is not something `extra`, it's not something that should be done, it's something that must be done.

In the prototyping phase you may be inclined to skip this part, and that may be ok, but as a principles, writing tests can help you code faster, therefore prototype faster, and if you use the principles stated before separating logic in services, it can make testing a breeze.

Make sure your package.json looks like this:

```
  "meteor": {
    "mainModule": {
      "client": "src/startup/client/index.js",
      "server": "src/startup/server/index.js"
    },
    "testModule": {
      "client": "src/__tests__/client.js",
      "server": "src/__tests__/server.js"
    }
  },
```

Inside client.js and respectively server.js you are going to import the tests. You will not write any tests in there.

There is another alternative, to completely remove `testModule` and all files `*.test.js` are going to be eagerly loaded. However we strongly recommend the `testModule` approach:

Add the following to your `package.json` file:

```
"scripts": {
  "test": "meteor test --driver-package decaffed:mocha --port 3050",
  "test-console": "TEST_WATCH=1 meteor test --driver-package meteortesting:mocha --port 3050",
  "test-chrome": "TEST_WATCH=1 TEST_BROWSER_DRIVER=chrome meteor test --driver-package meteortesting:mocha --port 3050",
  "test-chrome-watch": "TEST_WATCH=1 TEST_BROWSER_DRIVER=chrome meteor test --once --driver-package meteortesting:mocha --port 3050"
}
```

Note that if you want to run tests with `chromedriver` (meaning you want the browser test results in your console) you'll have to install the npm package. However this will come with a performance expense to your rebuilds, and it's not recommended for development mode. However it's a must for your CI, this is why when you're running your test for CI before run a `meteor npm i -S chromedriver`

Let's write a service and a test:

```
meteor npm i --save-dev chai
meteor npm test
```

```js
// src/api/modules/item/services/ItemService.js
export default class ItemService {
  constructor({ db }) {
    this.db = db;
  }

  insertItem(name) {
    return this.db.items.insert({
      name,
    });
  }
}
```

```js
// src/api/modules/item/services/ItemService.test.js
import ItemServiceModel from './ItemServiceModel';
import { assert } from 'chai';

describe('ItemServiceModel', function() {
  it('should be able to create an item', function() {
    const service = new ItemServiceModel({
      db: {
        items: {
          insert: () => 'ok',
        },
      },
    });

    const response = service.insertItem('Johnas');

    assert.equal(response, 'ok');
  });
});
```

```js
// src/__tests__/server.js
import '../api/modules/item/services/ItemService.test';
```

Now open your browser http://localhost:3050 and you'll have a nice interface to see your tests, and you can click on any individual test or group of tests to run them in isolation. (This is good when you have a large-suite and just one failing test)

It's strongly recommended that you give this a read: https://guide.meteor.com/testing.html, it will give you a better understanding how to test the full app or just some unit-tests, and also hints to some great utility belts:

- Resetting the database: https://atmospherejs.com/xolvio/cleaner
- Mocking the database: https://atmospherejs.com/hwillson/stub-collections
- Generate fake data: https://atmospherejs.com/dburles/factory

## Testing the GraphQL server

What if we want to test the GraphQL server, but on the server-side ?

```js
import { intialize } from 'meteor/cultofcoders:apollo';
import { SchemaLink } from 'apollo-link-schema';

const { server } = initialize({});

function getClient(context = {}) {
  const link = new SchemaLink({
    schema: server.schema,
    context,
  });

  const client = new ApolloClient({
    link,
    cache: new InMemoryCache(),
  });
}
```

Now feel free to test your schema using as `client` the client from `getClient()` function.
