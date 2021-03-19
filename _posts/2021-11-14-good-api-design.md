---
layout:     post
title:      Practical Advice on Good API Design
date:       2021-11-14
summary:    Developing an API takes time. This post aims to give you a glimpse of some best practices for API design.
categories: API
thumbnail: API
tags:
 - API
 - design
 - rest
 - http
---
When designing a good API, always start with requirements. Before writing a single line of documentation, do as much research as possible about what your API should do. Think not only about the service side, but also who is going to be your consumer and what would be most important for them. I highly recommend creating a list of requirements as well as restrictions to give the end result a better shape.

Start by creating lists like security requirements, client requirements, maintenance requirements. Pull together your design based on these. For this, you can use tools like OpenAPI or API Blueprint.

In this article I summarize dos and don’ts for multiple areas of API creation: general best practices, security concerns, schema implementation, and headers usage. Let’s dive into it.

## General Best Practices

### Usefulness
The API should provide the desired functionality — something that users need or want — and not just exist for the sake of checking a box. This also means that you should have some understanding of what users are trying to accomplish and why, so you can make reasonable tradeoffs and design decisions.

### Favor simplicity
There is frequently a tradeoff in APIs between “simple to understand/simple to use” and “powerful and flexible.” Unless there’s a good reason to do otherwise, you should favor simplicity in your APIs. This doesn't mean that you won’t provide support for power users — that can be done by allowing overridable defaults or additional API endpoints.

### Principle of Least Surprise
Your APIs should avoid side effects, revealing odd implementation details, or changes in info representation.. For example, you don’t want your APIs to display data sometimes as floats, and sometimes as integers — there should be one format users can expect. Other labels such as amount, count, size, etc. should also stay consistent.

### Consistency
Keep the structure of your APIs consistent. If you return wrapped objects, make sure you always return the same wrapper. Errors, Lists, Pagination, Single Resource, etc. should always have the same structure so they are easy to unpack. When naming keys, consistently name them across different endpoints. When designing a new API, create a list of requirements for this boilerplate so it might be used from day one.

### Separate concerns
Endpoints should do only one thing. Try to avoid endpoints that are doing more than one domain-specific atomic action.

**Do ✔️**

```
Return Location header with the address to create a resource
```

Don't ❌

```
Combine the responsibility of the GET and POST endpoints. POST shouldn't return a created object as its only responsibility is creation, not projection.
```
## Security Points
### Avoid exposure of internal IDs
Create business IDs — for your API. IDs should be random to prevent potential guess attempts. Refer to OWASP’s cheat sheets on Insecure Direct Object Reference for more information.

### Define clear access rules
When designing an API, clearly define what would be your authorization pattern. What roles exist and what resources they can access. Define your resources in OpenAPI and check if you’re not leaking anything as a nested resource to an unauthorized client.

### Automate the review process
If possible, automate security checks for the API. Use tools like OWASP Zap to find problems as early as possible.

### Use OWASP guidelines for headers
Refer to OWASP’s Secure Header Project for more information.

### Include abuse cases research in the review process
Finding security concerns during design allows you to fix problems before they occur. Solving security problems on deployed APIs may lead to breaking changes which are expensive to implement. You can find more information here.

## Schema Implementation
### Pagination
The main purpose of pagination is to let the consuming service know if there are more data points to fetch. Make it as easy to use for end customers without exposing details.

**Do ✔️**
```
{
  "data": [...]
  "pagination": {
    "previous": null,
    "next": "/items?pagination=78ce5de5217340f7"
  }
}
```
- Customers are aware there are more data points
- Customers are aware they are at the beginning of the iteration
- It’s easy to jump to the next page

Don't ❌

Ask the customer to build a URL based on parameters.
```
{
  "data": [...]
  "pagination": {
    "max_pages": 10,
    "current_page": 2,
    "current_limit": 10
  }
}
```
Which then user builds into:

`/items?page=3&limit=10`

