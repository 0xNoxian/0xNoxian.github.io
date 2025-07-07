---
title: "GraphQL-Introspection"
date: "2025-07-04"
categories: [Worse Quality]
tags: [GraphQL, API]
---
# GraphQL Introspection Hacking
GraphQL is a single-endpoint API that is based on a schema to operate, if you had the schema you would have, in a sense, documentation of that API. GraphQL has a functionality called introspection that can retrieve the schema, allowing you to easily obtain API documentation in the process. This all sounds awesome but usually introspection queries are disabled by developers because they don't want to give hackers free documentation of their API. This article is focused on bypassing those restrictions and retrieving the schema anyway, so if your current target has introspection queries enabled you are in luck and don't need this article.

Introspection, along with other features, is enabled or disabled by default in some implementations, below is a table from https://github.com/nicholasaleks/graphql-threat-matrix (it is being updated — I'm throwing this in case this article gets outdated):
legend:
✅  - Enabled by Default
⚠️  - Disabled by Default
❌  - No Support 

![image](https://github.com/user-attachments/assets/ef3c892d-ece4-44fe-88e4-4eb3ea2b0b21)

# Field Suggestions
You can see they are also mentioned in the table above. Field suggestions are used to make it easier to resolve errors in queries. While this is great and useful, unfortunately it can be used to enumerate those fields. Here is an example of how this might look in the wild:Not valid query
```GraphQL
query {
  usre(id: 1) {
    name
  }
}
```

Error message
```JSON
{
  "errors": [
    {
      "message": "Cannot query field \"usre\" on type \"Query\". Did you mean \"user\"?",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ]
    }
  ]
}
```
Field sugestions can be bruteforced usually using wordlists that consist of top-<number>-<language>-words, where number is how deeply we want to dig for queries and language is speaking language of the app, if multiple are supported i recommend using english

We can alias our queries in order to avoid being rate limited. Aliasing requests is also mentioned in the table above. Here is how that would look:
```GraphQL
query {
  a1: user { __typename }
  a2: me { __typename }
  a3: account { __typename }
  a4: login { __typename }
  a5: getUser { __typename }
}
```
response
```JSON
{
  "data": {
    "a2": { "__typename": "User" },
    "a4": { "__typename": "LoginResponse" }
  },
  "errors": [
    {
      "message": "Cannot query field \"user\" on type \"Query\". Did you mean \"User\"?",
      "path": ["a1"]
    },
    {
      "message": "Cannot query field \"account\" on type \"Query\".",
      "path": ["a3"]
    },
    {
      "message": "Cannot query field \"getUser\" on type \"Query\".",
      "path": ["a5"]
    }
  ]
}
```
# Tricking Regex
There might be a scenario where introspection isn't disabled, but the introspection query is being filtered out. In that case, we can use multiple methods to trick regex:
- Newline your Payload
```GraphQL
{
  __sc
  hema {
    types {
      name
    }
  }
}
```
Some regex might see this as a new word, but it will get processed as a normal __schema.
- Url Encoding

If passed in url you can url endode or double url encode __schema:
```URL
%255f%255fschema
```
- Be Creative

A lot depends on a context here, so research and use creativity
#  __type Meta-field
most introspection queries leverage the __schema
meta-field, clients could also use several other introspection meta-fields.

Example:
```GraphQL
{
  __type(name: "Query") {
    name
  }
}
```
If the response looks like this, you know that this field is allowed:
```JSON
{
  "data": {
    "__type": {
      "name": "Query"
    }
  }
}
```
You can check if some other fields are allowed and try to query as much as you can, developers might oversight this.
# Use The App
This one is more tedious, as you have to click through the entire app, different functionalities, pages, etc. But you will likely do it anyway when pentesting a web app, so you can kill two birds with one stone. After this reconnaissance, you would need to create documentation manually or with tools (if they exist). You won’t extract the entire schema from this method, but it’s at least something.
# Inspect Server Files
This point is pretty self-explanatory, you need to inspect JS files for API calls. You can discover API endpoints, keys, and queries that way. This is a pretty obvious step, so I’ll try to add value by recommending a really good tool to pull all files from the web server to VS Code. It’s called jxscout, a really good tool that works with Caido, Burp, and probably other proxies if you configure it accordingly:
https://github.com/francisconeves97/jxscout
# Resources
"Black Hat GraphQL: Attacking Next Generation APIs" by Nick Aleks and Dolev Farhi
https://github.com/francisconeves97/jxscout
