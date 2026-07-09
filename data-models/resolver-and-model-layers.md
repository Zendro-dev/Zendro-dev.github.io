---
layout: default
title: Resolver and model layers
parent: Data models
nav_order: 3
permalink: /data-models/resolver-and-model-layers
---

# The resolver layer and the model layer
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

The Zendro server uses two layers below the GraphQL schema (which defines the functions the GraphQL server understands) to process data: the resolver layer and the model layer. This is because Zendro supports a growing number of storage types, and the more abstract layer (the resolver) is meant to be storage-type agnostic. There is one exception (distributed data models cannot support one resolver function — see [Pagination types](#pagination-types) below), but otherwise it holds true. So the resolver layer provides an interface that the GraphQL schema can use, and, on the other side, an interface that a new storage type's model has to fulfill.

This page walks through the code the [code generator]({% link data-models/json-specification.md %}) produces for a model and its associations, from the resolver methods down to the model methods that implement them.

## The code generator at work

The code generator receives the specification described in the [JSON specification]({% link data-models/json-specification.md %}) and generates resolvers and models from it. This section looks at the code generated for the different attributes given in the JSON specification.

The model contains a constant named `definition` holding the full JSON specification. The following placeholders are used below (references to the JSON file included):

* `<model>`: the respective main record type (`model`)
* `<models>`: the pluralized version of `<model>`
* `<NAME>`: the name of the association as given in the JSON file (the respective key in the `associations` object)
* `<assoc>`: the name of the associated record (`<NAME>` with capitalization and plural forms properly applied)
* `<assocID>`: the association ID (`associations -> <NAME> -> targetKey`)
* `<ModelIDAttribute>`: the name of the ID attribute (`internalId` if present, otherwise `id`)
* `<assocIDValue>`: the actual value of `<assocID>`
* `<CrossTable>`: the name of the SQL cross table (`associations -> <NAME> -> keysIn`)

The only storage type examined here is `sql` (`storageType` for the main record, `associations -> <NAME> -> targetStorageType` for the association).

### The model itself

First, the methods created for handling the model itself, without regard to associations.

The following methods are created in the resolver:

Signature | Use | Link to model function (optional)
-----|-----|-----
errorMessageForRecordsLimit(query) | Puts out an error message if the record limit is violated |
async function checkCountAndReduceRecordsLimit(search, context, query) | Checks the record limit — if kept, reduces it by the count of the entries, otherwise throws `Error` |
checkCountForOneAndReduceRecordsLimit(context) | Checks the record limit for a single entry — if kept, reduces it by the count of the entries, otherwise throws `Error` |
async function validForDeletion(id, context) | Checks if a record can be deleted (i.e. if it has no associations) |
`<models>`({search, order, pagination}, context) | Performs a search of the model entries with limit-offset based pagination after checking for authorization | SEARCH_LO_ROOT
`<models>`Connection({search, order, pagination}, context) | **Root Resolver:** Performs a search of the model entries with cursor based pagination after checking for authorization | SEARCH_CURSOR_ROOT
readOne`<model>`({`<ModelIDAttribute>`}, context) | **Root Resolver:** Returns a single record matching the given ID after checking for authorization | SINGLE_ROOT
count`<models>`: async function({search}, context) | **Root Resolver:** Returns the number of records matching the given search term after checking for authorization | COUNT_ROOT
vueTable`<model>`(_, context) | **Root Resolver:** Returns a table of records as needed for displaying a vuejs table after checking for authorization (**deprecated**) |
add`<model>`(input, context) | **Root Mutation:** Adds a record for the model after checking for authorization | ADDING_ROOT
bulkAdd`<model>`Csv(_, context) | **Root Mutation:** Loads a csv file containing records and adds them to the model after checking for authorization | ADDING_IN_BULK_ROOT
delete`<model>`({`<ModelIDAttribute>`}, context) | **Root Mutation:** Deletes the record given by the ID value after checking for authorization | DELETING_ROOT
update`<model>`: async function(input, context) | **Root Mutation:** Updates a record given by the input argument with new values and/or associations after checking for authorization | UPDATING_ROOT
csvTableTemplate`<model>`(_, context) | **Root Resolver:** Returns the table's template after checking for authorization | TEMPLATE

The following methods are created in the model:

Signature | Use | Link to resolver function (optional)
-----|-----|-----
static init(sequelize, DataTypes) | Initializes the model |
static async readById(id) | Returns a single record given by the ID | SINGLE_ROOT
static async countRecords(search) | Counts the records matching the search term | COUNT_ROOT
static readAll(search, order, pagination) | Returns the records matching the search term with limit-offset based pagination | SEARCH_LO_ROOT
static readAllCursor(search, order, pagination) | Returns the records matching the search term with cursor based pagination | SEARCH_CURSOR_ROOT
static addOne(input) | Adds a record | ADDING_ROOT
static deleteOne(id) | Deletes a record | DELETING_ROOT
static updateOne(input) | Updates a record | UPDATING_ROOT
bulkAddCsv(context) | Adds several records from a csv file | ADDING_IN_BULK_ROOT
csvTableTemplate() | Returns the template of a table | TEMPLATE
static idAttribute() | Returns the name of the ID attribute |
static idAttributeType() | Returns the type of the ID attribute |
getIdValue() | Returns the value of the ID attribute |
static get definition() | Getter which returns the `definition` constant |
static base64Decode(cursor) | Decodes a base64 representation of a given cursor into a UTF-8 string |
base64Encode() | Encodes the current cursor into a base64 string |
stripAssociations() | Returns only the attributes of the current record |
externalIdsArray() | Returns `definition.externalIds` if present, otherwise `[]` |
externalIdsObject() | Returns an object containing only the attributes with external IDs as keys, or an empty object |

### For all associations

In the resolver the following entries are created:

Signature | Use
-----|-----
associationsArgsDef | An object containing, for each association, the name of the "add" method as key and the name of the association as value.
`<model>`.prototype.handleAssociation = async function(input, context) | Method which handles the execution of all "add"/"remove" statements as promises.
async function countAllAssociatedRecords(id, context) | Counts all existing associations of a record, to make sure no records are associated to one before it is deleted.

In the model, a method `static associate(models)` is created, filled in for each association type as outlined below.

### Association type *x_to_many*

For this association, the following methods are created in the resolver:

Signature | Use | Link to resolver function (optional)
-----|-----|-----
`<model>.prototype.<assoc>Filter({search, order, pagination}, context)` | **Field Resolver:** Returns the associated records via a search with limit-offset based pagination by calling the root resolver of the associated records |
`<model>.prototype.<assoc>Connection({search, order, pagination}, context)` | **Field Resolver:** Returns the associated records via a search with cursor based pagination by calling the root resolver of the associated records |
`<model>`.prototype.countFiltered`<assoc>`({search}, context) | **Field Resolver:** Counts the associated records by calling the root resolver of the associated records
`<model>`.prototype.add_`<assoc>` = async function(input) | **Field Mutation:** Adds records in a loop, setting the associated ID in the *associated record* to the ID of the main record, by calling the associated record's model field resolver implementation adding method | ADDING_TO1
`<model>`.prototype.remove_`<assoc>` = async function(input) | **Field Mutation:** Removes records in a loop, removing the associated ID in the *associated record*, by calling the associated record's model field resolver implementation deleting method | REMOVING_TO1

In the model file, an entry is added to `associate(models)` in the form `<model>.hasMany(models.<assoc>, {as: <assoc>, foreignKey: <assocID>})`. The model methods called from the resolver live in the associated model (which holds the association key).

### Association type *x_to_one*

For this association, the following methods are created in the resolver:

Signature | Use | Link to resolver function (optional)
-----|-----|-----
`<model>`.prototype.`<assoc>` = async function({search}, context) | **Field Resolver:** Returns the associated record via a search for the given record ID, by calling the associated record's root resolver for searching with limit-offset based pagination | SEARCH_LO_ROOT
`<model>`.prototype.add_`<assoc>` = async function(input) | **Field Mutation:** Adds a record | ADDING_TO1
`<model>`.prototype.remove_`<assoc>` = async function(input) | **Field Mutation:** Removes a record | REMOVING_TO1

In the model, the following entries are created:

Signature | Use | Link to model function (optional)
-----|-----|-----
`<model>`.belongsTo(models.`<assoc>`, {as: `<assoc>`, foreignKey: `<assocID>`}) | In **associate(models):** creates a to-one association to the targeted model via Sequelize |
static async add_`<assocID>`(`<ModelIDAttribute>`, `<assocIDValue>`) | Adds an entry by setting the associated ID in the *main record* to the ID of the associated record | ADDING_TO1
static async remove_`<assocID>`(`<ModelIDAttribute>`, `<assocIDValue>`) | Removes the associated ID of an associated record in the *main record* | REMOVING_TO1

### Association type *many_to_many* through *sql_cross_table* implementation

Here, an additional record type is introduced for the SQL cross table, so there are 3 models and 3 resolvers to consider. Unlike the previous two cases, this one is fully symmetric — both records involved define this type of connection with the name of the cross table as `keysIn`. The source key for both types is the ID of the own record, and the target key is the ID of the other record. Because of this symmetry, only 4 files need to be considered.

In each record resolver, the following entries are created:

Signature | Use | Link to model function (optional)
-----|------|-----
`<model>`.prototype.`<assoc>`Filter({search, order, pagination}, context) | **Field Resolver:** Returns the associated records via a search with limit-offset based pagination after checking for authorization | SEARCH_LO_TMTSCT
`<model>`.prototype.`<assoc>`Connection({search, order, pagination}, context) | **Field Resolver:** Returns the associated records via a search with cursor based pagination after checking for authorization | SEARCH_CURSOR_TMTSCT
`<model>`.prototype.countFiltered`<assoc>`({search}, context) | **Field Resolver:** Counts the associated records after checking for authorization | COUNT_TMTSCT
`<model>`.prototype.add_`<assoc>` = async function(input) | **Field Mutation:** Adds an associated record | ADDING_TMTSCT
`<model>`.prototype.remove_`<assoc>` = async function(input) | **Field Mutation:** Removes an associated record | REMOVING_TMTSCT

In each record model, the following entries are created:

Signature | Use | Link to resolver function (optional)
-----|------|-----
`<model>`.belongsToMany(models.`<assoc>`, {as: `<assoc>`, foreignKey: `<assocID>`, through: `<CrossTable>`, onDelete: 'CASCADE'}) | In **associate(models):** creates a many-to-many association to the targeted model |
`<assoc>`FilterImpl({search, order, pagination}) | Implements the search with limit-offset based pagination | SEARCH_LO_TMTSCT
`<assoc>`ConnectionImpl({search, order, pagination}) | Implements the search with cursor based pagination | SEARCH_CURSOR_TMTSCT
countFiltered`<assoc>`Impl({search}) | Implements the counting | COUNT_TMTSCT
static async add_`<assocID>`(record, add`<assoc>`) | Adds a record | ADDING_TMTSCT
static async remove_`<assocID>`(record, remove`<assoc>`) | Removes a record | REMOVING_TMTSCT

#### Cross table resolver / model

Normal resolver / model files are created here, where the respective model type has no associations of its own, so these files only contain the methods from [the model itself](#the-model-itself).

## Authorization checks and record limits

Not every access to Zendro is permitted. In many cases (see above), the user must be authorized to perform a certain action; possible authorizations for a given table include `read`, `create`, `delete`, `update`. These authorizations are connected to roles that users can have, stored in the database — see [Authentication and authorization]({% link authentication/index.md %}).

Additionally, read actions are refused if they would access too many records. GraphQL is a very powerful data query and manipulation language that gives the user control over what to query from the server, but this also makes it possible (by accident or malice) to build a query so large the server cannot handle it. To prevent this, the server has a limit on the number of records a single query can access, and the user must provide pagination arguments for any `readMany` query.

## Pagination types

Zendro provides two different types of pagination: limit-offset based and cursor based. Limit-offset based pagination is the better known one, where the user provides an offset (how many entries to skip) and a limit (how many entries to display on one page).

Limit-offset based pagination is not possible for distributed data models, i.e. models whose records are spread over different servers — the client making the request doesn't know how the entries are distributed, so the offset for the different servers cannot be provided. Since this data model is especially useful for Big Data, requesting the entire table and paginating on the client side is not feasible.

Instead, a different approach is used: if the different servers are told which entry was the last one "above" the requested page (a cursor), they can deliver the content that follows it, and the client can assemble the received data in order. This type of pagination works for any kind of data, although it is more complex (the entries for the table are found under `edges[] -> node`).

See [Pagination argument]({% link api/graphql-reference.md %}#pagination-argument) for how to use this in a GraphQL query.

## Data loader

When reading a record by its ID, Zendro uses a [data loader](https://github.com/graphql/dataloader) by default to improve read performance. It bundles IDs to be fetched and requests them in one composite query.

Example of fetching multiple records within one request:

```
{
  n0: readOneAccession(accession_id: "a-instance1") {
    accession_id
    collectors_name
    location(search:null){
      locationId
    }
  }
  n1: readOneAccession(accession_id: "b-instance1") {
    accession_id
    collectors_name
    location(search:null){
      locationId
    }
  }
  n2: readOneAccession(accession_id: "c-instance1") {
    accession_id
    collectors_name
    location(search:null){
      locationId
    }
  }
}
```

In this example, querying the associated location for each `readOneAccession` query invokes the location root resolver to search for associated records. Using the data loader, all `readById` requests to the `accession` and `location` models are collected and optimized to fetch those IDs together — reducing the number of executed queries from six to two.
