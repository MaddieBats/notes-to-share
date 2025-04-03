# Production Ready GraphQL

*Building well designed, performant and secure GraphQL APIs at scale*

*Marc-Andre Giroux*

## Introduction

Prior to GraphQL becoming popular, the architecture that people were using was generally Enpoint-based APIs which revolved around HTTP endpoints, such as a JSON API over HTTP. The endpoints are simple to implement and are very good at working with one use case, are chacheable, discoverable,a dn simple to use. 

These are less good if you have to serve multiple use cases. People started trying to make one-size-fits-all APIs which tried to do too much at once. One solution was to have a different endpoint for each variation. There were also aproaches that had one endpoint per use case but had a query parameter for each different type, and some tried to just use partial and full queries. Other approaches included letting clients select what they wanted back from the server or writing the query language in the query parameter. There was then a movement to make a fully customisable API that was actually good for developers. 

SoundCloud tried to solve this by having a different server for each sort of user (called Backends for Frontends).

Facebook announced the GraphQL release in 2015

What GraphQL is not:
* It is not a graph database
* It is not a library
* It is not about graph theory

It is instead *a specification for an api query language and server engine that is capable of executing diverse API queries*.

Hello World in GraphQL:

```graphql
query {
  me {
    name
  }
}
```

This asks for the current user and their name using the `me` and `name` fields.

This is sent to a GraphQL server as a simple string, and would return:

```graphql
{
  "data": {
    "me": {
      "name": "Maddie"
    }
  }
}
```

The response and query are similar shapes, with the response always having a `data` key with the response for client inside.

GraphQL allows you to select more than simple fields, i.e. in:

```graphql
query {
  me {
    name
    friends(first: 2) {
      name
      age
    }
  }
}
```

This fetches my name and the names and ages of two of my friends. 

Fields can be considered like functions in that they can take arguments and return a certain type.

This would give a response like:

```graphql
{
  "data": {
    "me": {
      "name": "Maddie",
      "friends": [{
        "name": "Mia",
        "age": 28
      }, {
        "name": "Allie",
        "age": 23
      }]
    }
  }
}
```

The `query` keyword at the start of the query tells the GraphQL server that we want to query the query root of the schema.

The GraphQL server uses a type system or schema, usually in the GraphQL Schema Definition Language (SDL), which is itself language agnostic, and always describes the final schema.
