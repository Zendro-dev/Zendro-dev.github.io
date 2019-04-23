# Data Models

For each one of the projects that you want to include in the project you will need to describe the model as well as its possibles relations with any other model. The description should be in a json file following the [json specs](#json-specs) for this purpose.
You will need to store all these json files in a single folder. From now on, in this document, we will assume that all json files for each one of your data models will be stored in the directory `/your-path/json-files`

### JSON Specs

Each json file describes one and only one model. (i.e if an association involves two models, this association needs to be specified in both json files, corresponding to each model).

For each model we need to specify the following fields in the json file:

Name | Type | Description
------- | ------- | --------------
*name* | String | Name of the model (it is recommended uppercase for the initial character).
*storageType* | String | Type of storage where the model is stored. So far can be one of __sql__ or __Webservice__
*attributes* | Object | The key of each entry is the name of the attribute and the value should be the a string indicating the type of the attribute. See [table](#types-spec) below for allowed types. Example: ```{ "attribute1" : "String", "attribute2: "Int" }```
*associations* | Object | The key of each entry is the name of the association and the value should be an object describing the associations. See [Associations Spec](#associations-spec) section below for the specifications of the associations.

### Types Spec
The following types are allowed for the attributes field

 Type |
------- |
String |
Int |
Float |
Boolean |



### Associations Spec

We will consider four possible types of associations:
1. belongsTo
2. hasOne
3. hasMany
4. belongsToMany

For all type of association, except for association  of type 4 (belongsToMany), the necessary arguments would be:

name | Type | Description
------- | ------- | --------------
*type* | String | Type of association (one of the six described above).
*target* | String | Name of model to which the current model will be associated with
*targetKey* | String | Key to identify the field in the target
*targetStorageType* | String | Type of storage where the target model is stored. So far can be one of __sql__ or __Webservice__
*label* | String | Name of the column in the target model to be used as a display name in the GUI
*sublabel* | String | Optional name of the column in the target model to be used as a sub-label in the GUI

When the association is of the type 4, it's necessary to describe a couple of two extra arguments given that the association is made with a cross table. The extra two arguments will be:

name | Type | Description
------- | ------- | --------------
*sourceKey* | String | Key to identify the source id
*keysIn* | String | Name of the cross table

## NOTE:
 It's important to notice that when an association includes at least one model which storage type is not `sql` then foreign keys should be explicitly written in the attributes field.
 For now when BOTH models have `sql` storageType foreigns keys should NOT be explicitly written in the attributes field, because the generator will authomatically add them.

Example:
```
{
  "model" : "Book",
  "storageType" : "sql",
  "attributes" : {
    "title" : "String",
    "genre" : "String",
    "publisher_id": "Int"
  },
  "associations":{

      "publisher" : {
        "type" : "belongsTo",
        "target" : "Publisher",
        "targetKey" : "publisher_id",
        "targetStorageType" : webservice",
        "label" : "name"
        }
  }
}
```

## NOTE:
THE SAME DATA MODELS DESCRIPTION(.json files) WILL BE USEFUL FOR GENERATING BOTH, THE [BACKEND](backendSetUp.md) AND [FRONTEND OR GUI](guiSetUp.md)

Fields *`label`* and *`sublabel`* in the specification are only needed by the GUI generator, but backend generator will only read required information, therefore extra fields such as *`label`* and *`sublabel`* will be ignored by the backend generator.

EXAMPLES OF VALID JSON FILE:
```
//dog.json
{
  "model" : "Dog",
  "storageType" : "Sql",
  "attributes" : {
    "name" : "String",
    "breed" : "String"
  },

  "associations" : {
    "person" : {
      "type" : "belongsTo",
      "target" : "Person",
      "targetKey" : "personId",
      "targetStorageType" : "sql",
      "label": "firstName"
    }
  }
}

```

```
//book.json
{
 "model" : "Book",
 "storageType" : "SQL",
 "attributes" : {
        "id" : Int,
        "title": String,
        "ISBN": Int
    },
 "associations" : {
        "authors" : {
            "type" : "belongsToMany",
            "target" : "Person",
            "targetKey" : "person_id",
            "sourceKey" : "book_id",
            "keysIn" : "person_to_book",
            "targetStorageType" : "sql",
            "label": "name",
            "sublabel": "lastname"
        }
    }
}
```

### About the associations type

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

2. __Many-to-Many__

Based on the previous example, this type of relationship underlines the fact, that an employee can belong to more than one department as well as the department can incorporate more than one employee. In this case, a new relation table has to be created that would hold the foreign-key pairs that define employee-to-department associations.

Example:
A.json belongsToMany B;
B.json belongsToMany A;
(AB relation table with foreignkey_A and foreignkey_B is created automatically)

3. __One-to-One__

This relation type contains restriction: an absence of the possibility to relate more than one pair of the elements. In this case it is a subject of intuition which table shell hold the foreign-key. However, the foreign-keys can't repeat and shall be unique.

For example, the A table is a catalog of well studied species that have strict registration number, whereas table B contains all discovered species, some of which are not cataloged yet.

A.json: hasOne B;
B.json: belongsTo A;
(table B keeps a unique foreignkey_A)
