# Overview
This document is intended to provide guidelines and examples used across the partition of a big monolithic app into separate micro-services using [REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm).

The concepts explained here are not tied to any technology or framework in particular (rails, JS, C#, etc) and are the result of our learnings while working on this awesome project.

We hope this document is of interest and useful to API designers.

This document borrows concepts and guidelines heavily from:
* [RESTful Web Services](http://www.amazon.com/RESTful-Web-Services-Leonard-Richardson/dp/0596529260)
* [REST API Design Rulebook](https://www.amazon.com/REST-Design-Rulebook-Mark-Masse/dp/1449310508/ref=sr_1_1?ie=UTF8&qid=1508110170&sr=8-1&keywords=rest+api+design+rulebook)
* [Restful Web Services Cookboox](http://www.amazon.com/RESTful-Web-Services-Cookbook-Scalability/dp/0596801688)
* [Heroku API Style Guide](https://github.com/interagent/http-api-design)
* [Service-Oriented Design with Ruby and Rails](http://www.amazon.com/Service-Oriented-Design-Addison-Wesley-Professional-Series/dp/0321659368)

# Contributions
API design is a team sport. We welcome [contributions](CONTRIBUTING.md).

# Contents
- [Introduction](#introduction)
  - [REST](#rest)
  - [REST APIs](#rest-apis)
- [Identifying REST Resources](#identifying-rest-resources)
  - [Resource Types](#resource-types)
    - [Singleton resource](#singleton-resource)
    - [Collection resource](#collection-resource)
    - [Controller resource](#controller-resource)
  - [Resource addressability](#resource-addressability)
  - [Resource representation](#resource-representation)
- [API Design](#api-design)
  - [Foundations](#foundations)
    - [Include versions in URIs](#include-versions-in-uris)
    - [Provide request ids for introspection](#provide-request-ids-for-introspection)
    - [Healthchecks](#healthchecks)
  - Partitioning functionality into APIs
  - [Requests](#requests)
    - [URI Paths](#uri-paths)
    - [HTTP Verbs](#http-verbs)
  - [Responses](#responses)
    - [HTTP Status Codes](#http-status-codes)
    - [Response Body](#response-body)
- Load Balancing
- Caching
- [Security](#security)
  - [Authentication](#authentication)
  - [Authorization](#authorization)
- [API Documentation](#api-documentation)
- [Recipes](#recipes)
  - [Filtering Resources](#filtering-resources)
  - [Paginating Resources](#paginating-resources)
  - [Sorting Resources](#sorting-resources)

# Introduction

## REST
[REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) stands for Representational State Transfer, and it is an architectural style for distributed hypermedia systems. Ir was described by Roy Fielding in 2000 in his doctoral dissertation.

A REST Application Programming Interface (REST API) is a type of web server that enables a client, either user-operated or automated, to access resources that model a system’s data and functions.

## REST APIs
Web clients use APIs to communicate with Web Services. An API exposes data and functions to facilitate interactions between computer programs and allow them to exchange information.

The REST architectural style is commonly applied to the design of APIs for web services. A Web API conforming to the REST architectural style is a REST API, and it's said to be RESTful.

A REST API consists of a set of interlinked resources. This set of resources is known as the REST API's resource model. A resource is an object with a certain type, associated data, relationships to other resources, and a set of methods that operate on it.

This style guide exposes our conventions for identifying resources and how the API make use of them.

# Identifying REST Resources
Resources are the heart of the design of services. At the most basic level, a resource is an object that represents something that could map to a record in the database, a file, a searh result, or even a procedure.

## Resource Types
There are 3 basic types of resources that can be defined:
- Singleton resource
- Collection resource
- Controller resource

### Singleton resource
A singleton resource is akin to an object or a database record.

For instance, the following URIs identify singleton resources:
```
https://my-domain.com/accounts/123
https://my-domain.com/accounts/123/subaccounts/321
```

### Collection resource
A collection resource is a collection of other resources.
If we were considering a collection of accounts, each account by itself
is a singleton resource.

For instance, the following URIs identify collection resources:
```
https://my-domain.com/accounts
https://my-domain.com/accounts/123/subaccounts
```

### Controller resource
Controller resources model procedural concepts, similar to executable
functions in the programming world.

Usually in REST design, these kind of resources are the ones that cannot
be mapped to standard CRUD methods.

The name chosen for a controller resource typically appears as the last segment in a URI path, with no child
resources to follow them in the hierarchy. The following example shows a controller resource
that allows a client to resend an invoice.

```
POST https://my-domain.com/invoices/123/resend
```

## Resource addressability
Every resource needs a way to be referenced (or addressed). Uniform
Resource Identifiers (URIs) provide a way of doing this.

For instance, a URI that addresses an account resource could be:

```
https://my-domain.com/accounts/123
```

Each resource MUST have at least one URI that will identify it and
provide a way of addressing it. Keep in mind that a resource could be
referenced by more than more URI, but we encourage not to do this
unless it is really necessary.

## Resource Representation
Resources need a way to be represented. A representation is just the
stream of bytes that exposes the resource.

We say that a representation is at the same time the format that the
resource can take, and resources can have more than one representation.
For instance, text, HTML, JSON, XML, JPG, etc, or a custom format
defined by us.

# API Design

## Foundations

### Include versions in URIs
Remember than an API is a contract between a publisher and a developer. To prevent surprise and breaking changes to users, it's best to require a version be specified in the URI of each request.

We suggest including the version information in the URI of every service request. As always, there are trade-offs. Other API designers prefer a more RESTful approach by using the `Accept` HTTP header to specify the version.

```
https://my-domain.com/api/v1/accounts
```

API versions should not change often. But when they do it,they should change the major version only:

```
https://my-domain.com/api/v2/accounts
```

### Provide request ids for introspection
We suggest API designers to trace HTTP requests all the way from the
client to the backend. This can be done by including a custom `X-Request-Id` HTTP header in each API response, populated with a Universally Unique IDentifier (UUID) value. By logging these values on the client, server and any backing APIs, this provides a mechanism to trace, diagnose and debug requests.

As this ID is randomly generated by the client, it does not contain any sensitive information, and should thus not violate the user's privacy. At the same time, this does not help with tracking clients because the ID is created per request and not per client.

### Healthchecks
Sometimes a service instance can be incapable of handling requests yet still be running. For instance, it might have ran out of DB connections. When this occurs, the monitoring system should generate a alert.

In order to detect that a running service is unable to handle requests, we should provide healthcheck endpoints.
This will allow us to keep the impact on the performance of an application as low as possible.
Using business endpoints for this purpose would involve unnecessary processing time.

It's useful to expose different endpoints for different checks.
Each project will have a different set of checks depending on its stack of technologies, and they will be available to be checked separately.

Each of these checks can pass (response status `200`) or fail (response status `500`).
It's possible to define custom checks in case needed.

```http
Request: GET https://my-domain.com/health/database.json

Response: 200 OK
Body:
{
  "database": {
    "message": "Schema version: 20160317201558",
    "success": true,
    "time": 0.0123434231
  }
}
```

```http
Request: GET https://my-domain.com/health/redis.json

Response: 500 Internal Server Error
Body:
{
  "redis": {
    "message": "Error: 'getaddrinfo: nodename nor servname provided, or not known'",
    "success": false,
    "time": 0.0123434231
  }
}
```

There is also the possibility to check them all, getting a response with the summarized status.
This endpoint will respond with status `200` if all checks pass.

```http
Request: GET https://my-domain.com/health/all.json

Response: 500 Internal Server Error
Body:
{
  "default": {
    "message": "Application is running",
    "success": true,
    "time": 0.00000999998883344233
  },
  "database": {
    "message": "Schema version: 20160317201558",
    "success": true,
    "time": 0.013348999986192212
  },
  "redis": {
    "message": "Error: 'getaddrinfo: nodename nor servname provided, or not known'",
    "success": false,
    "time": 0.0014479999954346567
  },
  "app_version": {
    "message": "Version: 1.2.0",
    "success": true,
    "time": 0.0009160000190604478
  }
}
```

## Partitioning functionality into APIs

## Requests

### URI paths
URIs identify REST resources, they are the public interface of a
API. Well-designed URI patterns make an API easy to consume, discover
and extend.

We encourage you to build URIs for your resources following these
guidelines:
- Use only nouns or verbs for the paths.
- Use:
  - singular nouns for singular resources.
  - plural nouns for collection resources.
  - verbs for controller resources.
- Describe resources in a language familiar to the business.
- Use `snake_case` syntax.
- Avoid deep nesting.

Resources that are deeply nested are generally painful to consume. You
shouldn't need more than two levels deep:

```
GET /accounts  # :)
GET /accounts/123/subaccounts  # :)
GET /accounts/123/subaccounts/321  # :)
GET /accounts/123/subaccounts/321/invoices  # :(
```

### HTTP Verbs
The Uniform Interface is a big constraint at the time of implementing a
REST API. It's goal is to make the interaction between components as
simple as possible.

HTTP 1.1 provides a set of methods that are the verbs for this Uniform
Interface:
- `GET`
- `POST`
- `PUT`
- `PATCH`
- `DELETE`
- `HEAD`
- `OPTIONS`

Some of the listed verbs are safe (do not request any server-side
effects), whereas some other are idempotent (can be replayed any number
of times without a change from the first one.

We encourage to use HTTP verbs for each action that is going to affect
our resources. For instance, going back to our accounts example:

Verb | Resource | Description | Safe | Idempotent
---- | ---------| ----------- | ---- | ----------
GET | /accounts | Get collection of accounts | Yes | Yes
GET | /accounts/123 | Get account 123 | Yes | Yes
POST | /accounts | Create a new account with info sent in the body of the request | No | No
PUT | /accounts/123 | Update account 123 | No | Yes
PATCH | /accounts/123 | Partial update account 123 | No | Yes
DELETE | /accounts/123 | Delete account 123 | No | No

## Responses

### HTTP Status Codes
HTTP provides three-digit integer status codes to specify the status of a response. RESTful APIs make use of these status code to reflect the outcome of the API response:

Status Code | Description
----------- | -----------
1xx | Informational status that states that the request has been received and is continuing.
2xx | Success status. Means that the request has been received and accepted.
3xx | Redirection status. In order to complete the request, further action needs to take place at a different URI.
4xx | Client error status. The request has a wrong format or could not be completed because of another client issue.
5xx | Server error status. The server failed to complete the request.

In the next table you will find some of the status code that we suggest
using to reflect the outcome of your API responses:

Status Code | Meaning | Description
----------- | ------- | -----------
200 | ok | The request has been completed and approved. This is a generig success case.
201 | Created | The request was successful and the resource has been created.
202 | Accepted | The request has been approved and accepted for processing, but has not been completed yet.
204 | No Content | The request has been completed and approved, but there is no response body. This status code is sometimes used as a response for controller resources.
303 | See Other | The response to the request can be found at a different URI.
400 | Bad Request | The request could not be completed because it was incorrectly formatted (e.g.: syntax error).
401 | Unauthorized | The request failed because the client was not authenticated.
403 | Forbidden | The request failed because the client did not have authorization for the requested resource.
404 | Not Found | The request failed because the resource does not exist.
409 | Conflict | The request failed because of an edit conflict.
422 | Unprocessable Entity | The request failed because it contained invalid parameteters. As opposed to 400 status code, 422 is returned when no format errors are present but a semantic error is present (e.g.: validation errors).
500 | Internal Server Error | Something went wrong on the server.

### Response Body

#### Timestamps for resources
We encourage providing `created_at` and `updated_at` timestamps for
resources by default:

```json
{
  "id": 123,
  // ...
  "created_at": "2017-01-01T12:00:00Z",
  "updated_at": "2017-01-01T12:00:00Z"
}
```

The timestamps should be formatted in
[ISO8601](https://www.iso.org/iso-8601-date-and-time-format.html).

#### Successful Response Bodies
By default, response bodies hold the actual resource data (the
resource's representation). It must have a valid format, and we suggest
using JSON in case your client is the browser.

Example of an account resource:

```
Request: GET /accounts/123

Response: 200 ok
Body:
{
  "id": 123,
  "name": "Coca Cola",
  "created_at": "2017-01-01T12:00:00Z",
  "updated_at": "2017-01-01T12:00:00Z"
}
```

Example of a collection of accounts resource:

```
Request: GET /accounts

Response: 200 ok
Body:
[
  {
    "id": 123,
    "name": "Coca Cola",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  },
  // ...
  {
    "id": 900,
    "name": "GM",
    "created_at": "2016-04-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  }
]
```

#### Error Response Bodies
The response body for error conditions must be simple and self
explanatory.

Validation errors are returned in the body with a `422`
status code. The body must have a simple layout and, at the same time be
specific about the fields that have an incorrect value:

```
Request: POST /accounts
Body:
{
  "name": ""
}

Response: 422 Unprocessable Entity
Body:
{
  "name": ["cannot be blank", "is too short"]
}
```

We encourage to return validation errors in a JSON that contains a key
for each attribute name, and an array of error messages for its value.

# Security
To properly secure our APIs, we encourage you to implement Authentication and Authorization.

## Authentication
Authentication refers to verifying that the client is who she says she is and that no one has tampered with her message.

When a request fails because the client is not authenticated the status `401` (`Unauthorized`) should be returned.

## Authorization
Authorization refers to determining what a given client is allowed to do on the system.

When a request fails because the client does not have access to a specific resource, the status `403` (`Forbidden`) should be returned.


# API Documentation
API documentation is important. It explains to the clients what resources and endpoints the API is exposing, and how to consume them.
At the same time, it's a concise reference manual containing all the information required to work with the API.

[Swagger](https://swagger.io/specification/) is the standard for documenting the APIs.
A Swagger JSON file should be exposed by each API, describing its behaviour and all of its endpoints.

Swagger's specification states that by convention, the Swagger file is named `swagger.json`.

```
GET https://my-domain.com/v1/swagger.json
```

# Recipes
The following recipes are meant to help with day-to-day design and implementation questions involving REST. They should not be taken as
gospel but rather guide your discussions in solving real-world problems.

## Filtering Resources
Complex searches may require a DSL definition in order to make filtering.
When dealing with simple filterings, we can take advantage of the query params to filter a resource by its attributes.

```
Request: GET /accounts?name=Coke

Response: 200 OK
Body:
[
  {
    "id": 123,
    "name": "Coke",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  }
]
```

We could do the same for nested resources:

```
Request: GET /accounts?name=Coke&primary_contact[email]=john@doe.com

Response: 200 OK
Body:
[
  {
    "id": 123,
    "name": "Coke",
    "primary_contact": {
      "first_name": "John",
      "last_name": "Doe",
      "email": "john@doe.com"
    },
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  }
]
```

## Paginating Resources
Collection resources should be paginated to avoid big responses, and make them easier to handle.
Requests are paginated using the `page` parameter. Omitting this parameter will return the first page.

```
Request: GET /accounts?page=2

Response: 200 OK
Body:
[
  {
    "id": 123,
    "name": "Coke",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  },
  // ...
  {
    "id": 222,
    "name": "Cars Inc",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  }
]
```

We can control how many items are returned with each page using the `per_page` parameter.
We also suggest to give this parameter a default value.

```
Request: GET /accounts?page=2&per_page=2

Response: 200 OK
Body:
[
  {
    "id": 123,
    "name": "Coke",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  },
  {
    "id": 124,
    "name": "Beds Inc",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  }
]
```

### Link Header
Pagination info is included in the [Link Header](https://tools.ietf.org/html/rfc5988). It's important for consumers to follow these headers rather than construct their own URIs. Other information like `total count` and `per page` can be helpful for client applications.

```
Request: GET /accounts?page=2

Response headers:
...
Link: <https://my-domain.com/accounts?page=1>; rel="first",
      <https://my-domain.com/accounts?page=1>; rel="prev",
      <https://my-domain.com/accounts?page=3>; rel="next",
      <https://my-domain.com/accounts?page=4>; rel="last"
X-Total-Count: 100
X-Total-Pages: 4
X-Per-Page: 25
X-Page: 2
```

Available values for `rel`:
| Name | Description |
|------|-------------|
| `first` | First page's URI |
| `prev`  | Previous page's URI |
| `next` | Next page's URI |
| `last` | Last page's URI |


## Sorting Resources
`sort` and `sort_direction` are options for sorting resources. `sort` should be an attribute on the resource and `sort_direction` accepts `asc` or `desc`.

```
Request: GET /accounts?sort=name&sort_directio=asc

Response: 200 OK
Body:
[
  {
    "id": 100,
    "name": "Amazon",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  },
  {
    "id": 123,
    "name": "Coke",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  },
  ...
  {
    "id": 432,
    "name": "Zoho",
    "created_at": "2017-01-01T12:00:00Z",
    "updated_at": "2017-01-01T12:00:00Z"
  }
]
```


# TODOs
- Give example about merge resource.
- Load Balancing
- Caching
- Enhance Security section
- Recipes
