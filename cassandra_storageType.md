[ &larr; back](setup_data_scheme.md)
<br/>

# Cassandra storageType

This documentation aims to point out specifics related to the cassandra [storageType](setup_data_scheme.md#json-specs). [Cassandra]() in general is _not_ general purpose database and aims at providing efficient access to Big Data by carefully describing the data and specific access patterns. Zendro provides a standardized API for all defined models and assumes certain considerations when defining your data model. Important for the cassandra storageType is the presence of a unique primary Key attribute providing access to a specific record identified by that key. Aside from simple primary keys cassandra offers [compound primary keys](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useCompoundPrimaryKeyConcept.html) to define efficient access on a cluster of servers. For now we _do not_ support these kind of primary key definitions. A primary key _must_ refer to one attribute.

## Restrictions
In comparision to relational databases, access and querying of data stored in a cassandra database has several restrictions. 

### Operators
Zendro directly implements querying of the cassandra database directly via [CQL](https://cassandra.apache.org/doc/latest/cql/index.html) (Cassandra Query Language) statements using the datastax [cassandra-driver](https://docs.datastax.com/en/developer/nodejs-driver/4.6/). CQL implements only a subset of logical operators available for example in relational databases. The operators included in Zendro are.

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

\* contains relates to cassandra [Collections](https://cassandra.apache.org/doc/latest/cql/types.html#collections).  

**_Note_** that cassandra specifically does _not_ implement the logical `or` operator.

### Pagination
Cassandra, due to its distributed nature does _not_ implement traditional limit-offset like pagination. That implies that _only_ the [cursor-pagination based graphql Connection type](https://graphql.org/learn/pagination/#complete-connection-model) `readAllCursor` query is supported. zendro implements the cursor-based-pagination via the `base64` encoded record.  

**_Note_** that cassandra does _not_ support [backward pagination](api_graphql.md#pagination-argument), so valid arguments given to the pagination argument can be `first` and `after`.

### Sorting
Cassandra _only_ supports the sorting of the results from a query by specifically defining this via the [compound primary key](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useCompoundPrimaryKeyConcept.html) and the definition of a column by which to partition the data. Since zendro, for now, doen't implement those kind of primary keys, _no_ sorting of cassandra results is possible. The default is defined by cassandras internal [`token`](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useToken.html) function.

### Associations
Associations with the `targetStoragetType` set to Cassandra have some restrictions on searching. Since the association is resolved via adding a search for either `eq` to the respective foreignkey or `in` the foreignkey array in case the association is of type `many_to_many`. Cassandra does not allow multiple Equal restrictions on the id field, the driver will throw an Error. To circumvent that a workaround where searches on the idField are merged with the search on the foreignkey(s) is implemented.

Be aware that the workaround only works because cassandra does _not_ support the OR operator. There are also the following pitfalls to consider: 
- cassandra does not allow SELECT queries on indexed columns with IN clause for the PRIMARY KEY.

- If there are multiple Equal restrictions on the id field cassandra will throw an Error.
- multiple searches on the idAttribute field will still throw the above Error since search nodes are merged one-by-one with the foreignkey(s).
- This workaround is only for associations where the foreignKey is stored on the side of the cassandra model, since IN clauses are only allowed on the primarykey column, not on any foreignkey column.

### Access Control
CQL implements an optional [`ALLOW FILTERING`](https://cassandra.apache.org/doc/latest/cql/dml.html?highlight=allow%20filtering#allow-filtering) argument to its queries that allows server side filtering of the result set. Since these types of queries may exhibit unpredictable performance issues, depending on several aspects such as secondary indeces on columns etc., it is required to specifically allow server side filtering directly in the query.  

Zendro implements this via Access Control of the data models. _Only_ users with the _editor_ role have priviledged access to send those queries that require server side filtering.

## Collection types
Cassandra implements several different data types regarding [collections](https://cassandra.apache.org/doc/latest/cql/types.html#collections), namley `sets`, `lists` and `maps` each with different characteristics. Zendro makes use of `sets` and `lists` depending on the use case.

### Foreign Key arrays
Zendro implements `many_to_many` relations between models via [foreign key arrays](setup_data_scheme.md#many-to-many-association-through-foreign-key-arrays). In a cassandra datamodel this is solved via a `set`, which represents a sorted list of unique values of a specific data-type.

### Array type attributes
Zendro supports Array type attributes by defining them in the [JSON data-model-definition](setup_data_scheme.md#json-specs) within square brackets, e.g. `[String]`. In cassandra models zendro internally solves this via `list` data-types, which represents a (sorted) collection of non-unique values.

## Distributed data models
Due to the nature of restrictions related to the ordering of result sets in cassandra (see [above](#Sorting)) it is not possible to define a distributed data model that is stored in both a relational and a cassandra database, i.e. has sql and cassandra adapters. It is however possible to define distributed data models that _only_ live in cassandra databases. These need to set a `cassandraResctrictions` flag to ensure the correct behaviour of the distributed setup:

```JSON
"model": "dog",
"storageType" : "distributed-data-model",
"registry": ["dog_instance1, dog_instance2"],
"cassandraRestrictions": true,
...
```
