---
title: Setting Up
description: 'Setting Up'
disqusPage: 'Starterpack:Setting Up'
---

# Setting Up

So, let's get started with React, let's connect to Apollo, add some routing, and see our first query!

First things first, let's setup our structure. As you know, our client code will exist in `src/ui`

```
meteor npm i -S react react-dom react-router react-router-dom
```

```js
// src/ui/main/index.js
import React, { Component, Fragment } from 'react';
import { Route } from 'react-router';
import routes from './routes';

export default class App extends Component {
  render() {
    return (
      <Fragment>
        {routes.map((route, idx) => (
          <Route key={idx} exact={true} {...route} />
        ))}
      </Fragment>
    );
  }
}
```

The routes will look like this:

```js
// src/ui/main/routes.js
import Home from '../pages/Home';

// The properties of this routes can be found here:
// https://reacttraining.com/react-router/core/api/Route/route-props
export default [
  {
    path: '/',
    component: Home,
  },
];
```

Let's create a dummy Home page:

```js
// src/ui/pages/Home/index.js
export default () => {
  <h1>Home Page</h1>;
};
```

Now we need to initialize all of this. And to do so, we go in our `src/ui/startup/client` folder.

First let's boot up our Apollo Client:

```js
// src/startup/client/apollo.js
import { initialize } from 'meteor/cultofcoders:apollo';

const { client } = initialize();

export default client;
```

Now let's start rendering!

```js
// src/ui/startup/client/index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { ApolloProvider } from 'react-apollo';
import { BrowserRouter } from 'react-router-dom';

import App from '../../ui/main';
import apolloClient from './apollo';

const ApolloApp = () => (
  <ApolloProvider client={apolloClient}>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </ApolloProvider>
);

// Note: If you don't want to use SSR, replace `ReactDOM.hydrate` with `ReactDOM.render`.
ReactDOM.hydrate(<ApolloApp />, document.getElementById('react-root'));
```

## Server Side Rendering

Perfect, now the next step is to enable Server Side Rendering (SSR). Which allows the server to render the application extremely fast.

```
meteor add server-render
```

```js
// src/startup/server/ssr.js

// And don't forget to import this file from your index.js file
import React from 'react';
import { StaticRouter } from 'react-router';

import { getRenderer } from 'meteor/cultofcoders:apollo';
import { onPageLoad } from 'meteor/server-render';

import { server } from './apollo';
import App from '../../ui/main';

const render = getRenderer({
  app: sink => (
    <StaticRouter location={sink.request.url} context={{}}>
      <App />
    </StaticRouter>
  ),
  server,
});

// hanlde SSR
onPageLoad(render);
```

You're set in place to start rocking your app!
