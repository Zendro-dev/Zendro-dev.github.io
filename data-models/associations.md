---
layout: default
title: Associations
parent: Data models
nav_order: 2
permalink: /data-models/associations
---

# Associations
{: .no_toc }
How to define One-to-One, One-to-Many, Many-to-One and Many-to-Many associations between data models.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

There are four types of associations, according to the relation between associated records of the two models:
1. `one_to_one`
2. `many_to_one`
3. `one_to_many`
4. `many_to_many`

For all association types, the following arguments are required:

name | Type | Description
------- | ------- | --------------
*type* | String | Type of association: `one_to_one`, `one_to_many`, `many_to_one`, or `many_to_many`.
*target* | String | Name of the model the current model will be associated with.
*implementation* | String | Implementation type of the association. Can be one of `foreignkeys`, `generic` or `sql_cross_table` (only for `many_to_many`).
*reverseAssociation* | String | The name of the reverse association from the other model. This field is only mandatory for building the [single-page-app](https://github.com/Zendro-dev/single-page-app), *not* for generating the graphql-server code.
*targetStorageType* | String | Type of storage where the target model is stored.
*useDataLoader* | Boolean | If `true`, the server can fetch multiple records within one query for the `readOne<model>` API.
*deletion* | String | Sets the record deletion behaviour. Defaults to `reject`. <br> `reject`: deletion of a record is rejected if there are related records for this association. <br> `update`: deletion of a record always succeeds, and all connections to related records via this association are dissolved automatically.

## Foreign keys
When a model involves a foreign key for the association, that key must be explicitly written into the attributes field of the given local model. Foreign keys are available to the user only as readable attributes; to edit them, use the API mechanism described in [Extra mutation fields to update or create associations]({% link api/graphql-reference.md %}#extra-mutation-fields-to-update-or-create-associations).

To store to-many associations (many-to-many or one-to-many) via foreign keys, Zendro stores the foreign keys in arrays: the model has an array attribute holding the IDs of the associated records.

### Single-end foreign keys
Storing the foreign key on a single end of the association means only one of the two associated data models holds the foreign-key attribute. This guarantees fast write actions, avoids error-prone operations when writing multiple records to update an association, and requires less storage space — but it can become slow to read and search, especially in a distributed context where the associated records may be spread over multiple servers.

To define single-end foreign key associations, add the following arguments:

name | Type | Description
------- | ------- | --------------
*targetKey* | String | A unique identifier for the association, stored in either of the two models involved. Can be an array for to-many associations.
*keysIn* | String | Name of the model where `targetKey` is stored.

Examples:

```js
{
    "model" : "book",
    "storageType" : "sql",
    "attributes" : {
        "title" : {
            "type": "String",
            "description": "The book's title"
        },
        "publisher_id": "Int"
    },
    "associations": {
        "publisher" : {
            "type" : "many_to_one", // association type
            "implementation": "foreignkeys",
            "reverseAssociation": "books",
            "target" : "publisher", // the target model name is `publisher`
            "targetKey" : "publisher_id", // foreign key for this association
            "keysIn": "book", // FK to `publisher` will be stored in the `book` model
            "targetStorageType" : "sql"
        }
    }
}
```

```js
{
    "model" : "publisher",
    "storageType" : "sql",
    "attributes" : {
        "publisher_id": "Int",
        "publisher_name": "String"
    },
    "associations": {
        "books" : {
            "type" : "one_to_many", // association type
            "implementation": "foreignkeys",
            "reverseAssociation": "publisher",
            "target" : "book", // the target model name is `book`
            "targetKey" : "publisher_id", // foreign key for this association
            "keysIn": "book", // FK to `book` will be stored in the `book` model
            "targetStorageType" : "sql"
        }
    }
}
```

### Paired-end foreign keys
Storing the association via paired-end foreign keys means both associated data models contain a reference (foreign key) to the associated records. This guarantees read and search efficiency, especially in a distributed context, at the cost of time and storage space when handling write actions — since keys are stored at both ends, information needs to be updated at both ends too, which is slower and more error-prone.

Many-to-many associations can be stored via paired-end associations. In this case both models hold an array attribute storing the IDs of the associated records; these two attributes are described in the association as `sourceKey` and `targetKey`. To indicate that the association is many-to-many via arrays as foreign keys, set `implementation` to `foreignkeys`.

To define paired-end foreign key associations, add the following arguments:

name | Type | Description
------- | ------- | --------------
*sourceKey* | String | Attribute belonging to the source model, storing the associated IDs of the target model. Can be an array for to-many associations.
*targetKey* | String | Attribute belonging to the target model, storing the associated IDs of the source model. Can be an array for to-many associations.
*keysIn* | String | Name of the model where `sourceKey` is stored.

Examples:

Assume a many-to-many association between models `book` and `author`, and a one-to-many association between `author` and `card`:

```js
{
    "model" : "author",
    "storageType" : "sql",
    "database": "default-sql",
    "attributes" : {
        "id": "String",
        "name": "String",
        "lastname": "String",
        "email": "String",
        "book_ids": "[ String ]",
        "card_ids": "[ String ]"
    },

    "associations": {
        "books": {
            "type": "many_to_many", // association type
            "implementation": "foreignkeys",
            "reverseAssociation": "authors",
            "target": "book", // target model name
            "targetKey": "author_ids", // foreign key array stored in target model
            "sourceKey": "book_ids", // foreign key array stored in source model
            "keysIn": "author", // source model name
            "targetStorageType": "sql"
        },
        "cards": {
            "type": "one_to_many", // association type
            "implementation": "foreignkeys",
            "reverseAssociation": "author",
            "target": "card", // target model name
            "targetKey": "author_id", // foreign key stored in target model
            "sourceKey": "card_ids", // foreign key array stored in source model
            "keysIn": "author", // source model name
            "targetStorageType": "sql"
        }
    },

    "internalId": "id"
}
```

```js
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

    "associations": {
        "authors": {
            "type": "many_to_many", // association type
            "implementation": "foreignkeys",
            "reverseAssociation": "books",
            "target": "author", // target model name
            "targetKey": "book_ids", // foreign key array stored in target model
            "sourceKey": "author_ids", // foreign key array stored in source model
            "keysIn": "book", // source model name
            "targetStorageType": "sql"
        }
    },

    "internalId": "id"
}
```
```js
{
    "model" : "card",
    "storageType" : "sql",
    "database": "default-sql",
    "attributes" : {
        "card_id": "String",
        "genre": "String",
        "author_id": "String"
    },

    "associations": {
        "author": {
            "type": "many_to_one", // association type
            "implementation": "foreignkeys",
            "reverseAssociation": "cards",
            "target": "author", // target model name
            "targetKey": "card_ids", // foreign key array stored in target model
            "sourceKey": "author_id", // foreign key stored in source model
            "keysIn": "card", // source model name
            "targetStorageType": "sql"
        }
    },

    "internalId": "id"
}
```

## Many-to-many through a cross table
When the association is `many_to_many` and is stored via a cross table, set `implementation` to `sql_cross_table` — this is only available for `sql`-stored models.

Example:
```js
//User model
{
    "model" : "User",
    "storageType" : "SQL",
    "attributes" : {
        "email" : "String",
        "password" : "String"
    },
    "associations" : {
        "roles" : {
            "type" : "many_to_many", // association type
            "implementation": "sql_cross_table",
            "reverseAssociation": "users",
            "target" : "Role", // target model name
            "targetKey" : "role_Id", // foreign key stored in target model
            "sourceKey" : "user_Id", // foreign key stored in source model
            "keysIn" : "role_to_user", // cross table name
            "targetStorageType" : "sql",
            "label": "name"
        }
    }
}
```

```js
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
            "type" : "many_to_many", // association type
            "implementation": "sql_cross_table",
            "reverseAssociation": "roles",
            "target" : "User", // target model name
            "targetKey" : "user_Id", // foreign key stored in target model
            "sourceKey" : "role_Id", // foreign key stored in source model
            "keysIn" : "role_to_user", // cross table name
            "targetStorageType" : "sql",
            "label": "email"
        }
    }
}
```

```js
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

## Generic associations
To generate a generic association, use the `generic` implementation type. This generates code stubs in the models, for you to implement the management of the association yourself.

## Differences between backend and frontend (GUI)
The same data model description files (`.json`) are used to generate both the backend and frontend. Fields such as *`label`* and *`sublabel`* in the model specification, only needed for the GUI generator, are ignored by the backend generator.

The field `reverseAssociation` is only mandatory for generating the queries used by the single-page-app to communicate with the graphql-server. Generating the graphql-server code without setting this field produces a warning.

## About the association types

In Zendro there are four association types: **\* : 1** (many-to-one), **1 : \*** (one-to-many), **\* : \*** (many-to-many) and **1 : 1** (one-to-one).

As usual, in any association type the foreign key is placed inside one of the two associated tables, but both table models need to be notified about that link — this association information is placed in each model's definition JSON file. Below we consider models for two tables, A and B, defined in files `A.json` and `B.json` respectively.

Within the body of any model JSON file, the model described in that file is the *source* model, and any other model referenced is a *target*. So within the JSON source, you need to specify how the current model is associated with all its targets.

Some examples for the four association types:

### Many-to-one
This happens when more than one element of table A can reference the same element of table B — the foreign key is created in table A.

Example: table A contains a list of employees, and each of them works in a single department only (catalog B).

Data model definition for table A:
```json
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

(Table A keeps a non-unique `foreignkey_B`.)

### One-to-many
This is the reverse of the many-to-one association above, so we can reuse the same example. Data model definition for table B:

```json
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

### Many-to-many
Based on the previous example: an employee can belong to more than one department, and a department can have more than one employee. A new relation table needs to be created, holding the foreign-key pairs that define employee-to-department associations.

Data model definition for table A:
```json
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
Data model definition for table B:

```json
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

(An AB relation table with `foreignkey_A` and `foreignkey_B` is created automatically.)

### One-to-one
This relation type comes with a restriction: it's not possible to relate more than one pair of elements. It's usually a matter of intuition which table should hold the foreign key, but the foreign keys can't repeat and must be unique.

Example: table A is a catalog of well-studied species with a strict registration number, while table B contains all discovered species, some of which are not yet cataloged.

Data model definition for table B:
```json
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
(Table B keeps a unique `foreignkey_A`.)

## Next steps

* [Resolver and model layers]({% link data-models/resolver-and-model-layers.md %}) — the code the generator builds for each association type.
* [Extended API with associations]({% link api/graphql-reference.md %}#extended-api-with-associations) — the GraphQL queries and mutations generated for associations.
