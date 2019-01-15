---
title: Queries & Mutations
description: 'Learn the basics about queries and mutations'
disqusPage: 'Backend:Queries-and-Mutations'
---

# Queries and Mutations

Please go through https://graphql.org/learn/, it explains really well how GraphQL can be defined. And, because we're developers and we don't want to re-invent the wheel, I will assume from now on that you have read it at least once.

### What is the difference between a query and a mutation ?

For GraphQL, in essence, there is really no difference, but it is an important distinction. An easy way to put it:

**Query** = Fetching data without altering server's state
**Mutation** = Action which may alter server's state.

Now. If you like to dig a lil' bit deeper into this definition. You may ask, what if I want to fetch data, but also increase a counter somewhere, that I fetched this data X times? Is my query still a query or is it a mutation? I believe it matters how the user perceives it that makes the definition.

Let's get back to our initial example where we loaded our data.

```js
import { load } from 'graphql-load';

load({
  typeDefs: `
    type Query {
      sayHello: String
    }
  `
  resolvers: {
    Query: {
      sayHello: () => 'Hello world!'
    }
  }
});
```

When we define our GraphQL API we have to define the way it looks via type definitions, and for each type definition inside `Query` we have to create resolvers (the logic, how it executes when it's interogated)

Using `load` function we are able to load multiple types, queries and resolvers that get automatically merged together, it's the easiest way. You can also use `load` to load just typeDefs or just resolvers. [Read more about grapqhl-load here](https://github.com/cult-of-coders/graphql-load). Everything we loaded is handled by our package, and getting passed to the `apollo-server` when we run `initialize()`, this means that if you load() stuff after you run initialize(), they will not be reflected in your GraphQL schema.

Let me help you wrap your mind around this with more examples:

Types:

```js
type Query {
  me: User
}

type User {
  name: String
  age: Int
  nameAndAge: String
}
```

Resolvers:

```js
{
  Query: {
    me() {
      // We're mocking a user here, as if it was returned from the database
      return {
        name: 'Theodor',
        age: -1, // I'm timeless
      }
    }
  },
  User: {
    nameAndAge(user) {
      return `${user.name} ${user.age}`;
    }
  }
}
```

Example of a query:

```
query {
  me {
    name
    age
    nameAndAge
  }
}
```

So. You are starting to see some similarities between defining queries and actual types. That's because there's really no difference in defining it. It's just the main `Query` type is the one that gets interogated (resolved). Same logic applies to main `Mutation` type.

A bit confusing in the beginning I know... I know... but bare with me, by the time I'm finished with you, things will make sense.

Take a look and explore how to pass-in arguments:

- https://graphql.org/graphql-js/passing-arguments/

If we would like to create a mutation. Something which we save in the database:

Types:

```js
type Mutation {
  insertPost(post: PostInsertInput): String
}

input PostInsertInput {
  title: String!
  description: String!
}
```

Resolvers:

```js
{
  Mutation: {
    insertPost(_, args) {
5. Create a query that returns only the first 5 ele
      const { post } = args; // post has the form {title, description}
      console.log(post);
      // Do some action in the database
      return 'ok';
    }
  }
}
```

Query:

```js
mutation insertPost($post: PostInsertInput) {
  insertPost(post:$post)
}
```

Query Variables:

```js
{
  "post": {
    "title": "Hello",
    "description": "Lorem ipsum..."
  }
}
```

There are the basics of creating and consuming queries.

Exercises:

1. Try to see how to define and return an array of elements of type `Integer` from a mutation. For example: `separateIntoPrimeFactors(n: Int!)`

2. For mutation instead of returning a string return a type of `MutationResponse` that contains status as `String` and errors as an array of `String`.

3. Try to create types and resolvers that would return something meaningful for this query:

```
query {
  posts {
    title
    description
    isPublished
    comments {
      text
      author {
        name
      }
    }
  }
}
```

4. How would you specify that a field is an `array`, that contains minimum 1 element of type `String`?

5. Create a query that returns the same message you passed:
```
query {
  echo(message:"Is anybody out there?")
}
```

6. Try to pass an argument to a subquery like this.

```js
query {
  me {
    fullName(withPrefix: true)
  }
}
```

And you can just return something different if `withPrefix` is true or false, doesn't matter. Be creative.

7. Create a [fragment](https://graphql.org/learn/queries/#fragments). And use it.
