---
title: Mutations
description: 'Lets create a form and send it to the server!'
disqusPage: 'Recipes:Mutations'
---

# Mutations

In this chapter, we're gonna create a Form from scratch, and send the data to the server. We are not going to use the database, this is for showing you the main concepts of how to perform mutations.

There is really no main difference between `Query` or `Mutation` in the way you use GraphQL API. They are literally the same. The distinction has been introduced to separate operations that modify the state on the servers vs operations that just fetch data without affecting anything.

Let's define our module:

```js
// src/api/modules/item/index.js

export default {
  typeDefs: `
    type Item {
      _id: Int!
      name: String!
      isAvailable: Boolean
    }

    input ItemCreateInput {
      name: String!
      isAvailable: Boolean!
    }

    type Mutation {
      itemsInsert(item: ItemCreateInput!): Item
    }
  `,
  resolvers: {
    Mutation: {
      itemsInsert(_, args) {
        console.log(`I received`, args);
        const { item } = args;

        // Let's mock a database insertion
        // We're not really performing anything on the database

        item._id = 10;
        return item;
      },
    },
  },
};
```

By now you should know what an `input` is and how to create a form.

Let's create the form:

```js
import React from 'react';
import { Mutation } from 'react-apollo';
import gql from 'graphql-tag';
import { AutoForm } from 'uniforms-antd';
import SimpleSchema from 'simpl-schema';

const ItemCreateSchema = new SimpleSchema({
  name: {
    type: String,
  },
  isAvailable: {
    type: Boolean,
    optional: true,
  },
});

const ITEMS_INSERT = gql`
  mutation itemsInsert($item: ItemCreateInput!) {
    itemsInsert(item: $item) {
      _id
    }
  }
`;

// AutoForm auto-generates your form based on the schema you provided!
const ItemForm = ({ onSubmit }) => (
  <AutoForm schema={ItemCreateSchema} onSubmit={onSubmit} id="item-create" />
);

export default () => (
  <Mutation mutation={ITEMS_INSERT}>
    {mutate => {
      return (
        <ItemForm
          onSubmit={item =>
            mutate({ variables: { item } }).then(response =>
              console.log(`I received`, response)
            )
          }
        />
      );
    }}
  </Mutation>
);
```

Now after you added the route, you will be able to just click **Submit** and see first the logs on the server, and then the response on the client.

Feel free to read more about `<Mutation>` component here:
https://www.apollographql.com/docs/react/essentials/mutations.html
