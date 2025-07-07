---
title: "GraphQL Basics"
date: "2025-07-07"
categories: [Worse Quality]
tags: [GraphQL, API]
---
# Introduction
This article tells you about basic concepts in GraphQL, these will help you understand more complex definitions or explanations of a bug or an attack type. I tried to make it as short as possible, to learn more you can see at a resources section.
# Introspection Schema
Introspection schema is a GraphQL feature that helps users of an API see what action they can perform in the endpoint, it is usefull for developers as it enables them to dynamically explore object types, mutations, queries, and fields supported by the server. Iformation from the schema can be retirieved by introspection queries here are three types of them:
- __schema Introspection query
- __type Introspection query
- __typename Introspection Query
### __schema Introspection query
This query is used to fetch what operations are available, here is example of how this could look like:
```GraphQL
{
  __schema {
    types {
      name
    }
  }
}
```
response:
```JSON
{
  "data": {
    "__schema": {
      "types": [
        {
          "name": "Int"
        },
        {
          "name": "String"
        },
        {
          "name": "Student"
        },
        {
          "name": "Teacher"
        },
        {
          "name": "__Type"
        },
        {
          "name": "__Field"
        }
      ]
    }
  }
}
```
- The name String and Int are Scalar type, Scalar type is a basic piece of data — like a single number, a word, or true/false, that can’t be divided into smaller parts inside a GraphQL query.
- The names Student and Teacher are object types. An object type is like a group of related information made up of different fields. Each field can be a simple value (like a number or text) or another object. This helps organize data in a clear, structured way.
### __type Introspection query
The __type query is used to ask the GraphQL server for detailed information about a specific type in the schema, here is example of how this could look like:
```GraphQL
{
  __type(name: "Student") {
    name
    kind
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```
Response:
```JSON
{
  "data": {
    "__type": {
      "name": "Student",
      "kind": "OBJECT",
      "fields": [
        {
          "name": "id",
          "type": {
            "name": "ID",
            "kind": "SCALAR"
          }
        },
        {
          "name": "name",
          "type": {
            "name": "String",
            "kind": "SCALAR"
          }
        },
        {
          "name": "age",
          "type": {
            "name": "Int",
            "kind": "SCALAR"
          }
        },
        {
          "name": "enrolledCourses",
          "type": {
            "name": null,
            "kind": "LIST"
          }
        }
      ]
    }
  }
}
```
### __typename Introspection Query
__typename is a special field in GraphQL that returns the name of the object type for the data being queried. When you include __typename in a query, the response will show which object type each part of the data belongs to. This is useful for developers to understand exactly what kind of object they are receiving in the response.
Example:
```GraphQL
{
  __typename
  product(id: "2025") {
    __typename
    name
    cost
  }
}
```
Response:
```JSON
{
  "data": {
    "__typename": "Query",
    "product": {
      "__typename": "Product",
      "name": "Moisturizer",
      "cost": 29.99
    }
  }
}
```
# Anatomy
**GraphQL Document** is a string written in the GraphQL language that defines one or more operations and fragments. Below is example GraphQL Document:
```GraphQL
{
  product(id: "2025") {
    name 
    cost(unit: USD)
  }
}
```
In this example product, name, and cost are called Fields; id, cost are called Arguments.
- A field - piece of data you can query on a GraphQL type. Fields define what data is available.
- Arguments - allow clients to pass input to fields or operations (like filtering, formatting, or selecting a unit).

**Operation** is single query, mutation, or subscription that can be interpreted by a GraphQL execution engine. Below is example GraphQL operation:
```GraphQL
query Search ($category: Category) {
  product(id: "2025", shelf: $shelf) {
    name
    cost
  }
}
```
In this example query is type of operation we are performing, Search is operation name and ($category: Category) is variable definition
- Operation name - an optional identifier you can give to a query, mutation, or subscription it is useful for debugging, logging, caching, or identifying the request.
- Variable definitions - declared in the operation header, just after the operation name. They allow you to pass dynamic values into your query instead of hardcoding them.

Variables are sent independently of the query document, using a format specific to the communication method. In most modern GraphQL servers, this format is typically JSON.
```JSON
{
  "category": "FOOD"
}
```
# Operation Types
- Query - type of operation used to request data from the server it's the most basic and common way to interact with a GraphQL API.
- Mutation - type of operation used to modify data on the server, such as creating, updating, or deleting records.
Example mutation:
```GraphQL
mutation AddToCart($productId: ID!, $quantity: Int!) {
  addToCart(productId: $productId, quantity: $quantity) {
    success
    message
    cart {
      totalItems
      totalPrice
    }
  }
}
```
- Subscription - type of operation used to receive real-time updates from the server when specific events occur, typically over a persistent connection like WebSockets.
Example subscription:
```GraphQL
subscription OnMessageAdded {
  messageAdded {
    id
    content
    author {
      name
    }
  }
}
```
# Aliases
An alias in GraphQL allows you to rename the result of a field in the response it is useful when you want to query the same field multiple times with different arguments, or when you just want a different name in the response.
Example alias:
```GraphQL
{
  usPrice: product(id: "2025") {
    name
    cost(unit: USD)
  }
  euPrice: product(id: "2025") {
    name
    cost(unit: EUR)
  }
}
```
Response:
```JSON
{
  "data": {
    "usPrice": {
      "name": "Moisturizer",
      "cost": 29.99
    },
    "euPrice": {
      "name": "Moisturizer",
      "cost": 26.50
    }
  }
}
```
# Conclusion
In this article we touched a lot of interesting things, though i didn't talk about everything and will probably need to re write this post.
# Resources
"Black Hat GraphQL: Attacking Next Generation APIs" by Nick Aleks and Dolev Farhi

https://www.geeksforgeeks.org/graphql/introspection-in-graphql/

https://www.apollographql.com/blog/the-anatomy-of-a-graphql-query
