---
title: Queries
description: 'Make your first query run!'
disqusPage: 'Recipes:Queries'
---

# Queries

Let's create a query server-side and then interogate it on our client.
For the sake of simplicity we are not going to store our main types in `entities` folder, this is just for illustration.

First, let's define some sample data:

```js
// src/api/modules/item/items.js
export default [
  {
    _id: 1,
    name: 'Big',
    isAvailable: true,
  },
  {
    _id: 2,
    name: 'Bang',
    isAvailable: false,
  },
  {
    _id: 3,
    name: 'BigBang',
    isAvailable: true,
  },
];
```

Now let's define our Item type and the Query:

```js
// src/api/modules/item/index.js
import items from './items';

export default {
  typeDefs: `
    type Item {
      _id: Int!
      name: String!
      isAvailable: Boolean
    }

    type Query {
      items: [Item]!
    }
  `,
  resolvers: {
    Query: {
      items() {
        return items;
      },
    },
  },
};
```

And import this file inside your API entry-point in `src/api/index.js`:

```js
import { load } from 'graphql-load';
import ItemModule from './modules/item';

load([ItemModule]);
```

Now if you go to http://localhost:3000/graphql, you should be able to run this query:

```gql
query {
  items {
    _id
    name
  }
}
```

## React Integration

```jsx
// src/ui/pages/Items/index.js
import React from 'react';
import { Query } from 'react-apollo';
import gql from 'graphql-tag';

const ITEMS_LIST = gql`
  query {
    items {
      _id
      name
    }
  }
`;

const Items = ({ items }) => (
  <ul>
    {items.map(item => (
      <li key={item._id}>
        {item._id} :: {item.name}
      </li>
    ))}
  </ul>
);

export default () => (
  <Query query={ITEMS_LIST}>
    {({ data, loading }) => {
      if (loading) {
        return 'Please wait...';
      }

      return <Items items={data.items} />;
    }}
  </Query>
);
```

Don't forget to add the route for this component. Now feel free to test it!

## Arguments

Let's say we would like to receive only the available items and we want to specify this as an argument:

Modify your type definition:

```gql
type Query {
  items(showAvailableOnly: Boolean): [Item]!
}
```

And inside your resolver:

```js
{
  Query: {
    items(_, args) {
      const { showAvailableOnly}  = args;
      if (showAvailableOnly) {
        return items.filter(item => item.isAvailable === true);
      }

      return items;
    }
  }
}
```

Now if you go to http://localhost:3000/graphql you can run this:

```
query {
  items(showAvailableOnly:true) {
    _id
    name
    isAvailable
  }
}
```

However, if we want to manipulate these values inside the React component, so it can specify what to use, we are going to use the following approach:

```js
query items($showAvailableOnly:Boolean) {
  items(showAvailableOnly:$showAvailableOnly) {
    _id
    name
    isAvailable
  }
}
```

This may seem a bit confusing at first, you will first notice that after query comes `items` that's just a name we gave to the query that helps Apollo to do a lot of nice things, but that name could have been anything.

Next to it we define the variables, it's like saying, these are the variables that I'm allowing in this query, and this is their type.

And next, when we actually make the query, we need to provide the variables that our query really accepts. For example, this would also work:

```js
query myItems($available:Boolean) {
  items(showAvailableOnly:$available) {
    _id
    name
    isAvailable
  }
}
```

Now, let's use the last one we showed (with `myItems`) and replace it inside your React component, then you will be able to specify variables, by making your `Query` like this:

```js
<Query query={ITEMS_LIST} variables={{ available: true }}>
  {...}
</Query>
```

## Inputs

But what happens when we want to send more parameters ? For example we have a super complex list, that takes in a lot of arguments ?

The recommended solution is to create an `input` type like so:

```
input ItemListFilters {
  isAvailable: Boolean
  startsWith: String
  numberOfWords: Int!
}

type Query {
  items(filters: ItemListFilters!): [Item]
}
```

Note that the `!` means you **must** provide an input.

So your query can look like this:

```js
query myItems($filters:ItemListFilters) {
  items(filters:$filters) {
    _id
    name
    isAvailable
  }
}
```

And ofcourse, your `Query` will look like this:

```js
const filters = {
  isAvailable: false,
  startsWith: 'Big',
  numberOfWords: 2
}

return (
  <Query query={ITEMS_LIST} variables={{ filters }}>
    {...}
  </Query>
)
```

This won't really work because you did not add any logic in your resolvers to handle this, but this is to show how to think about multiple arguments.

Feel free to read more about `<Query>` component here:
https://www.apollographql.com/docs/react/essentials/queries.html
