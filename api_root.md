[ &larr; back](README.md)
<br/>
# Zendro API Documentation

Given a data scheme described using our [custom format](setup_data_scheme.md), the Zendro backend generator will implement default CRUD API that can be accessed through a well-known GraphQL query language or through an *export service*. To get more information about GraphQL queries and mutations you can read its [official documentation](https://graphql.org/learn/queries/). When back-end server is up, the regular GraphQL service is accessible at `http://<back_srv>/graphql`. The service for exporting massive joined database slices has another URL: `http://<back_srv>/export`. Both services accept POST requests with authentication information in its header. In the case of GraphQL, request body should follow the GraphQL standard. Data export service accepts its own request parameters.

Zendro back-end server implementation follows the GraphQL convention to refer to a request that does not cause any data change as *query* and a request that modifies data as *mutation*. The export service would never modify data, so all its requests can be referred as queries identically.


Zendro API documentation consists of three parts:
<br/><br/>

### Access Permissions

Back-end server can work in two modes: *development* and *production*, depending on the presence of `acl` argument in the command line that runs back-end server. The development mode will cause all user permissions to be ignored. In this mode it is possible to omit authentication header in the requests and start to explore server's API without configuring any permissions. However, for obvious reasons, it is highly recommended to open remote access to the server running in production mode (with `acl` switched-*on*).

[ > ACL](api_acl.md)
<br/><br/>
### GraphQL API

Classical REST services suppose that all its requests have a predefined form, and are usually URL driven. Here each atomic resource is considered as an *endpoint* and can be referred by a quite restricted query or mutation, for example:

```
<GET>
/books/:1000/name
/books/:1000/author
```  
It exists a possibility to parametrize such requests inserting some logic into them, however it is more likely an *anti-pattern* because in this case each different service would have its own "programming" interface, and the style of these interfaces can strongly differ from one project to another. However, basic CRUD operations are common in the WWW world and many efforts were made by different groups to parametrize and standardize corresponding requests. The standard chosen in Zendro is GraphQL. This standard introduces a set of request body constructs aimed mainly to manipulate the response data in terms of CRUD operations. As an example you can consider GraphQL query that restricts server response to only two fields called "name" and "author" for the "book" model record with a given ID:


```
<POST>
{
  book(id: "1000") {
    name
    author
  }
}
```

In this project it is automatically generated a set of GraphQL queries and mutations that, from our point of view, would cover most of the needs of Zendro end users.

[ > GraphQL Queries and Mutations](api_graphql.md)
<br/><br/>
### Batch Data Exporting

Unfortunately current NodeJS GraphQL implementation used in Zendro does not support batch download in a fully optimal way because of lack of the non-blocking response data streaming (see [this](https://github.com/graphql/graphql-js/issues/1537) discussion). When it is required to join selected fields of the related data models and get it as a separate file stream, using the *export* service would be the correct solution.


This service comes into play when a database cut is required for subsequent automated manipulations. For example: to create dynamically updated reports (diagrams, calculations) or to append project specific table *views* (tables that unite more than one data model).

Records would be exported as a CSV file, in which each field is quoted via `"` for reducing the ambiguity among fields.

[ > Data Export](api_export.md)

### SQL Statements in the Data model

One of the supported storage types (and the standard storage type for completely local databases) is SQL. If this storage type is used, all database access commands are ultimately transformed into SQL.

A list of basic GraphQL statements and their transformation into SQL can be seen here.

[ > SQL Statements](api_sql.md)
