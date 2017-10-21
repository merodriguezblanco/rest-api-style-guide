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
- Recipes

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

TODO: Give example about merge resource.

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
