[ &larr; back](setup_root.md)
<br/>
# Data Models

For each one of the data sets that you want to include in the project you will need to describe the data model. This description should include its relations or associations with any other model. The description should be placed in a json file following the [json specs](#json-specs) for this purpose. You will need to store all these json files in a single folder. Another limitation is that each model should have a unique name independently of its type. From now on, in this document, we will assume that all json files for each one of your data models will be stored in the directory `/your-path/json-files`

### JSON Specs

Each json file describes one and only one model. However, one model can reference another model using the associations mechanism described below.

For a complete description of each model we need to specify the following fields in the json file:

Name | Type | Description
------- | ------- | --------------
*model* | String | Name of the model (it is recommended to use snake_case naming style to obtain nice names in the auto-generated GraphQL API). The string here can not contain spaces.
*storageType* | String | Type of storage where the model is stored. So far can be one of: <ul> <li>  __sql__ for local relational databases supported by [sequelize](#http://docs.sequelizejs.com/) such as PostgreSql/MySql etc. </li>  <li>__generic__ for any database that your project would connect remotely.</li>  <li>__zendro\_server__ for models stored in any other instance created with zendro-tools.</li></ul>
*url* | String | This field is only mandatory for __zendro\_server__ stored models. Indicates the url where the zendro server storing the model is runnning.
*attributes* | Object |  The key of each entry is the name of the attribute and there are two options for the value . It can be either a string indicating the type of the attribute or an object where the user indicates the type of the attribute(in the _type_ field) together with an attribute's description (in the _description_ field). See the [table](#supported-data-types) below for allowed types. Example of option one: ```{ "attribute1" : "String", "attribute2: "Int" }``` Example of option two: ``` { "attribute1" : {"type" :"String", "description": "Some description"}, "attribute2: "Int ```
*associations* | Object | The key of each entry is the name of the association and the value should be an object describing the corresponding association. See [Associations Spec](#associations-spec) section below for details.
*internalId* | String | This string corresponds to the name of the attribute that uniquely identifies a record. If this field is not specified, an _id_, default attribute, will be added. 

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

We will consider two types of associations according to the number of records
that can be associated:
1. to_one
2. to_many

For both types of association, the necessary arguments would be:

name | Type | Description
------- | ------- | --------------
*type* | String | Type of association, either `to_one` or `to_many`.
*target* | String | Name of model to which the current model will be associated with.
*targetKey* | String | A unique identifier of the association stored in any of the two models involved in the association.
*keyIn* | String | Name of the model where the targetKey is stored.
*targetStorageType* | String | Type of storage where the target model is stored.
*label* | String | Name of the column in the target model to be used as a display name in the GUI. This will be useful as a preview of the association.
*sublabel* | String | Optional name of the column in the target model to be used as a sub-label in the GUI. Also used for a more explicit preview of the association.

When the association is of type *to_many*, and it refers to a more particular type of association *many_to_many*,  it's necessary to describe two extra arguments given that the association is made with a cross table. These arguments are:

name | Type | Description
------- | ------- | --------------
*sourceKey* | String | Key to identify the source id
*keysIn* | String | Name of the cross table

## NOTE:
Be aware that in the case of this type of association the user is required to describe the cross table used in the field _keysIn_ as a model in its own. For example, if we have a model `User` and a model `Role`, and they are associated in a _manytomany_ way, then we also need to describe the `role_to_user` model:

```
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

```
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

```
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

## NOTE:
It's important to notice that when a model involves a foreign key for the association, this key should be explicitly written into the attributes field of the given local model. Although, foreign keys will be available for the user only as readable attributes, for editing these attributes we offer the possibility as part of the API, please see [this](api_graphql.md#extra-mutation-fields-to-update-or-create-associations) section for more info.


Example:
```
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

## NOTE:

The same data model description files (.json) can be used for generating both the [BACKEND](setup_backend.md) and [FRONTEND OR GUI](setup_gui.md). Fields such as  *`label`* and *`sublabel`* in the model specification that are only needed for GUI generator are ignored by the backend generator.

### About the associations type

In Zendro tools, associations are based on the [sequelize ORM terminology](http://docs.sequelizejs.com/manual/tutorial/associations.html). As usually, in any association type the foreign-key shall be placed inside one of the two associated tables. However, here both table models need to be notified about that link. This association information shall be placed in each model definition JSON files. Below we consider the models for two tables A and B, that are defined in the files A.json and B.json correspondingly.

Within the body of any model JSON file, the model that is described in this file is considered as a source model, and any other model is considered as a target. Therefore, within the JSON source it is required to specify how the current model is associated with all its targets.

The four types of association provided by Sequelize can be taken as a guide for understanding where the `targeKey`, responsible of the association between two tables, is stored.

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

2. __Many-to-Many__

Based on the previous example, this type of relationship underlines the fact that an employee can belong to more than one department as well as the department can incorporate more than one employee. In this case, a new relation table has to be created that would hold the foreign-key pairs that define employee-to-department associations.

Example:
A.json belongsToMany B;
B.json belongsToMany A;
(AB relation table with foreignkey_A and foreignkey_B is created automatically)

3. __One-to-One__

This relation type contains a restriction: an absence of the possibility to relate more than one pair of the elements. In this case, it is a subject of intuition which table shall hold the foreign-key. However, the foreign-keys can't repeat and shall be unique.

For example, the A table is a catalog of well studied species that have strict registration number, whereas table B contains all discovered species, some of which are not cataloged yet.

A.json: hasOne B;
B.json: belongsTo A;
(table B keeps a unique foreignkey_A)
