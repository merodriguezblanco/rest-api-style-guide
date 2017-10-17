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
API design is a team sport. We welcome contributions.

# Contents
- [Introduction](#introduction)
  - [REST](#rest)
- Identifying REST Resources
- Recipes

## Introduction

### REST
[REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) stands for Representational State Transfer, and it is an architectural style for distributed hypermedia systems.

A REST Application Programming Interface (REST API) is a type of web server that enables a client, either user-operated or automated, to access resources that model a system’s data and functions.
