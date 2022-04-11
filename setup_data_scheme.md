[&larr; back](setup_root.md)
<br/>

# Table of Contents
* TOC
{:toc}

# Data Models

For each one of the data sets that you want to include in the project you will need to describe the data model. This description should include its relations or associations with any other model. The description should be placed in a json file following the [json specs](#json-specs) for this purpose. You will need to store all these json files in a single folder. Another limitation is that each model should have a unique name independently of its type. From now on, in this document, we will assume that all json files for each one of your data models will be stored in the directory `/your-path/json-files`

## JSON Specs

Each json file describes one and only one model. However, one model can reference another model using the associations mechanism described below.

For a complete description of each model we need to specify the following fields in the json file:

Name | Type | Description
------- | ------- | --------------
*model* | String | Name of the model (it is recommended to use snake_case naming style to obtain nice names in the auto-generated GraphQL API). The string here can not contain spaces.
*model_name_in_storage* | String | The name of the model in the storage itself. E.g the table-name in relation dbs, the collection in mongodb, the node in neo4j, etc. By default Zendro uses the lowercase pluralized *model* property. 
*database* | String | Name of the database connection as a key defined in [`data_models_storage_config.json`](https://github.com/Zendro-dev/graphql-server/blob/master/config/data_models_storage_config.json). If this field is not defined, the database connection used will be `default-<storageType>`.
*storageType* | String | Type of storage where the model is stored. So far can be one of: <ul> <li>  __sql__ for local relational databases supported by [sequelize](#http://docs.sequelizejs.com/) such as PostgreSql/MySql etc. </li>  <li>__generic__ for any database that your project would connect remotely.</li>  <li>__zendro\_server__ for models stored in any other instance created with zendro-tools.</li><li>  __cassandra__ for local cassandra databases supported by datastax node [cassandra-driver](https://docs.datastax.com/en/developer/nodejs-driver/4.6/). Refer to [cassandra storageType documentation](cassandra_storageType.md) for cassandra specific restrictions.</li></ul>
*url* | String | This field is only mandatory for __zendro\_server__ stored models. Indicates the url where the zendro server storing the model is runnning.
*attributes* | Object |  The key of each entry is the name of the attribute and there are two options for the value . It can be either a string indicating the type of the attribute or an object where the user indicates the type of the attribute(in the _type_ field) together with an attribute's description (in the _description_ field). See the [table](#supported-data-types) below for allowed types. Example of option one: ```{ "attribute1" : "String", "attribute2: "Int" }``` Example of option two: ``` { "attribute1" : {"type" :"String", "description": "Some description"}, "attribute2: "Int ```
*associations* | Object | The key of each entry is the name of the association and the value should be an object describing the corresponding association. See [Associations Spec](#associations-spec) section below for details.
*indices* | [String] |  Names of attributes for generating corresponding indices.
*internalId* | String | This string corresponds to the name of the attribute that uniquely identifies a record. If this field is not specified, an _id_, default attribute, will be added.

## Supported Data Types

The following types are allowed for the attributes field.

 Type |
------- |
String |
Int |
Float |
Boolean |
Date |
Time |
DateTime |

For more info about `Date`, `Time`, and `DateTime` types, please see this [info](https://github.com/excitement-engineer/graphql-iso-date/blob/HEAD/rfc3339.txt).

Example:

* Date: A date string, such as `2007-12-03`.
* Time: A time string at UTC, such as `10:15:30Z`.
* DateTime: A date-time string at UTC, such as `2007-12-03T10:15:30Z`

## Associations Spec

We will consider four types of associations according to the relation between associated records of the two models:
1. `one_to_one`
2. `many_to_one`
3. `one_to_many`
4. `many_to_many`

For all types of association, the necessary arguments would be:

name | Type | Description
------- | ------- | --------------
*type* | String | Type of association, either `one_to_one`, `one_to_many`, `many_to_one`, or `many_to_many`.
*target* | String | Name of model to which the current model will be associated with.
*implementation* | String | implementation type of the association. Can be one of `foreignkey`, `generic` or `sql_cross_table` (only for `many_to_many`)`
*reverseAssociation* | String | The name of the reverse association from the other model. This field is only mandatory for building the [single-page-app](https://github.com/Zendro-dev/single-page-app), *not* for generating the the graphql-server code via this repository.
*targetKey* | String | A unique identifier of the association stored in any of the two models involved in the association.
*keysIn* | String | Name of the model where the targetKey is stored.
*targetStorageType* | String | Type of storage where the target model is stored.
*label* | String | Name of the column in the target model to be used as a display name in the GUI. This will be useful as a preview of the association.
*sublabel* | String | Optional name of the column in the target model to be used as a sub-label in the GUI. Also used for a more explicit preview of the association.
*useDataLoader* | Boolean | If it is set to `true`, server could fetch multiple records within one query for `readOne<model>` API. 

**Note**: The `keysIn` argument points to the model that stores the information about the foreignKey(s). That can be either a single key, a foreignkey array or a cross-model.

### Foreign keys
It's important to notice that when a model involves a foreign key for the association, this key should be explicitly written into the attributes field of the given local model. Although, foreign keys will be available for the user only as readable attributes, for editing this attributes we offer the possibility as part of the API, please see [this](api_graphql.md#extra-mutation-fields-to-update-or-create-associations) section for more info.

Example:

```json
{
  "model" : "book",
  "storageType" : "sql",
  "attributes" : {
    "title" : {"type":"String", "description": "The book's title"},
    "publisher_id": "Int"
  },
  "associations":{
      "publisher" : {
        "type" : "many_to_one", // association type
        "implementation": "foreignkey",
        "reverseAssociation": "books",
        "target" : "publisher", // Model's name is `publisher`
        "targetKey" : "publisher_id", // Local alias for this association
        "keysIn": "book", // FK to publisher will be stored in the Book model
        "targetStorageType" : "generic", //  It's a remote database
        "label" : "name" // Show in GUI the name of the publisher taken from external DB
        }
  }
}
```

#### Many to many association through foreign key arrays

To store many-to-many associations via foreignkeys Zendro offers to store the foreign keys in arrays. In this case the model will have an array attribute which will store ids from the associated records. Please note that both sides of the association will store an array for this functionality, these two attributes will be described in the association as `sourceKey` and `targetKey`.
Also, for indicating that the association is a many-to-many association via arrays as foreign key, we need to specify in the association info the `implementation` field as `foreignkey`.

name | Type | Description
------- | ------- | --------------
*sourceKey* | String | Attribute of type array, belonging to source model, which stores the associated ids of target model.
*targetKey* | String | Attribute of type array, belonging to target model, which stores the associated ids of source model.
*implementation* | 'foreignkey' | Set the `implementation` field to 'foreignkey'

Example:
Assume we have an association between two models: `book` and `author`. The models description should be as below:

```json
{
    "model" : "author",
    "storageType" : "sql",
    "database": "default-sql",
    "attributes" : {
        "id": "String",
        "name": "String",
        "lastname": "String",
        "email": "String",
        "book_ids": "[ String ]"
    },

    "associations":{
      "books":{
        "type": "many_to_many",
        "implementation": "foreignkey",
        "reverseAssociation": "authors",
        "target": "book",
        "targetKey": "author_ids",
        "sourceKey": "book_ids",
        "keysIn": "author",
        "targetStorageType": "sql"
      }
    },

    "internalId": "id"
  }
```

```json
{
    "model" : "book",
    "storageType" : "sql",
    "database": "default-sql",
    "attributes" : {
        "id": "String",
        "title": "String",
        "genre": "String",
        "ISBN": "String",
        "author_ids": "[ String]"
    },

    "associations":{
      "authors":{
        "type": "many_to_many",
        "implementation": "foreignkey",
        "reverseAssociation": "books",
        "target": "author",
        "targetKey": "book_ids",
        "sourceKey": "author_ids",
        "keysIn": "book",
        "targetStorageType": "sql"
      }
    },

    "internalId": "id"
  }
```
### Many to many association through table
When the association is of type *many_to_many* and it referes to a more particular type of association *many_to_many*, stored in a cross table, it's necessary to describe two extra arguments . In this case the type is referred to as *to_many_through_sql_cross_table* and it's only available for `sql` stored models. The additional arguments are:

name | Type | Description
------- | ------- | --------------
*sourceKey* | String | Key to identify the source id
*targetKey* | String | Key to identify the target id
*implementation* | 'sql_cross_table' | Set the `implementation` field to 'sql_cross_table'

```json
//User model
{
  "model" : "User",
  "storageType" : "SQL",
  "attributes" : {
    "email" : "String",
    "password" : "String"
  },
  "associations" :{
    "roles" : {
      "type" : "many_to_many",
      "implementation": "sql_cross_table",
      "reverseAssociation": "users",
      "target" : "Role",
      "targetKey" : "role_Id",
      "sourceKey" : "user_Id",
      "keysIn" : "role_to_user",
      "targetStorageType" : "sql",
      "label": "name"
    }
  }

}
```

```json
//Role model
{
  "model" : "Role",
  "storageType" : "SQL",
  "attributes" : {
    "name" : "String",
    "description" : "String"
  },
  "associations" : {
    "users" : {
      "type" : "many_to_many",
      "implementation": "sql_cross_table",
      "reverseAssociation": "roles",
      "target" : "User",
      "targetKey" : "user_Id",
      "sourceKey" : "role_Id",
      "keysIn" : "role_to_user",
      "targetStorageType" : "sql",
      "label": "email"
    }
  }
}
```

```json
//role_to_user model
{
  "model" : "role_to_user",
  "storageType" : "SQL",
  "attributes" : {
    "user_Id" : "Int",
    "role_Id" : "Int"
  }
}

```


### generic associations
To generate a generic association the `generic` implementation type can be used. This will genereate code stubs in the models for the user specific implementation of resolving the management of the association.

### Differences between backend and Frontend (GUI)

The same data model description files (.json) can be used for generating both the [BACKEND](setup_backend.md) and [FRONTEND OR GUI](setup_gui.md). Fields such as  *`label`* and *`sublabel`* in the model specification that are only needed for GUI generator are ignored by the backend generator.

The field `reverseAssociation` is only mandatory for generating the queries used in the single-page-application to communicate with the graphql-server. Generating the graphql-server code without setting this field will give an appropriate warning.

### About the association types

In Zendro, there are four association types, namely __* : 1__ (many-to-one), __1 : *__ (one-to-many), __* : *__ (many-to-many) and __1:1__ (one-to-one).

As usually, in any association type the foreign-key shall be placed inside one of the two associated tables. However both table models need to be notified about that link. This association information shall be placed in each model definition JSON files. Below we consider the models for two tables A and B, that are defined in the files A.json and B.json correspondingly.

Within the body of any model JSON file, the model that is described in this file is considered as a source model, and any other model is considered as a target. Therefore, within the JSON source it is required to specify how the current model is associated with all its targets.

Let's use some examples to explain these four types of association: 

1. __Many-to-One__
    This case happens when more than one element of the table A can reference the same element of table B. In this case, the foreign-key has to be created in the table A.

    Example: Table A contains a list of employees and each of them can work in a single department only (catalog B).

    Data Model Definition for table A:
    ```
    {
      "model": "employee",
      "storageType": "mongodb",
      "attributes": {
          "employee_id": "String",
          "name": "String",
          "department_id": "String"
      },
      "associations": {
          "department": {
              "type": "many_to_one",
              "implementation": "foreignkeys",
              "reverseAssociation": "employees",
              "target": "department",
              "targetKey": "department_id",
              "keysIn": "employee",
              "targetStorageType": "mongodb"
          }
      },
      "internalId": "employee_id",
      "id": {
          "name": "employee_id",
          "type": "String"
      },
      "useDataLoader": true
    }
    ```
  
    (table A keeps a not unique foreignkey_B)

2. __One-to-Many__ This association type is the reverse type of Many-to-One association type. Hence, we can use the same example to explain. And the below data model definition is for table B:

    Data Model Definition for table B:

    ```
    {
        "model": "department",
        "storageType": "mongodb",
        "attributes": {
            "department_id": "String",
            "department_name": "String"
        },
        "associations": {
            "employees": {
                "type": "one_to_many",
                "implementation": "foreignkeys",
                "reverseAssociation": "department",
                "target": "employee",
                "targetKey": "department_id",
                "keysIn": "employee",
                "targetStorageType": "mongodb"
            }
        },
        "internalId": "department_id",
        "id": {
            "name": "department_id",
            "type": "String"
        },
        "useDataLoader": false
    }
    ```

3. __Many-to-Many__

    Based on the previous example, this type of relationship underlines the fact, that an employee can belong to more than one department as well as the department can incorporate more than one employee. In this case, a new relation table has to be created that would hold the foreign-key pairs that define employee-to-department associations.

    Data Model Definition for table A:
    ```
    {
      "model": "employee",
      "storageType": "mongodb",
      "attributes": {
          "employee_id": "String",
          "name": "String",
          "department_ids": "[String]"
      },
      "associations": {
          "departments": {
              "type": "many_to_many",
              "implementation": "foreignkeys",
              "reverseAssociation": "employees",
              "target": "department",
              "targetKey": "employee_ids",
              "sourceKey": "department_ids",
              "keysIn": "employee",
              "targetStorageType": "mongodb"
          }
      },
      "internalId": "employee_id",
      "id": {
          "name": "employee_id",
          "type": "String"
      },
      "useDataLoader": true
    }
    ```
    Data Model Definition for table B:

    ```
    {
        "model": "department",
        "storageType": "mongodb",
        "attributes": {
            "department_id": "String",
            "department_name": "String",
            "employee_ids": "[String]"
        },
        "associations": {
            "employees": {
                "type": "many_to_many",
                "implementation": "foreignkeys",
                "reverseAssociation": "departments",
                "target": "employee",
                "targetKey": "department_ids",
                "sourceKey": "employee_ids",
                "keysIn": "department",
                "targetStorageType": "mongodb"
            }
        },
        "internalId": "department_id",
        "id": {
            "name": "department_id",
            "type": "String"
        },
        "useDataLoader": false
    }
    ```

    (AB relation table with foreignkey_A and foreignkey_B is created automatically)

4. __One-to-One__

    This relation type contains restriction: an absence of the possibility to relate more than one pair of the elements. In this case it is a subject of intuition which table shall hold the foreign-key. However, the foreign-keys can't repeat and shall be unique.

    For example, the A table is a catalog of well studied species that have strict registration number, whereas table B contains all discovered species, some of which are not cataloged yet.

    Data Model Definition for table B:
    ```
    {
        "model": "discovered_specie",
        "storageType": "mongodb",
        "attributes": {
          "discovered_specie_id": "String",
          "studied_specie_id": "String"
        },
        "associations": {
          "unique_studied_specie": {
            "type": "one_to_one",
            "implementation": "foreignkeys",
            "reverseAssociation": "unique_discovered_specie",
            "target": "studied_specie",
            "targetKey": "studied_specie_id",
            "keysIn": "discovered_specie",
            "targetStorageType": "mongodb"
          }
        },
        "internalId": "discovered_specie_id",
        "id": {
            "name": "discovered_specie_id",
            "type": "String"
        }
    }
    ```
    (table B keeps a unique foreignkey_A)

## The Resolver Layer and the Model Layer

The Zendro server uses two different layers below the GraphQL schema (which defines the functions that the GraphQL server understands) to process data, the resolver layer and the model layer. The main reason for this is that Zendro supports a growing number of storage types, and the more abstract layer (the resolver) is supposed to be storage type agnostic. There is one exception to this (in the case of distributed data models, one resolver function cannot be supported, see [below](#pagination-types)), but otherwise, it holds true. So the resolver layer provides an interface that the GraphQL schema can use, and (on the other side) an interface that a new storage type model has to fulfill.

## The code generator at work

The code generator receives the specification as described [above](#json-specs) and generates resolvers and models from it. In this section we take a look at the code that is generated from the different attributes that can be given by the JSON specification.

The model contains a constant named `definition` that contains the full JSON specification. The following placeholders will be used from now on (references to the JSON file included):

* `<model>`: The respective main record type (`model`)
* `<models>`: The pluralized version of `<model>`
* `<NAME>`: The name of the association as given in the JSON file (respective key in the object `associations`)
* `<assoc>`: The name of the associated record (`<NAME>` with capitalization and plural forms properly applied)
* `<assocID>`: The association ID (`associations -> <NAME> -> targetKey`)
* `<ModelIDAttribute>`: The name of the ID attribute (`internalId` if present, otherwise "`id`")
* `<assocIDValue>`: The actual value of `<assocID>`
* `<CrossTable>`: The name of the SQL cross table (`associations -> <NAME> -> keysIn`)

The only storage type that is examined here is `sql` (`storageType` for the main record, `associations -> <NAME> -> targetStorageType` for the association).

### The model itself

First we observe the methods that are created for handling the model itself with no regard yet to the associations.

The following methods are created in the resolver:

Signature | Use | Link to model function (optional)
-----|-----|-----
errorMessageForRecordsLimit(query) | Puts out an error message if record limit is violated |
async function checkCountAndReduceRecordsLimit(search, context, query) | Checks the record limit - if kept, reduce it by the count of the entries, otherwise throw `Error` |
checkCountForOneAndReduceRecordsLimit(context) | Checks the record limit for a single entry - if kept, reduce it by the count of the entries, otherwise throw `Error` |
async function validForDeletion(id, context) | Checks if a record can be deleted (i.e. if this record has no associations) |
`<models>`({search, order, pagination}, context) | Performs a search of the model entries with limit-offset based pagination after checking for authorization | SEARCH_LO_ROOT
`<models>`Connection({search, order, pagination}, context) | __Root Resolver:__  Performs a search of the model entries with cursor based pagination after checking for authorization | SEARCH_CURSOR_ROOT
readOne`<model>`({`<ModelIDAttribute>`}, context) | __Root Resolver:__  Returns a single record which matches the given ID after checking for authorization | SINGLE_ROOT
count`<models>`: async function({search}, context) | __Root Resolver:__  Returns the number of records which match the given search term after checking for authorization | COUNT_ROOT
vueTable`<model>`(_, context) | __Root Resolver:__  Returns a table of records as needed for displaying a vuejs table after checking for authorization (__deprecated__) |
add`<model>`(input, context) | __Root Mutation:__  Adds a record for the model after checking for authorization | ADDING_ROOT
bulkAdd`<model>`Csv(_, context) | __Root Mutation:__  Loads a csv file containing records and adds them to the model after checking for authorization | ADDING_IN_BULK_ROOT
delete`<model>`({`<ModelIDAttribute>`}, context) | __Root Mutation:__  Deletes the record given by the ID value after checking for authorization | DELETING_ROOT
update`<model>`: async function(input, context) | __Root Mutation:__  Updates a record which is given by the input argument with new values and/or associations as given by the input argument after checking for authorization | UPDATING_ROOT
csvTableTemplate`<model>`(_, context) |  __Root Resolver:__  Returns the table's template after checking for authorization | TEMPLATE

The following methods are created in the model:

Signature | Use | Link to resolver function (optional)
-----|-----|-----
static init(sequelize, DataTypes) | Initializes the model |
static async readById(id) | Returns a single record given by the ID | SINGLE_ROOT
static async countRecords(search) | Counts the records given by the search term | COUNT_ROOT
static readAll(search, order, pagination) | Returns the records given by the search term with limit-offset based pagination | SEARCH_LO_ROOT
static readAllCursor(search, order, pagination) | Returns the records given by the search term with cursor based pagination | SEARCH_CURSOR_ROOT
static addOne(input) | Adds a record | ADDING_ROOT
static deleteOne(id) | Deletes a record | DELETING_ROOT
static updateOne(input) | Updates a record | UPDATING_ROOT
bulkAddCsv(context) | Adds several records from a csv file | ADDING_IN_BULK_ROOT
csvTableTemplate() | Returns the template of a table | TEMPLATE
static idAttribute() | Returns the name of the ID attribute |
static idAttributeType() | Returns the type of the ID attribute |
getIdValue() | Returns the value of the ID attribute |
static get definition() | Getter which returns the `definition` constant |
static base64Decode(cursor) | Decodes a base 64 representation of a given cursor into a UTF-8 string |
base64Encode() | Encodes the current cursor into a base 64 string |
stripAssociations() | Returns only the attributes of the current record |
externalIdsArray() | Returns `definition.externalIds` if present, otherwise `[]` |
externalIdsObject() | Returns an object containing only the attributes with external IDs as keys, or an empty object |

### For all associations

In the resolver the following entries are created:

Signature | Use
-----|-----
associationsArgsDef | Constant is created which is an object, containing for each association the name of the "add" method as key and the name of the association as value.
`<model>`.prototype.handleAssociation = async function(input, context) | Method which handles the execution of all "add"/"remove" statements as promises.
async function countAllAssociatedRecords(id, context) | *Method which counts all existing associations of a record, so that it can be made sure that no records are associated to one if this record is to be deleted.*

In the model a method `static associate(models)` is created, which is filled for each association type as outlined below.

### Association type *x_to_many*

For this association the following methods are created in the resolver:

Signature | Use | Link to resolver function (optional)
-----|-----|-----
`<model>.prototype.<assoc>Filter({search, order, pagination}, context)` | __Field Resolver:__  Returns the associated records via a search with limit-offset based pagination by calling the root resolver of the respective associated records |
`<model>.prototype.<assoc>Connection({search, order, pagination}, context)` | __Field Resolver:__ Returns the associated records via a search with cursor based pagination by calling the root resolver of the respective associated records |
`<model>`.prototype.countFiltered`<assoc>`({search}, context) | __Field Resolver:__ Counts the associated records by calling the root resolver of the associated records
`<model>`.prototype.add_`<assoc>` = async function(input) | __Field Mutation:__ Adds records in a loop that sets the associated ID in the *associated record* to the ID of the main record by calling the model field resolver implementation adding method of the associated record | ADDING_TO1
`<model>`.prototype.remove_`<assoc>` = async function(input) | __Field Mutation:__ Removes records in a loop that removes the associated ID in the *associated record* by calling the model field resolver implementation deleting method of the associated record | REMOVING_TO1

In the model file, an entry is added to the method `associate(models)` in  the form `<model>.hasMany(models.<assoc>, {as: <assoc>, foreignKey: <assocID>})`. The model methods that are called from the resolver are in the associated model (which holds the association key).

### Association type *x_to_one*

For this association the following methods are created in the resolver:

Signature | Use | Link to resolver function (optional)
-----|-----|-----
`<model>`.prototype.`<assoc>` = async function({search}, context) | __Field Resolver:__ Returns the associated record via a search for the given record ID by calling the root resolver of the associated record for searching with limit-offset based pagination | SEARCH_LO_ROOT
`<model>`.prototype.add_`<assoc>` = async function(input) | __Field Mutation:__ Adds a record | ADDING_TO1
`<model>`.prototype.remove_`<assoc>` = async function(input) | __Field Mutation:__ Removes a record | REMOVING_TO1


In the model the following entries are created:

Signature | Use | Link to model function (optional)
-----|-----|-----
`<model>`.belongsTo(models.`<assoc>`, {as: `<assoc>`, foreignKey: `<assocID>`}) | In __associate(models):__ Creates a to-one-association to the targeted model via Sequelize |
static async add_`<assocID>`(`<ModelIDAttribute>`, `<assocIDValue>`) | Adds an entry by setting the associated ID in the *main record* to the ID of the associated record | ADDING_TO1
static async remove_`<assocID>`(`<ModelIDAttribute>`, `<assocIDValue>`) | Removes the associated ID of an associated record in the *main record* | REMOVING_TO1

### Association type *many_to_many* through *sql_cross_table* implementation

In this case an additional record type is introduced that contains the SQL cross table, so there are 3 models and 3 resolvers to consider. Unlike the former two cases, this one is fully symmetric. *Both* records involved define this type of connection with the name of the cross table as `keysIn`. The source key for both types is the ID of the own record, and the target key is the ID of the other record. Because of the symmetry, only 4 files must be considered.

In each record resolver, the following entries are created:

Signature | Use | Link to model function (optional)
-----|------|-----
`<model>`.prototype.`<assoc>`Filter({search, order, pagination}, context) | __Field Resolver:__ Returns the associated records via a search with limit-offset based pagination after checking for authorization | SEARCH_LO_TMTSCT
`<model>`.prototype.`<assoc>`Connection({search, order, pagination}, context) | __Field Resolver:__ Returns the associated records via a search with cursor based pagination after checking for authorization | SEARCH_CURSOR_TMTSCT
`<model>`.prototype.countFiltered`<assoc>`({search}, context) | __Field Resolver:__ Counts the associated records after checking for authorization | COUNT_TMTSCT
`<model>`.prototype.add_`<assoc>` = async function(input) | __Field Mutation:__  Adds an associated record | ADDING_TMTSCT
`<model>`.prototype.remove_`<assoc>` = async function(input) | __Field Mutation:__  Removes an associated record | REMOVING_TMTSCT

In each record model, the following entries are created:

Signature | Use | Link to resolver function (optional)
-----|------|-----
`<model>`.belongsToMany(models.`<assoc>`, {as: `<assoc>`, foreignKey: `<assocID>`, through: `<CrossTable>`, onDelete: 'CASCADE'})`| In __associate(models):__ Creates a many_to_many association to the targeted model |
`<assoc>`FilterImpl({search, order, pagination}) | Implements the search with limit-offset based pagination | SEARCH_LO_TMTSCT
`<assoc>`ConnectionImpl({search, order, pagination}) | Implements the search with cursor based pagination | SEARCH_CURSOR_TMTSCT
countFiltered`<assoc>`Impl({search}) | Implements the counting | COUNT_TMTSCT
static async add_`<assocID>`(record, add`<assoc>`) | Adds a record | ADDING_TMTSCT
static async remove_`<assocID>`(record, remove`<assoc>`) | Removes a record | REMOVING_TMTSCT

#### Cross Table Resolver / Model

Normal resolver / model files are created, where the respective model type has no associations of its own so that these files contain only the methods from [the model itself](#the-model-itself).

## Authorization-checks and record limits

Not every access to Zendro is permitted. In many cases (see above), the user must be authorized to perform a certain action. Possible authorizations for a given table include `read`, `create`, `delete`, `update`. These authorizations with respect to tables are connected to roles that users can have and are stored within the database.

Additionally, reading actions are refused if they access too many records. GraphQL is a very powerful data query and manipulation language that gives the user the control about what to query the server, but this makes it possible (by accident or malice) to make such a large query that the server cannot handle it. To  prevent this, the server has a set limit of records that can be accessed by a single query and the user is required to provide pagination arguments in case of a _readMany_ query.

## Pagination types

As seen above, Zendro provides two different types of pagination: Limit-offset based and cursor based. Limit-offset based pagination is the better known one, where the user provides an offset (how many entries to skip) and a limit (how many entries to display on one page).

Limit-offset based pagination is not possible for distributed data models, i.e. models where records of a single table are spread over different servers. This is because the client (who makes the request) does not know how the entries are distributed, so the offset for the different servers cannot be provided. Since this data model is especially useful for Big Data, it is not feasible to request the entire table and paginate the data on the client side.

Instead, a different model has to be used. If the different servers are told which was the last entry "above" the requested page (a cursor), they can deliver the content that would follow up to it and the client can now get the received data in order. This type of pagination works for any kind of data, although it is more complex (the entries for the table are found under `edges[] -> node`).

## Custom Validator Function for AJV
It is possible to add custom asynchronous validation functions with keyword `asyncValidatorFunction`.

### A Running Example
If there is a model called `example`, the attribute `id` in the model would be validated. Only if its value is `"1"`, it passes the validation. 

In specifically, it should be implemented by two steps:
1. find a file called `example.js` in folder `validations`.
2. add keyword `asyncValidatorFunction` and corresponding asynchronous validation function for attribute `id` in `validatorSchema`.

And the example code is as follows:
```
example.prototype.validatorSchema = {
  "$async": true,
  "properties": {
    "id": {
      "asyncValidatorFunction": async function(data) {
        if (data === "1") {
          return true
        } else {
          return new Promise(function(resolve, reject) {
            return reject(new Ajv.ValidationError([{
              keyword: 'asyncValidatorFunction',
              message: `${data} is not 1`
            }]))
          })
        }
      }
    }
  }
}
```
## Data Loader
When reading a record by its id, by default Zendro uses a [data loader](https://github.com/graphql/dataloader) to improve read performance. It does so by bundling IDs to be fetched and request those in one composite query.

Here is an example for fetching multiple records within one request:

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
In the above example querying the associated location for each `readOneAccession` query invokes the location root resolver to search for associated records. By using the data loader we can collect all `readById` requests to the `accession` and the `location` model and optimize the queries to fetch those Ids together. This reduces the amount of executed queries from six to two.
