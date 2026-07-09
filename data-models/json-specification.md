---
layout: default
title: JSON specification
parent: Data models
nav_order: 1
permalink: /data-models/json-specification
---

# JSON specification
{: .no_toc }
Detailed technical specification for Zendro's data model JSON files, aimed at software developers and system administrators.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

For each data set you want to include in the project, you need to describe its data model. This description should include its relations, or associations, with any other model, and must be placed in a JSON file following the specification below. Store all these JSON files in a single folder — from now on, in this document, we'll assume that folder is `/your-path/json-files`. Each model should have a unique name, independent of its type.

## JSON Specs

Each JSON file describes one and only one model. However, one model can reference another model using the associations mechanism described in [Associations]({% link data-models/associations.md %}).

To fully describe a model, specify the following fields in its JSON file:

Name | Type | Description
------- | ------- | --------------
*model* | String | Name of the model (it is recommended to use snake_case naming style to obtain nice names in the auto-generated GraphQL API). The string here cannot contain spaces.
*model_name_in_storage* | String | The name of the model in the storage itself, e.g. the table name in relational databases, the collection in MongoDB, the node in Neo4j, etc. By default Zendro uses the lowercase pluralized *model* property.
*database* | String | Name of the database connection as a key defined in [`data_models_storage_config.json`](https://github.com/Zendro-dev/graphql-server/blob/master/config/data_models_storage_config.json). If this field is not defined, the database connection used will be `default-<storageType>`.
*storageType* | String | Type of storage where the model is stored. So far can be one of:<br/> - **sql** for local relational databases supported by [sequelize](http://docs.sequelizejs.com/), such as PostgreSQL/MySQL etc.<br/> - **generic** for any database that your project connects to remotely.<br/> - **zendro-server** for models stored in another instance created with zendro-tools.<br/> - **cassandra** for local Cassandra databases, supported by the DataStax node [cassandra-driver](https://docs.datastax.com/en/developer/nodejs-driver/4.6/). See [Cassandra storage type]({% link data-models/cassandra-storage-type.md %}) for Cassandra-specific restrictions.<br/> - **mongodb** for local MongoDB databases, supported by the mongodb-driver.<br/> - **neo4j** for local Neo4j databases, supported by the neo4j-driver.<br/> - **presto/trino** for local Presto/Trino databases, supported by the presto-driver.<br/> - **amazon-s3** for the Amazon S3 cloud storage service and local object storage MinIO, supported by the amazon-s3-driver.<br/> - **distributed-data-model** for a distributed setup, which connects all relevant adapters.<br/> - **adapter** for the different adapters in a distributed setup: sql-adapter, generic-adapter, cassandra-adapter, mongodb-adapter, amazonS3-adapter, trino-adapter, neo4j-adapter, ddm-adapter, zendro-webservice-adapter.
*url* | String | Only mandatory for **zendro-server** stored models. Indicates the URL where the Zendro server storing the model is running.
*attributes* | Object | The key of each entry is the name of the attribute. The value can either be a string indicating the attribute's type, or an object where you indicate the type (in the `type` field) together with a description (in the `description` field). See the [table](#supported-data-types) below for allowed types. Example of option one: `{ "attribute1" : "String", "attribute2": "Int" }`. Example of option two: `{ "attribute1" : {"type": "String", "description": "Some description"}, "attribute2": "Int" }`
*associations* | Object | The key of each entry is the name of the association, and the value should be an object describing the corresponding association. See [Associations]({% link data-models/associations.md %}) for details.
*indices* | [String] | Attributes for which to generate indices. By default, indices are generated for *internalId*; it is recommended to also add indices for attributes that are foreign keys.
*operatorSet* | String | Lets you specify the operator set for generic models, distributed adapters and Zendro servers. The following operator sets are supported: `GenericPrestoSqlOperator`, `MongodbNeo4jOperator`, `CassandraOperator`, `AmazonS3Operator`. See [documentation of operators]({% link api/graphql-reference.md %}#operators) for details.
*internalId* | String | The name of the attribute that uniquely identifies a record. If this field is not specified, a default attribute named *id* will be added.
*spaSearchOperator* | 'like' \| 'iLike' | Optional attribute to specify which operator the single-page-app text search field should use. Defaults to `iLike`.

## Supported Data Types

The following types are allowed for the attributes field:

Type |
------- |
String |
Int |
Float |
Boolean |
Date |
Time |
DateTime |

For more info about `Date`, `Time`, and `DateTime` types, please see [this reference](https://github.com/excitement-engineer/graphql-iso-date/blob/HEAD/rfc3339.txt).

Examples:

* Date: a date string, such as `2007-12-03`.
* Time: a time string at UTC, such as `10:15:30Z`.
* DateTime: a date-time string at UTC, such as `2007-12-03T10:15:30Z`.

## Next steps

* [Associations]({% link data-models/associations.md %}) — connecting data models to each other.
* [Resolver and model layers]({% link data-models/resolver-and-model-layers.md %}) — what the code generator builds from this specification.
