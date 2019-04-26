[ &larr; back](README.md)
<br/>
### ScienceDB API Documentation

Given a data model described using [this specifications and format](setup_data_scheme.md), the ScienceDb backend generator will implement default CRUD API that can be accessed through a well-known GraphQL query language or through an *export service*. To get more information about GraphQL queries and mutations you can read it's [official documentation](https://graphql.org/learn/queries/). When your server is up, the regular GraphQL service is accessible at `http://<back_srv>/graphql`. The service aimed for exporting massive joined database slices has another URL: `http://<back_srv>/export`. Both services accept POST requests with authentication information in it's header. In the case of GraphQL, request body should follow the GraphQL standard. Data export service accept custom request format that should be specified within JSON string.  

ScienceDB back-end server implementation follow with the GraphQL convention to refer a request that does not produce any data change as *query* and a request that modify data as *mutation*. The export service would never modify data, so all it's requests can be referred as queries.

ScienceDB API documentation consists of three parts:
<br/><br/>

_**Access Permissions**_

Back-end server can work in two modes: *development* and *production*, depending on the presence of `acl` argument in the command line that runs server. The development mode will cause all user permissions to be ignored. In this mode it is possible to omit authentication header in the requests and start to explore server's API without configuring any permissions. However it is highly recommended to open remote access to the server running in production mode for obvious reasons.

[ > ACL](api_acl.md)
<br/><br/>
_**GraphQL API**_
 
Classical REST service suppose all requests to have a predefined form, and usually is URL driven. Here each atomic resource is considered as an *endpoint* and can referred by a quite restricted request, for example:
```
GET /books/:id/comments
POST /books/:id/comments
```  
There exist a possibility to parametrize such requests inserting some logic into them, however it is more likely an *anti-pattern* because in this case each different service would have it's own "programming" interface, and style of these interfaces can strongly differ from one project to another. However, basic CRUD operations are common in the WWW world and many efforts was made by different groups to parametrize and standardize corresponding requests. The standard chosen in ScienceDB is a GraphQL. This standard introduce a set of request body constructs aimed mainly to manipulate the response data in terms of CRUD operations. As an example you can consider GraphQL query that restricts server response to only one field called "name" for the "book" record with a given ID:

```
{
  book(id: "1000") {
    name
  }
}
```

In this project it is automatically generated a set of GraphQL queries and mutations that, from our point of view, would cover most of the needs of ScienceDB end users.

[ > GraphQL Queries and Mutations](api_graphql.md)
<br/><br/>
_**Data exporting**_

Unfortunately current NodeJS GraphQL implementation used in ScienceDB does not support batch download in a fully optimal way because of lack of the non-blocking response data steaming. If it is required to join selected fields of the related data models and get it as a separate file stream it is possible to use the *export* service. 

This service helps when a full database cut is required for subsequent automated manipulations. For example: to create dynamically updated graphical reports or to append project specific table *views* (tables that unite more than one data model).

[ > Data Export](api_export.md)