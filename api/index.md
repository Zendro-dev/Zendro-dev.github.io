---
layout: default
title: Zendro API
nav_order: 7
has_children: true
permalink: /api
---

# Zendro API
{: .no_toc }
Given a data scheme described using our [custom format]({% link data-models/json-specification.md %}), the Zendro backend generator implements a default CRUD API accessible through a well-known GraphQL query language. To learn about GraphQL queries and mutations in general, see the [official documentation](https://graphql.org/learn/queries/). When the back-end server is up, the GraphQL service is accessible at `http://<back_srv>/graphql`, accepting POST requests with authentication information in the header, and a request body following the GraphQL standard.
{: .fs-6 .fw-300 }

Zendro's back-end server implementation follows the GraphQL convention of referring to a request that doesn't change any data as a *query*, and one that modifies data as a *mutation*.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
### Access permissions

By default, the back-end server ignores all user permissions (access control is off), so it's possible to omit the authentication header in requests and start exploring the server's API without configuring any permissions first. Access control checks can be switched on independently of whichever mode (development or production) you run Zendro in — see [Access control (ACL)]({% link api/access-control.md %}) for how. For obvious reasons, it's highly recommended to only open remote access to a server with access control switched *on*.

[ > Access control (ACL) ]({% link api/access-control.md %})

### GraphQL API

Classical REST services suppose that all requests have a predefined form and are usually URL-driven. Each atomic resource is considered an *endpoint*, referred to by a fairly restricted query or mutation, for example:

```
<GET>
https://some.domain/books/1000/name
https://some.domain/books/1000/author
```

It's possible to parametrize such requests by inserting logic into them, but that's more of an *anti-pattern*, since each different service would end up with its own "programming" interface, and the style of these interfaces can strongly differ from one project to another. Basic CRUD operations are common across the web, though, and many groups have worked to parametrize and standardize the corresponding requests — the standard Zendro uses is GraphQL. It introduces a set of request body constructs aimed mainly at manipulating the response data in terms of CRUD operations. As an example, here's a GraphQL query that restricts the server response to only the fields "name" and "author" for the "book" model record with a given ID:

```
<POST>
{
  book(id: "1000") {
    name
    author
  }
}
```

This project automatically generates a set of GraphQL queries and mutations that, from our point of view, cover most of the needs of Zendro end users.

[ > GraphQL queries and mutations ]({% link api/graphql-reference.md %})

### Batch data exporting

It's possible to download all records for one model in batches. All records are exported as a CSV file, in which each field is quoted with `"` to reduce ambiguity between fields. There are two ways to download records — the Zendro command line interface or the Zendro Single Page App.

[ > Import and export data ]({% link guides/data-import-export.md %})

### SQL statements in the data model

One of the supported storage types — and the standard one for completely local databases — is SQL. When this storage type is used, all database access commands are ultimately transformed into SQL.

[ > SQL statements ]({% link api/sql-reference.md %})