This could cause customers to skip some data as they change limits or to wrongly parse the current page. Caching might be harder to implement due to more unpredictable requests.

### Handling nullability - single value
Be consistent about nullability. The best practice is an explicit “null.” Avoid dropping keys or using empty strings or 0. This makes it easier for clients to parse responses.

**Do ✔️**
```
{
  "name": "Example book",
  "score": null
}
```

**Don’t ❌**

```
{
  "name": "Example book",
  "score": 0
}
```
In the “don’t” example the client has to guess if there are no reviews or the book is really bad and deserves a score of 0.

### Handling nullability - array
Whenever returning a nested array inside an object, use an empty array over “null.”

**Do ✔️**
```
{
  "name": "Example book",
  "reviews": []
}
```

**Don’t ❌**
```
{
  "name": "Example book",
  "reviews": null
}
```
### Clear versioning
It should be easy for consumers to understand versioning and how the current version compares with previous iterations. Let’s work with the standard semantic versioning to explain my point. You have version 0.8.4, with 0 being major, 8 being minor, and 4 being patch.

When customers see a difference in major versioning, they usually expect a breaking change. With minor versioning, they look for new functionalities without breaking the contract. And with patches, it’s mostly small updates.

Here are some examples.

**Breaking changes:**

- Delete any response field
- Rename any field
- Change in enums
- Add new mandatory request field

**New endpoints:**

- A new method for existing endpoint eg. POST for existing GET endpoint
- New endpoint

**Extensions of existing endpoints:**

- Add a new optional request field
- Add response field

### Extendability
APIs should be extensible. That means whenever the API returns something, you should take into consideration it might need to return more information in the future.

**Do ✔️**
```
{
  "categories": [{ "id": 1 }, { "id": 2 }, { "id": 3 }]
}
```

**Don’t ❌**
```
{
  "categories": [1, 2, 3]
}
```
### Developer friendly
Think about APIs as something used by humans, not machines. Whenever something goes wrong, it will be a person evaluating this and you should make it as easy as possible for them.

**Do ✔️**

```
{
  "error": {
    "code": "validation",
    "description": "Id provided doesn't match pattern ... Please refer to documentation for more details about formatting"
  }
}
```

**Don’t ❌**

```
{
  "error": "validation"
}
```
### Idempotent
APIs should behave in an idempotent way. This means whenever you allow users to mutate data, multiple requests should not create different side effects. An example might be a CREATE request. Triggering 20 identical requests shouldn't create 20 entities unless explicitly specified by the user. To prevent that, you could allow users to generate an ID or incorporate a Mutation-Check header to prevent the same requests. As a response, the API could return a 409 Conflict.

**Example:**
```
POST /books HTTP/1.1
Mutation-Check: 4ab1030b-6020-4856-834b-7c4be91293a3

Body: 
{
  "name": "Example book"
}
```
This request should be allowed to pass only once. The next attempt to create an example book with the same Mutation-Check should fail.

### Headers usage
Allow multiple projection layers
APIs should be centered around concerns/domain, not the presentation layer. It might be that different clients require different projections (eg. server ↔︎ server and server ↔︎ mobile communication). I suggest using headers to distinguish which projection layer as well as version is required by the client.

**Do ✔️**

Create multiple accept versions: `Accept: application/vnd.example.v1+json Accept: application/vnd.example.v2+json Accept: application/vnd.example.v1-ios+json`

**Don’t ❌**

Push all edge cases into a generic API.

### Use headers to provide meaningful context
Headers could be used to store more information not related to content. E.g.

Location of the newly created resource
Request limit
Request ID (issue-resolving/tracking)
## Summary
Developing an API takes time. This post aims to give you a glimpse of some best practices for API design. It's not exhaustive, but it should provide a good start to creating your own APIs that are intuitive and easy to use. What other guidelines do you think are important?


Note post cross-published: https://www.cobalt.io/blog/practical-advice-on-good-api-design