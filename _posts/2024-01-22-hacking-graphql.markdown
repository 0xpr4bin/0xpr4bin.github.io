---
title: "Hacking GraphQL API"
date:  2024-01-08 15:04:23
categories: [graphql]
tags: [graphql]
---

### GraphQL

- GraphQL is an open-source data query and manipulation language for APIs and a query runtime engine. GraphQL enables declarative data fetching where a client can specify exactly what data it needs from an API.

**Difference between Rest API and GraphQl**

- With a REST API, you would typically gather the data by accessing multiple endpoints. In the example, these could be `/users/<id>` endpoint to fetch the initial user data. Secondly, there’s likely to be a `/users/<id>/`posts endpoint that returns all the posts for a user. The third endpoint will then be the `/users/<id>/followers` that returns a list of followers per user.
- In GraphQL on the other hand, you’d simply send a single query to the GraphQL server that includes the concrete data requirements. The server then responds with a JSON object where these requirements are fulfilled.

**Queries and Mutations**

- Query is essentially used to fetch data from the servers, something alike GET requests in REST API.
- Whereas mutations used to update, create and delete the data , usually performing like POST , PATCH AND DELETE in REST API.

**Introspection Query**

- Introspection is the ability to query which resources are available in the current API schema. Given the API, via introspection, we can see the queries, types, fields, and directives it supports.
- This is the full request to perform you GraphQL introspection on your target.

```
query IntrospectionQuery{__schema{queryType{name}mutationType{name}subscriptionType
{name}types{...FullType}directives{name description locations args
{...InputValue}}}}fragment FullType on __Type{kind name description 
fields(includeDeprecated:true){name description args{...InputValue}type
{...TypeRef}isDeprecated deprecationReason}inputFields
{...InputValue}interfaces{...TypeRef}enumValues(includeDeprecated:true)
{name description isDeprecated deprecationReason}possibleTypes
{...TypeRef}}fragment InputValue on __InputValue{name description type
{...TypeRef}defaultValue}fragment TypeRef on __Type{kind name ofType
{kind name ofType{kind name ofType{kind name ofType
{kind name ofType{kind name ofType{kind name ofType{kind name}}}}}}}}
```

**Manual Introspection to find __types and __schemas**

```
query Introspection{
    __schema{
        types{
            name
          }
}}
```

- This shows all the available Operational types on the schemas.
- And To identify the available query types.

```
query Introspection {
    __schema{
        queryType{
            fields{
            name
            }
        }
   }
}
```

- To identify the available mutations types.

```
query Introspection {
    __schema{
        mutationType{
            fields{
            name
          }
        }
   }
}
```

- Now if you want to know the available fields and their names for specific query or mutation types.

```
query Introspection {
    __type(name:"User"){
            fields{
            name
    }
   }
}
```


### Finding vulnerabilities on the graphQl APIs

**Insecure Direct Object Reference (IDOR)**

```
query allUsers {
    getUser(id:3){
        id
        username
        password
    }
   }
}
```

Change the `id` to any other user id for IDOR which could lead to access another user's username and password.

**Sensitive data Exposures**

```
query Introspection {
    __schema{
        types{
            name
    }
   }
}
```

- Sometimes running this query might return sensitive query or mutation types which might be used internally or could be used to access sensitive data and manipulate them.

**Injections (sqli, xss, csti/ssti, ssrf, etc)**

```
query allUsers {
    getUser(username:"john'"){
        id
        username
        password
    }
   }
}
```

Graphql are mostly vulnerable to sql injection, as it's architecture is kind of  similar to relational database.

Many client side as well as sever side vulnerabilities could be found on the graphql APIs.
Only the syntax is hard to construct, as it is no different than REST APIs, using above manual introspection to find available query and mutation types, it would be efficient to exploit the potential vulnerabilities.
Some of the best tools for graphql testing are:

- Graphql voyager
- [InQL burpsiute extension](https://github.com/doyensec/inql)  


[References](https://raz0r.name/articles/looting-graphql-endpoints-for-fun-and-profit)
