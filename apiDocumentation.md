[ &larr; back](README.md)
<br/>
### ScienceDB API Documentation

Given a data model described using [this specifications and format](dataModels.md), the ScienceDb backend generator will implement default CRUD API that can be accessed through a well-known GraphQL query language or an *export service*. To get more information about GraphQL queries and mutations you can read it's [official documentation](https://graphql.org/learn/queries/). When your server is up, the regular GraphQL service is accessible at `http://<back_srv>/graphql`. The service supported aimed for exporting massive joined database slices has another URL: `http://<back_srv>/export`. Both services accept regular POST requests with authentication information in it's header. In the case of GraphQL, request body should follow the GraphQL standard. The data export module accepts custom request format that is to be passed as JSON string.  

ScienceDB back-end server implementation follow with the GraphQL convention to refer a request that does not produce any data change as *query* and a request that cause a data change as *mutation*. The export service would never modify any data, so all it's requests can be referred as queries.

ScienceDB API documentation consists of three parts:
<br/><br/>

_**Access Permissions**_

The back-end server can work in two modes: *development* and *private*, depending on the presence of `acl` argument in the command line that runs the server. The development mode will cause all user permissions to be ignored so that it is possible to omit authentication header in the requests and start to explore server's API without configuring any permissions. However it is highly recommended to open remote access to the server running in private mode for obvious reasons.

[ > ACL](projectCustomizing.md)
<br/><br/>
_**GraphQL API**_
 
Classical REST services suppose all requests to be of strictly predefined form, and usually is URL driven. Each atomic resource is considered as an *endpoint* and would accept a quite restricted requests, for example:
```
GET /books/:id/comments
POST /books/:id/comments
```  
It is obvious the possibility to parametrize such requests inserting some logic into them, however it is more likely an anti-pattern because in this case each different service would have it's own "programming" interface, and style of these interfaces can strongly differ from one project to another. However, the basic CRUD operations are so common in the WWW world that a lots of efforts was made by different groups to standardize request parameters. The standard chosen in ScienceDB is a GraphQL. This standard introduce a set of request body constructs to manage the response data in terms of CRUD operations that can vary strongly. As an example you can consider a GraphQL query that restricts a data fields in the server response for an element with a given ID:

```
{
  book(id: "1000") {
    name
  }
}
```

In this project it is automatically generated a set of GraphQL queries and mutations that, from our point of view, would cover the most of the needs of ScienceDB end users.

[ > GraphQL queries and mutations](GrapgQLAPI.md)
<br/><br/>
_**Data exporting**_

Unfortunately current NodeJS GraphQL implementation used in ScienceDB does not support batch download in a fully optimal way because of lack of the non-blocking response data steaming. If it is required to join selected fields of the related data models and get it as a separate file stream it is possible to use our additional *export* service. 

This service can be used when a full database cut is required for subsequent automated manipulations, for example to create dynamically updated graphical reports, or to append project specific table *views* (tables that unite more that on data model).

[ > Data export](DataExport.md)