[&larr; back](setup_root.md)
<br/>

# Data Models

For each one of the projects that you want to include in the project you will need to describe the model as well as its possible relations with any other model. The description should be placed in a json file following the [json specs](#json-specs) for this purpose. You will need to store all these json files in a single folder. Another limitation is that each model should have a unique name independently of it's type. From now on, in this document, we will assume that all json files for each one of your data models will be stored in the directory `/your-path/json-files`

## JSON Specs

Each json file describes one and only one model. However, one model can reference to another model using association mechanism described above. Withing `associations` block it is required to specify model names from another json files.

For each model we need to specify the following fields in the json file:

Name | Type | Description
------- | ------- | --------------
*model* | String | Name of the model (it is recommended to use snake_case naming style to obtain nice names in the auto-generated GraphQL API).
*storageType* | String | Type of storage where the model is stored. So far can be one of __sql__(for local relational databases supported by [sequelize](#http://docs.sequelizejs.com/) such as PostgreSql/MySql etc. ) or __Webservice__ for any database that your project would remotely connect to or __cenz\_server__ for models stored in any other instance created with cenz-tools.
*url* | String | This field is only mandatory for __cenz\_server__ stored models. Indicates the url where the cenz server storing the model is runnning.
*attributes* | Object |  The key of each entry is the name of the attribute and theres two options for the value . Either can be a string indicating the type of the attribute or an object where the user can indicates the type of the attribute(in the _type_ field) but also can indicates an attribute's description (in the _description_ field). See the [table](#supported-data-types) below for allowed types. Example of option one: ```{ "attribute1" : "String", "attribute2: "Int" }``` Example of option two: ``` { "attribute1" : {"type" :"String", "description": "Some description"}, "attribute2: "Int ```
*associations* | Object | The key of each entry is the name of the association and the value should be an object describing corresponding association. See [Associations Spec](#associations-spec) section below for details.

### Supported Data Types

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

### Associations Spec

We will consider two types of associations accordingly to the number of records
that can posibly be associated:

1. to_one
2. to_many

For both type of association, the necessary arguments would be:

name | Type | Description
------- | ------- | --------------
*type* | String | Type of association (like belongsTo, etc.)
*target* | String | Name of model to which the current model will be associated with.
*targetKey* | String | A unique identifier of the association for the case where there appear more than one association with the same model.
*keyIn* | String | Name of the model where the targetKey is stored.
*targetStorageType* | String | Type of storage where the target model is stored. So far can be one of __sql__ or __Webservice__.
*label* | String | Name of the column in the target model to be used as a display name in the GUI.
*sublabel* | String | Optional name of the column in the target model to be used as a sub-label in the GUI.

When the association is of type *to_many* and it referes to a more particular type of association *many_to_many*  it's necessary to describe two extra arguments given that the association is made with a cross table. In this case the type is referred to as *to_many_through_sql_cross_table*. The additional arguments are:

name | Type | Description
------- | ------- | --------------
*sourceKey* | String | Key to identify the source id
*keysIn* | String | Name of the cross table

#### NOTE - Many-to-many

Be aware that in the case of this type of association the user is required to describe the cross table used in the field _keysIn_ as a model in its own. For example, if we have a model `User` and a model `Role` and they are associated in a _manytomany_ way, then we also need to describe the `role_to_user` model:

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
      "type" : "to_many",
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
      "type" : "to_many",
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

#### Foreign keys

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
        "type" : "to_one", // association type
        "target" : "publisher", // Model's name is `publisher`
        "targetKey" : "publisher_id", // Local alias for this association
        "keyIn": "book", // FK to publisher will be stored in the Book model
        "targetStorageType" : Webservice", //  It's a remote database
        "label" : "name" // Show in GUI the name of the publisher taken from external DB
        }
  }
}
```

#### Differences between backend and Frontend (GUI)

The same data model description files (.json) can be used for generating both, the [BACKEND](setup_backend.md) and [FRONTEND OR GUI](setup_gui.md). Fields such as  *`label`* and *`sublabel`* in the model specification that are only needed for GUI generator are ignored by the backend generator.

#### About the associations type

In ScienceDB, association relationships are based on the [sequelize ORM terminology](http://docs.sequelizejs.com/manual/tutorial/associations.html). As usually, in any association type the foreign-key shall be placed inside one of the two associated tables. However here both table models need to be notified about that link. This association information shall be placed in each model definition JSON files. Below we consider the models for two tables A and B, that are defined in the files A.json and B.json correspondingly.

Within the body of any model JSON file, the model that is described in this file is considered as a source model, and any other model is considered as target. Therefore, within the JSON source it is required to specify how current model is associated with all it's targets.

To refer association link within source model JSON file, the next keywords can be used:

* belongsTo
* hasOne
* hasMany
* belongsToMany

The meaning of these keywords and their valid combinations are discussed from the point of view of the standard association relationships: __* : 1__ (many-to-one), __* : *__ (many-to-many) and __1:1__ (one-to-one).

1. __Many-to-One__
    This case happens when more than one element of the table A can reference the same element of table B. In this case, the foreign-key has to be created in the table A.
  
    Example: Table A contains a list of employees and each of them can work in a single department only (catalog B).
  
    A.json: belongsTo B;
    B.json: hasMany A;
    (table A keeps a not unique foreignkey_B)
  
1. __Many-to-Many__
  
    Based on the previous example, this type of relationship underlines the fact, that an employee can belong to more than one department as well as the department can incorporate more than one employee. In this case, a new relation table has to be created that would hold the foreign-key pairs that define employee-to-department associations.
  
    Example:
    A.json belongsToMany B;
    B.json belongsToMany A;
    (AB relation table with foreignkey_A and foreignkey_B is created automatically)
  
1. __One-to-One__
  
    This relation type contains restriction: an absence of the possibility to relate more than one pair of the elements. In this case it is a subject of intuition which table shell hold the foreign-key. However, the foreign-keys can't repeat and shall be unique.
  
    For example, the A table is a catalog of well studied species that have strict registration number, whereas table B contains all discovered species, some of which are not cataloged yet.
  
    A.json: hasOne B;
    B.json: belongsTo A;
    (table B keeps a unique foreignkey_A)

#### The code generator at work

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

##### The model itself

First we observe the methods that are created for handling the model itself with no regard yet to the associations.

The following methods are created in the resolver:

* The methods `errorMessageForRecordsLimit(query)`, `async function checkCountAndReduceRecordsLimit(search, context, query)` and `checkCountForOneAndReduceRecordsLimit(context)`, which handle the checking of the records limit (see below)
* The method `async function validForDeletion(id, context)`, which checks if a given record can be deleted
* A root resolver method `<models>({search, order, pagination}, context)`, which performs a search of the model entries with limit-offset based pagination after checking for authorization (*SEARCH_LO_ROOT*)
* A root resolver method `<models>Connection({search, order, pagination}, context)`, which performs a search of the model entries with cursor based pagination after checking for authorization (*SEARCH_CURSOR_ROOT*)
* A root resolver method `readOne<model>({<ModelIDAttribute>}, context)`, which returns a single record which matches the given ID after checking for authorization (*SINGLE_ROOT*)
* A root resolver method `count<models>: async function({search}, context)`, which returns the number of records which match the given search term after checking for authorization (*COUNT_ROOT*).
* *A (deprecated) root resolver method `vueTable<model>(_, context)`, which returns a table of records as needed for displaying a vuejs table after checking for authorization*
* A root resolver method `add<model>(input, context)`, which adds a record for the model after checking for authorization (*ADDING_ROOT*)
* A root resolver method `bulkAdd<model>Csv(_, context)`, which loads a csv file containing records and adds them to the model after checking for authorization (*ADDING_IN_BULK_ROOT*)
* A root resolver method `delete<model>({<ModelIDAttribute>}, context)`, which deletes the record given by the ID value after checking for authorization (*DELETING_ROOT*)
* A root resolver method `update<model>: async function(input, context)`, which updates a record which is given by the input argument with new values and/or associations as given by the input argument after checking for authorization (*UPDATING_ROOT*)
* A root resolver method `csvTableTemplate<model>(_, context)`, which returns the table's template after checking for authorization (*TEMPLATE*)

The following methods are created in the model:

* A method `static init(sequelize, DataTypes)`, which initializes the model
* *SINGLE_ROOT:* A root resolver implementation method `static async readById(id)`, which returns a single record given by the ID
* *COUNT_ROOT:* A root resolver implementation method `static async countRecords(search)`, which counts the records given by the search term
* *SEARCH_LO_ROOT:* A method `static readAll(search, order, pagination)`, which returns the records given by the search term with limit-offset based pagination
* *SEARCH_CURSOR_ROOT:* A method `static readAllCursor(search, order, pagination)`, which returns the records given by the search term with cursor based pagination
* *ADDING_ROOT:* A method `static addOne(input)`, which adds a record
* *DELETING_ROOT:* A method `static deleteOne(id)`, which deletes a record
* *UPDATING_ROOT:* A method `static updateOne(input)`, which updates a record
* *ADDING_IN_BULK_ROOT:* A method `bulkAddCsv(context)`, which adds several records from a csv file
* *TEMPLATE:* A method `csvTableTemplate()`, which returns the template of a table
* A method `static idAttribute()`, which returns the name of the ID attribute
* A method `static idAttributeType()`, which returns the type of the ID attribute
* A method `getIdValue()`, which returns the value of the ID attribute
* A getter for the model class `static get definition()` which returns the `definition` constant.
* A method `static base64Decode(cursor)`, to decode a base 64 representation of a given cursor into a UTF-8 string
* A method `base64Encode()`, which encodes the current cursor into a base 64 string
* A method `stripAssociations()`, which returns only the attributes of the current record
* A method `externalIdsArray()`, which returns `definition.externalIds` if present, otherwise `[]`.
* A method `externalIdsObject()`, which returns an object containing only the attributes with external IDs as keys, or an empty object.

##### For all associations

In the resolver the following entries are created:

* a constant named `associationsArgsDef` is created which is an object, containing for each association the name of the "add" method as key and the name of the association as value.
* a method named `<model>.prototype.handleAssociation = async function(input, context)`, which handles the execution of all "add"/"remove" statements as promises.
* *a method named `async function countAllAssociatedRecords(id, context)` which counts all existing associations of a record, so that it can be made sure that no records are associated to one if this record is to be deleted.*

In the model a method `static associate(models)` is created, which is filled for each association type as outlined below.

##### Association type *to_many*

For this association the following methods are created in the resolver:

* Two methods to return the associated records via a search, with an order and a pagination (limit-offset-based or cursor-based) possibly added (named `<model>.prototype.<assoc>Filter({search, order, pagination}, context)` for the limit-offset-based case and `<model>.prototype.<assoc>Connection({search, order, pagination}, context)` for the cursor-based case), in both cases by calling the root resolver of the respective associated records
* A method to count the associated records: `<model>.prototype.countFiltered<assoc>({search}, context)` by calling the root resolver of the associated records
* A field resolver method for adding records in a loop that sets the associated ID in the *associated record* to the ID of the main record: `<model>.prototype.add_<assoc> = async function(input)` by calling the model field resolver implementation adding method of the associated record (*ADDING_TO1*)
* A field resolver method for removing records in a loop that removes the associated ID in the *associated record*: `<model>.prototype.remove_<assoc> = async function(input)` by calling the model field resolver implementation deleting method of the associated record (*REMOVING_TO1*)

In the model file, an entry is added to the method `associate(models)` in  the form `<model>.hasMany(models.<assoc>, {as: <assoc>, foreignKey: <assocID>})`. The model methods that are called from the resolver are in the associated model (which holds the association key).

##### Association type *to_one*

For this association the following methods are created in the resolver:

* A method to return the associated record via a search for the given record ID: `<model>.prototype.<assoc> = async function({search}, context)` by calling the root resolver of the associated record for searching with limit-offset based pagination (*SEARCH_LO_ROOT*)
* A field resolver method for adding a record that calls the respective model method (see below): `<model>.prototype.add_<assoc> = async function(input)` (*ADDING_TO1*)
* A field resolver method for removing a record that calls the respective model method (see below): `<model>.prototype.remove_<assoc> = async function(input)` (*REMOVING_TO1*)

In the model the following entries are created:

* An entry to the method `associate(models)` in the form `<model>.belongsTo(models.<assoc>, {as: <assoc>, foreignKey: <assocID>})`
* *ADDING_TO1:* A field resolver implementation method to add an entry by setting the associated ID in the *main record* to the ID of the associated record: `static async add_<assocID>(<ModelIDAttribute>, <assocIDValue>)`
* *REMOVING_TO1:* A field resolver implementation method for removing a record that removes the associated ID in the *main record*: `static async remove_<assocID>(<ModelIDAttribute>, <assocIDValue>)`

##### Association type *to_many_through_sql_cross_table*

In this case an additional record type is introduced that contains the SQL cross table, so there are 3 models and 3 resolvers to consider. Unlike the former two cases, this one is fully symmetric. *Both* records involved define this type of connection with the name of the cross table as `keysIn`. The source key for both types is the ID of the own record, and the target key is the ID of the other record. Because of the symmetry, only 4 files must be considered.

###### Record Resolver

* Two methods to return the associated records via a search for both types of pagination. In this case, special "`impl`" functions of the models are called (see below). Names: `<model>.prototype.<assoc>Filter({search, order, pagination}, context)` for the limit-offset based pagination (*SEARCH_LO_TMTSCT*) and `<model>.prototype.<assoc>Connection({search, order, pagination}, context)` for the cursor based pagination (*SEARCH_CURSOR_TMTSCT*) after checking for authorization
* A method for counting the associated records, calling a special "`impl`" function of the model (see below - *COUNT_TMTSCT* ): `<model>.prototype.countFiltered<assoc>({search}, context)` after checking for authorization
* A field resolver method for adding an associated record, calling the respective model method (see below): `<model>.prototype.add_<assoc> = async function(input)` (*ADDING_TMTSCT*)
* A field resolver method for removing an associated record, calling the respective model method (see below): `<model>.prototype.remove_<assoc> = async function(input)` (*REMOVING_TMTSCT*)

###### Record Model

* An entry to the method `associate(models)` in the form `<model>.belongsToMany(models.<assoc>, {as: <assoc>, foreignKey: <assocID>, through: <CrossTable>, onDelete: 'CASCADE'})`
* *SEARCH_LO_TMTSCT:* An implementation function for the search with limit-offset based pagination: `<assoc>FilterImpl({search, order, pagination})`
* *SEARCH_CURSOR_TMTSCT:* An implementation function for the search with cursor based pagination: `<assoc>ConnectionImpl({search, order, pagination})`
* *COUNT_TMTSCT:* An implementation function for the counting: `countFiltered<assoc>Impl({search})`
* *ADDING_TMTSCT:* A field resolver implementation method for adding a record: `static async add_<assocID>(record, add<assoc>)`
* *REMOVING_TMTSCT:* A field resolver implementation method for removing a record: `static async remove_<assocID>(record, remove<assoc>)`

###### Cross Table Resolver / Model

Normal resolver / model files are created, where the respective model type has no associations of its own.

## Authorization-checks and record limits

Not every access to Cenzontle is permitted. In many cases (see above), the user must be authorized to perform a certain action. Possible authorizations for a given table include `read`, `create`, `delete`, `update`. These authorizations with respect to tables are connected to roles that users can have and are stored within the database.

Additionally, reading actions are refused if they access too many records. GraphQL is a very powerful data query and manipulation language that gives the user the control about what to query the server, but this makes it possible (by accident or malice) to make such a large query that the server cannot handle it. To  prevent this, the server has a set limit of records that can be accessed by a single query.

## Pagination types

As seen above, Cenzontle provides two different types of pagination: Limit-offset based and cursor based. Limit-offset based pagination is the better known one, where the user provides an offset (how many entries to skip) and a limit (how many entries to display on one page).

Limit-offset based pagination is not possible for distributed data models, i.e. models where records of a single table are spread over different servers. This is because the client (how makes the request) does not know how the entries are distributed, so the offset for the different servers cannot be provided. Since this data model is especially useful for Big Data, it is not feasible to request the entire table and paginate the data on the client side.

Instead, a different model has to be used. If the different servers are told which was the last entry "above" the requested page (a cursor), they can deliver the content that would follow up tp it and the client can now get the received data in order. This type of pagination works for any kind of data, although it is more complex (the entries for the table are found under `edges[] -> node`).
