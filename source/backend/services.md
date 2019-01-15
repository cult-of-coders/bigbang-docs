---
title: Services
description: 'Learn to decouple logic in your GraphQL app'
disqusPage: 'Backend:Services'
---

# Services

Well, well, well. We finally got here... the place where you, as a backend developer is going to spend most of the time.

Remember those resolvers for Query and Mutation? Well.. abstract them fully into services:

```js
// BAD
export default {
    Mutation: {
        addPost(_, args, context) {
            // security checks
            // lots of logic
        }
    }
}
```

```js
// GOOD
import PostService from './services/PostService';

export default {
    Mutation: {
        addPost(_, { post }, { userId }) {
            PostService.addPost(userId, post);
        }
    }
}
```

We do this because we want to be able to test our code without going to the GraphQL Playground manually.


## What is a service?

A service is a unit of work (a function) or a group of very related functionality (a class)

Using services is linked to the [Single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle)

```simple-text
Q: So when do we need to use services?
A: Everytime!
Q: Where do we use services?
A: Everywhere!
```

Ok we're done here, everything is clear now, right?

Let's say you create a Post, and you need to notify the Admin to approve it.
Where do you store that unit of logic ? Inside a resolver ? Inside a function ? Inside a class ?

The answer is: **NOT inside the resolver**

Usually, you tend to couple logic in your resolver which is a very very bad terrible thing, because the resolver is a `Controller` it should not know implementation details.

It is also bad because resolvers are a proxy layer between the client and the server, they shouldn't store business logic, or logic of any kind.

Let's start focusing about services in depth, and understand some nice principles about
crafting them in a nice way.

Don't forget to read: https://github.com/ryanmcdermott/clean-code-javascript

## Sample

```js
class ItemService {
  static createItem() {
    // put logic here for item creation
  }

  static updateItem(id, data) {
    const item = this._getItem(id);
    // do something with it
  }

  static getItem(itemId) {
    // returns the item from database or throws an exception
  }
}

export default ItemService;
```

In most cases, you want your services to be a class with static methods, and not an instance of that class `new ItemService()`, however
we'll see below why in some cases using the instance makes more sense.

## Structuring

By default we are going to put them inside:
`/src/modules/{module}/services`

Usually {module} represents the collection name it handles it, but this is not always the case, it can represent something that is handling multiple collections to achieve its goal.

If your services need more decoupling feel free to nest them:
`/src/modules/{module}/services/{submodule}`

Name your services with uppercase if classes (ItemService), or lowercase if functions (doSomething).
If your service is a class, suffix it with service, if it's a function, make the sure the filename is a verb.

Inside a function service module, you can create additional functions, but you must export only one, if you need to
export multiple functions, it will become a service class.

## Creating Services

Go API first. Don't try to think about the logic, try to think about how you are going to use it. This is why [**TDD**](https://technologyconversations.com/2013/12/20/test-driven-development-tdd-example-walkthrough/) works so well, because
it lets you think about the API and you also write a test, and passing tests means you finished implementation.

So, instead of patterns, try to think about how you would use it, and just write a test for it first. You will notice that your development speed will increase dramatically (after first tries). No kidding!

#### Helpful questions:

1. What would be the cleanest, easiest way to use this Service?
2. How can I make it so it's easier to understand by others?
3. What comments can I leave so the next developer that comes in understands this?
4. Does my service have a single responsability?
5. Is there any functionality in my service that is outside its scope so I can decouple it?

## Conventions

Again, don't forget to read: https://github.com/ryanmcdermott/clean-code-javascript. I'm not joking, read it, daily, until it's in your bloodstream.

#### Functions should be actions or interrogations

Please favor longer names for small scope, lower names for large scopes.

```js
// BAD
object.author();
object.veryBad();
object.closing();
object.mkSmtGdAbtIt();
```

```js
// GOOD
object.getAuthor();
object.isGood();
object.close();
object.makeSomethingGoodAboutIt();
```

#### Variable names should be substantives or interrogations

```js
// BAD
let making;
let made;
```

```js
// GOOD
let isMaking;
let isMade;
```

#### Functions should be small

A function should not be larger than 7-10 lines. Honestly, if you have that, then it must be refactored. Exceptions being, large switch cases, Pure Components and render() functions from React.

#### Classes hide in large functions

There are cases where you have a single function as service, and that function begins to grow,
and you find that decoupling it in multiple functions gets tedious because you will need to pass
many variables along, and you realize that those variables are strings or numbers and can't be properly modified,
so you need to return them, and even if your intentions are pure, your code will become
more messy.

The rule is simple: when it's difficult to decouple a function, create a class. Because inside a class, each method
has access to `this` context, so you no longer need to pass variables along.

#### Don't extend classes

Another opinionated advice, just don't use extend. The only exception is if the class you extend
is abstract (you don't instantiate/use it stand-alone).

#### Provide JSDoc and Comments

Whenever you feel like you are doing something that is not verbose, leave a comment, and also leave JSDoc, especially for
variables. It becomes truly helpful when you are using an API written by someone else and the IDE shows you which variables it needs and a description for them.

If you find a snippet, or something that describes a certain thing, don't be afraid to leave links,
especially if you're writing an facade to an external API, leave link to the docs, it will save you and other devs
time.

Leave as much comments as you can, but don't go off the grid, stay to the point. And only leave comments when necessary:

```js
// BAD (The code already tells you what you do)
// We are iterating through users
users.forEach(user => {});
```

```js
// GOOD (Even if you can read it nicely, you need to understand the intention fast)
// We are calculating the total cost of all products so we can use it in total calculation cost.
let totalCost = 0;
products.forEach(product => {
  totalCost += product.cost;
});
```

## Dependency Injection

Inversion principle is simple, let the service decide what dependencies it has. We are kinda doing this already
because each module imports what it needs in order to execute properly.

The reason for changing it a bit is the fact that it will allow easy testing. For example, we want to test that `PaymentService.charge` is been called when something happens.

```js
import PaymentService from 'somewhere';

class ItemService {
  static createItem(item) {
    Items.insert(item);
    PaymentService.charge(item.userId, 20.0);
  }
}

export default ItemService;
```

Now in my test how would I be sure that `PaymentService.charge` is called without actually altering `PaymentService`?
This requires a change in strategy, by injecting dependencies in the constructor:

```js
import PaymentService from 'somewhere';

class ItemService {
  constructor({ paymentService }) {
    this.paymentService = paymentService;
  }

  createItem(item) {
    Items.insert(item);
    this.paymentService.charge(item.userId, 20.0);
  }
}

export default new ItemService({
  paymentService: PaymentService,
});

export { ItemService };
```

Ok now if we would like to test it, we have access to the exported variable: `ItemService` and we can pass-in a [stub](http://sinonjs.org/releases/v4.0.1/stubs/) for `PaymentService` in its constructor.

The only problem with this pattern is the fact that it creates an instance without you wanting to,
therefore you can create a separate `ServiceModel` file (ItemServiceModel.js) and instantiate (and export) it inside `ItemService.js`, and when you test you only need to play with `ItemServiceModel`, or you can have a `container.js` file that instantiates all services and exports them.
