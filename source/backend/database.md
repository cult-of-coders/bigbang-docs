---
title: Database
description: 'Learn how to easily play with MongoDB'
disqusPage: 'Backend:Database'
---

# Database

We're going to use Meteor's built-in MongoDB database to quickly play with stored data.

You can find the API here:
https://docs.meteor.com/api/collections.html#Mongo-Collection

But mostly you're gonna use it like this:

```js
MyCollection.find(mongoSelectors, {
  limit: 10, // useful for pagination, limits the results set
  skip: 0, // useful for pagination, skips the first results found
  fields: { // specify which fields to fetch
    field: 1,
    'nested.field': 1,
  }
  sort: {
    'createdAt': -1 // sorting by fields
  }
});

MyCollection.insert(document);
MyCollection.update(mongoSelectors, modifier, {multi: true});
MyCollection.remove(modifier);
```

```js
// file: server/collections.js
import { Mongo } from 'meteor/mongo';

export const Todos = new Mongo.Collection('todos');

// Add some demo data
Meteor.startup(() => {
  const count = Todos.find().count();
  if (count === 0) {
    for (let i = 0; i < 5; i++) {
      Todos.insert({
        title: `Todo - ${i}`,
      });
    }
  }
});
```

Types & Resolvers:

```js
import { Todos } from './collections';

load({
  typeDefs: `
    type Query {
      todos: [Todo]
    }

    type Todo {
      title: String!
    }
  `,
  resolvers: {
    Query: {
      todos() {
        return Todos.find().fetch();
      },
    },
  },
});
```

Now you can safely query:

```js
query {
  todos {
    title
  }
}
```

Let's create a todo:

```js
type Mutation {
  todoInsert(todo: TodoInsertInput!): String
}

type TodoInsertInput {
  title: String!
}
```

```js
{
  Mutation: {
    todoInsert(_, args) {
      const {todo} = args;
      Todos.insert(todo);
    }
  }
}
```

```js
mutation todoInsert($todo: TodoInsertInput!) {
  todoInsert(todo: $todo)
}
```

Variables:

```js
  {
    "todo": {
      "title": "My custom Todo"
    }
  }
```

# Convenience

`cultofcoders:apollo` allows you to avoid thinking about collections at first, allowing you to create them inside the type:

```js
type Todo @mongo(name: "todos") {
  title: String
}
```

# Database via context

Inside your resolver you have access to a god object `db` from which you can access all your created databases:

```js
{
  Query: {
    todos(_, args, context) {
      const { todos } = context.db; // todos representing the collection's name

      return todos.find().fetch();
    }
  }
}
```

Exercies:

1. Add parameter called `search` to the `todos` query that searches for todos that contain the title (case insensitive)
2. Extend the same todo query to allow pagination with pageSize, and pageNumber.
3. Add to the ToDo input a `isChecked: Boolean`

# Useful Packages

- Collection Hooks : https://github.com/matb33/meteor-collection-hooks
- Collection Schema : https://github.com/aldeed/meteor-collection2
- Collection Behaviors : https://github.com/zimme/meteor-collection-behaviours

