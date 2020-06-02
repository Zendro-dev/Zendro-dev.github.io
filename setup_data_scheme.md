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

#### NOTE - Foreign keys

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

#### NOTE - Differences between backend and Frontend (GUI)

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

#### The code generator at work (WIP)

The code generator receives the specification as described [above](#json-specs) and generates resolvers and models from it. In this section we take a look at the code that is generated from the different attributes that can be given by the JSON specification.

The model contains a constant named `definition` that contains the full JSON specification. The following placeholders will be used from now on:

* `<record>`: The respective main record type
* `<assoc>`: The name of the associated record (with capitalization and plural forms properly applied)
* `<assocID>`: The association ID
* `<ModelIDAttribute>`: The name of the ID attribute
* `<assocIDValue>`: The actual value of the associated ID
* `<CrossTable>`: The name of the SQL cross table

##### Case *to_many* association

For this association the following methods are created in the resolver:

* Two methods to return the associated records via a search, with an order and a pagination (limit-offset-based or cursor-based) possibly added (named `<record>.prototype.<assoc>Filter({search, order, pagination}, context)` for the limit-offset-based case and `<record>.prototype.<assoc>Connection({search, order, pagination}, context)` for the cursor-based case)
* A method to count the associated records: `<record>.prototype.countFiltered<assoc>({search}, context)`
* A method for adding records in a loop that sets the associated ID in the *associated record* to the ID of the main record: `<record>.prototype.add_<assoc>(input)`
* A method for removing records in a loop that removes the associated ID in the *associated record*: `<record>.prototype.remove_<assoc>(input)`

In the model file, an entry is added to the method `associate(models)` in  the form `<record>.hasMany(models.<assoc>, {as: <assoc>, foreignKey: <assocID>})`. The model methods that are called from the resolver are in the associated model (which holds the association key).

##### Case *to_one* association

For this association the following methods are created in the resolver:

* A method to return the associated record via a search for the given record ID: `<record>.prototype.<assoc>({search}, context)`
* A method for adding a record that calls the respective model method (see below): `<record>.prototype.add_<assoc>(input)`
* A method for removing a record that calls the respective model method (see below): `<record>.prototype.remove_<assoc>(input)`

In the model the following entries are created:

* An entry to the method `associate(models)` in the form `<record>.belongsTo(models.<assoc>, {as: <assoc>, foreignKey: <assocID>})`
* A method to add an entry by setting the associated ID in the *main record* to the ID of the associated record: `static async add_<assocID>(<ModelIDAttribute>, <assocIDValue>)`
* A method for removing a record that removes the associated ID in the *main record*: `static async remove_<assocID>(<ModelIDAttribute>, <assocIDValue>)`

##### Case *to_many_through_sql_cross_table*

In this case an additional record type is introduced that contains the SQL cross table, so there are 3 models and 3 resolvers to consider. Unlike the former two cases, this one is fully symmetric. *Both* records involved define this type of connection with the name of the cross table as `keysIn`. The source key for both types is the ID of the own record, and the target key is the ID of the other record. Because of the symmetry, only 4 files must be considered.

###### Record Resolver

* Two methods to return the associated records via a search for both types of pagination. In this case, special "`impl`" functions of the models are called (see below). Names: `<record>.prototype.<assoc>Filter({search, order, pagination}, context)` for the limit-offset based pagination and `<record>.prototype.<assoc>Connection({search, order, pagination}, context)` for the cursor based pagination
* A method for counting the associated records, calling a special "`impl`" function of the model (see below): `<record>.prototype.countFiltered<assoc>({search}, context)`
* A method for adding an associated record, calling the respective model method (see below): `<record>.prototype.add_<assoc>(input)`
* A method for removing an associated record, calling the respective model method (see below): `<record>.prototype.remove_<assoc>(input)`

###### Record Model

* An entry to the method `associate(models)` in the form `<record>.belongsToMany(models.<assoc>, {as: <assoc>, foreignKey: <assocID>, through: <CrossTable>, onDelete: 'CASCADE'})`
* An implementation function for the search with limit-offset based pagination: `<assoc>FilterImpl({search, order, pagination})`
* An implementation function for the search with cursor based pagination: `<assoc>ConnectionImpl({search, order, pagination})`
* An implementation function for the counting: `countFiltered<assoc>Impl({search})`
* A method for adding a record: `static async add_<assocID>(record, add<assoc>)`
* A method for removing a record: `static async remove_<assocID>(record, remove<assoc>)`

###### Cross Table functions

Normal resolver / model files are created, where the respective model type has no associations of its own.
