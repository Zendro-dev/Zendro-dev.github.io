---
layout: default
title: Cassandra storage type
parent: Data models
nav_order: 5
permalink: /data-models/cassandra-storage-type
---

# Cassandra storage type
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

This page covers specifics related to the `cassandra` [storageType]({% link data-models/json-specification.md %}#json-specs). [Cassandra](https://cassandra.apache.org/_/index.html) is, in general, *not* a general purpose database — it aims to provide efficient access to Big Data by carefully describing the data and its specific access patterns. Zendro provides a standardized API for all defined models and assumes certain considerations when defining your data model. For the Cassandra storageType, a unique primary key attribute is required, providing access to a specific record identified by that key. Aside from simple primary keys, Cassandra also offers [compound primary keys](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useCompoundPrimaryKeyConcept.html) to define efficient access across a cluster of servers — these are *not* currently supported. A primary key *must* refer to a single attribute.

## Restrictions
Compared to relational databases, accessing and querying data stored in a Cassandra database has several restrictions.

### Operators
Zendro queries the Cassandra database directly via [CQL](https://cassandra.apache.org/doc/latest/cql/index.html) (Cassandra Query Language) statements, using the DataStax [cassandra-driver](https://docs.datastax.com/en/developer/nodejs-driver/4.6/). CQL implements only a subset of the logical operators available in, for example, relational databases. The operators included in Zendro are:

zendro-operator | operation
------- | -------
eq | `= `
lt | `<`
gt | `>`
lte | `<=`
gte | `>=`
ne | `!=`
in | `in`
contains | `contains`*
and | `and`

\* `contains` relates to Cassandra [Collections](https://cassandra.apache.org/doc/latest/cql/types.html#collections).

**Note** that Cassandra specifically does *not* implement the logical `or` operator.

### Pagination
Due to its distributed nature, Cassandra does *not* implement traditional limit-offset pagination. This means *only* the [cursor-pagination based GraphQL Connection type](https://graphql.org/learn/pagination/#complete-connection-model) `readAllCursor` query is supported. Zendro implements cursor-based pagination via the base64-encoded record.

**Note** that Cassandra does *not* support [backward pagination]({% link api/graphql-reference.md %}#pagination-argument), so the only valid pagination arguments are `first` and `after`.

### Sorting
Cassandra *only* supports sorting query results by specifically defining this via the [compound primary key](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useCompoundPrimaryKeyConcept.html) and the column used to partition the data. Since Zendro does not implement that kind of primary key for now, *no* sorting of Cassandra results is possible. The default order is defined by Cassandra's internal [`token`](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useToken.html) function.

### Associations
Associations with `targetStorageType` set to Cassandra have restrictions on searching. The association is resolved by adding a search for either `eq` on the respective foreign key, or `in` on the foreign key array in the case of `many_to_many` associations. Cassandra does not allow multiple equality restrictions on the ID field — the driver throws an error — so a workaround merges searches on the ID field with the search on the foreign key(s).

This workaround only works because Cassandra does *not* support the `or` operator. There are also the following pitfalls to consider:
- Cassandra does not allow `SELECT` queries with an `IN` clause on indexed columns for the primary key.
- Multiple equality restrictions on the ID field will throw an error.
- Multiple searches on the ID attribute will still throw the above error, since search nodes are merged one by one with the foreign key(s).
- This workaround only applies to associations where the foreign key is stored on the Cassandra model's side, since `IN` clauses are only allowed on the primary key column, not on any foreign key column.

### Access control
CQL implements an optional [`ALLOW FILTERING`](https://cassandra.apache.org/doc/latest/cql/dml.html?highlight=allow%20filtering#allow-filtering) argument for its queries, allowing server-side filtering of the result set. Since these queries may have unpredictable performance depending on factors like secondary indices on columns, `ALLOW FILTERING` must be explicitly requested in the query.

Zendro implements this via the data models' Access Control List. *Only* users with the *editor* role can send queries that require server-side filtering.

## Collection types
Cassandra implements several different [collection](https://cassandra.apache.org/doc/latest/cql/types.html#collections) types — `sets`, `lists` and `maps` — each with different characteristics. Zendro uses `sets` and `lists` depending on the use case.

### Foreign key arrays
Zendro implements `many_to_many` relations between models via [paired-end foreign keys]({% link data-models/associations.md %}#paired-end-foreign-keys). In a Cassandra data model this is implemented via a `set`, a sorted list of unique values of a specific data type.

### Array type attributes
Zendro supports array-type attributes, defined in the [JSON data model definition]({% link data-models/json-specification.md %}#json-specs) with square brackets, e.g. `[String]`. In Cassandra models, Zendro implements this internally via `list` data types, a sorted collection of non-unique values.

## Distributed data models
Due to the restrictions on ordering result sets in Cassandra (see [Sorting](#sorting) above), it is not possible to define a distributed data model stored in both a relational and a Cassandra database, i.e. one with both sql and cassandra adapters. It is, however, possible to define distributed data models that live *only* in Cassandra databases. These need to set a `cassandraRestrictions` flag to ensure correct behaviour:

```json
"model": "dog",
"storageType": "distributed-data-model",
"registry": ["dog_instance1", "dog_instance2"],
"cassandraRestrictions": true,
```
