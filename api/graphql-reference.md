---
layout: default
title: GraphQL reference
parent: Zendro API
nav_order: 2
permalink: /api/graphql-reference
---

# GraphQL
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
Concrete requests sent to the backend server are model-dependent. Let's assume our first data model is called `record`, described as follows:

```js
//Record.json
{
  "model": "record",
  "storageType": "Sql",
  "attributes": {
    "name": "String",
    "description": "String"
  }
}
```

### GraphQL Queries
* `records(search, order, pagination) : [Records]` - Checks user authorization and returns a number of records, specified by the pagination argument, matching the search argument, sorted as specified by the order argument. See [General filter arguments](#general-filter-arguments) below for details on `search`, `order` and `pagination`. Example:
  ```python
  query {
    records(search: {field: name, value: { value: "%test%"}, operator: like}, order: [ {field: name, order: ASC}], pagination: {limit: 10}) {
      name
      description
    }
  }
  ```

* `readOneRecord(id): Record` - Checks user authorization and returns the record matching the given ID. Example:
  ```python
  query {
    readOneRecord(id: 23) {
      name
      description
    }
  }
  ```

* `countRecords(search): Integer` - Counts the records matching the conditions specified in the search argument. Example:
  ```python
  query {
    countRecords( search: {field: name, value: { value: "%test%"}, operator: like} )
  }
  ```

* `vueTableRecord: vueTableRecord` - Returns a table of records as needed for displaying a vuejs table. Example:
  ```python
  query {
    vueTableRecord {
      data {
        name
        description
      }
    }
  }
  ```
### Mutations

* `addRecord(record): Record` - Checks user authorization and creates a new record with the data specified in the record argument. Example:
  ```python
  mutation {
    addRecord(name: "testRecord", description: "testing record" ) {
      name
      description
    }
  }
  ```

* `deleteRecord(id): String` - Checks user authorization and deletes the record with the specified ID. Example:
  ```python
  mutation {
    deleteRecord(id: 23)
  }
  ```

* `updateRecord(record): Record` - Checks user authorization and updates the record specified in the input argument. Example:
  ```python
  mutation {
    updateRecord(id: 23 name: "updated name") {
      name
      description
    }
  }
  ```

* `bulkAddRecordCsv: String` - Loads a csv file of records. The csv file must be attached to the request.

### General Filter Arguments
When retrieving a set of records of any data model, specific arguments can help select only certain records. Two general arguments are pagination and order — see below:

#### Search argument
This argument's type depends on the data model name. Assuming our data model is called `Record`, the GraphQL type of this argument is called `searchRecordInput`, an object with the following fields:

name | Type | Description
------- | ------- | --------------
*field* | String | Any of the record's attribute names — the column by which records are filtered.
*value* | Object | Value used to filter the records; can be type `String` or type `Array` (default type is String), and the actual value should also be specified. Example: `value: { type: String, value: "%string_to_filter%"}`
*valueType* | enum | One of `Array, String, Int, Float, Boolean, DateTime`
*operator* | String | Operator used to filter the records. Example: `eq`, `like` ...
*search* | [searchRecordInput] | Recursively, another search argument.

Although the search argument's type depends on the data model name, the argument name is always `search`.

**Example**: to filter the first 100 records whose name contains the substring 'test':

```python
query {
  records(search: {field: name, value: "%test%", operator: like}, pagination: {limit: 100}) {
    name
    description
  }
}
```

#### Operators
Zendro supports the following operators. Depending on the storage type of the model, some operators are not supported and hence not exposed in the model's schema.

##### Pattern matching operators

Operator | Description | Example
--- | --- | ---
`like` | pattern matching with wildcards for the entire string | `value: "%abc_"`
`notLike` | negated `like` | `value: "%abc%"`
`iLike` | case insensitive pattern matching with wildcards for the entire string | `value: "%abc_"`
`notILike` | negated `iLike` | `value: "%abc%"`
`regexp` | pattern matching via regular expression | `value: "^[a\|b\|c]"`
`notRegexp` | negated `regexp` | `value: "^[a\|b\|c]"`
`iRegexp` | case insensitive pattern matching via regular expression | `value: "^[a\|b\|c]"`
`notIRegexp` | negated `iRegexp` | `value: "^[a\|b\|c]"`

**Note:** for now, Zendro only supports the `i` flag for regular expressions, to search case-insensitively via the `iRegexp` / `notIRegexp` operators. Since regular expression syntax varies between storage types to a certain degree, there might be some unexpected edge cases.

##### Comparative operators

Operator | Description | Example
--- | --- | ---
`eq` | equality | `value: 5`
`ne` | inequality | `value: 4`
`gt` | greater than | `value: 6`
`gte` | greater than or equal | `value: 14.2`
`lt` | less than | `value: 10`
`lte` | less than or equal | `value: 20`
`between` | range containment | `value:"6,10" valueType:Array`
`notBetween` | negated `between` | `value:"6,10" valueType:Array`

##### Array operators

Operator | Description | Example
--- | --- | ---
`in` | check if a column matches any value in a list | `value:"a,b,c" valueType:Array`
`notIn` | negated `in` | `value:"a,b,c" valueType:Array`
`contains` | check if a value is contained in an array column | `value:3`
`notContains` | negated `contains` | `value:3`

##### Logical operators

Operator | Description | Example
--- | --- | ---
`or` | logical or, combining multiple searches | `{operator: or search:[ {<search>}, {<search>}]}`
`and` | logical and, combining multiple searches | `{operator: and search:[ {<search>}, {<search>}]}`
`not` | logical not; searches are combined with `and` | `{operator: not search:[ {<search>}, {<search>}]}`

##### StorageType compatibility

| Operator | sql | mongodb | neo4j | cassandra | presto<br>trino | amazonS3 |
| --- | --- | --- | --- | --- | --- | --- |
| `like`        |🟢|🟢<sup>1</sup>|🟢<sup>1</sup>|🔴|🟢|🟢|
| `notLike`     |🟢|🟢<sup>1</sup>|🟢<sup>1</sup>|🔴|🟢|🟢|
| `iLike`       |🟢|🟢<sup>1</sup>|🟢<sup>1</sup>|🔴|🟢<sup>2</sup>|🟢<sup>2</sup>|
| `notILike`    |🟢|🟢<sup>1</sup>|🟢<sup>1</sup>|🔴|🟢<sup>2</sup>|🟢<sup>2</sup>|
| `regexp`      |🟢|🟢|🟢<sup>3</sup>|🔴|🟢|🔴|
| `notRegexp`   |🟢|🟢|🟢<sup>3</sup>|🔴|🟢|🔴|
| `iRegexp`     |🟢|🟢|🟢<sup>3</sup>|🔴|🟢|🔴|
| `notIRegexp`  |🟢|🟢|🟢<sup>3</sup>|🔴|🟢|🔴|
| `eq`          |🟢|🟢|🟢|🟢|🟢|🟢|
| `ne`          |🟢|🟢|🟢|🔴|🟢|🟢|
| `gt`          |🟢|🟢|🟢|🟢|🟢|🟢|
| `gte`         |🟢|🟢|🟢|🟢|🟢|🟢|
| `lt`          |🟢|🟢|🟢|🟢|🟢|🟢|
| `lte`         |🟢|🟢|🟢|🟢|🟢|🟢|
| `between`     |🟢|🔴|🔴|🔴|🟢|🟢|
| `notBetween`  |🟢|🔴|🔴|🔴|🟢|🟢|
| `in`          |🟢|🟢|🟢|🟢|🟢|🟢|
| `notIn`       |🟢|🟢|🟢|🔴|🟢|🟢|
| `contains`    |🟢|🟢|🟢|🟢|🟢|🟢|
| `notContains` |🟢|🟢|🟢|🔴|🟢|🟢|
| `or`          |🟢|🟢|🟢|🔴|🟢|🟢|
| `and`         |🟢|🟢|🟢|🟢|🟢|🟢|
| `not`         |🟢|🟢|🟢|🔴|🟢|🟢|

<sup>1</sup> &nbsp; implemented via `regexp`
<sup>2</sup> &nbsp; implemented via `LOWER(<col>) LIKE LOWER(<value>)`
<sup>3</sup> &nbsp; Neo4j's implementation of [regular expressions](https://neo4j.com/docs/cypher-manual/current/clauses/where/#query-where-regex) expects to match the whole string. Since Zendro tries to unify operator behaviour, a wildcard `.*` is added to the beginning and end of the pattern, except where otherwise specified via `^` and/or `$`.

#### Order argument
The order argument's type also depends on the data model name. For our `Record` data model, the order argument is called `orderRecordInput`, an object containing the name of the attribute to sort by and the order to use (`ASC` or `DESC`). When retrieving a set of records, pass an array of order arguments, one per attribute to sort by. Although the order argument's type depends on the data model name, the argument name is always `order`.

**Example**: to sort the first 100 records alphabetically by name:
```python
query {
  records(order: [ {field: name, order: ASC}], pagination: {limit: 100}) {
    name
    description
  }
}
```

#### Pagination argument
The pagination argument is generic across all data models, controlling the maximum number of records that can be retrieved. For efficiency — especially with big data — the pagination argument is required by the GraphQL schema. Its name is always `pagination`, an object containing the number of records to retrieve and the offset to start from. Zendro provides two types of pagination: standard limit-offset and cursor-based. See [Pagination types]( {% link data-models/resolver-and-model-layers.md %}#pagination-types) for more details.

**Limit-Offset**

attribute | Type  | Description
------ | ------- | --------
limit | Integer | Number of records to retrieve
offset | Integer | Starting point for retrieving records

`limit` is mandatory; `offset` is optional.

**Example**: to retrieve the second 10 records of the `Record` data model:
```python
query {
  records( pagination: {offset: 11, limit: 10}) {
    name
    description
  }
}
```

**Cursor-based**

attribute | Type  | Description
------ | ------- | --------
first | Integer | Number of records to retrieve
after | String | base64 encoded record after which to return records
last | Integer | Number of records to retrieve
before | String | base64 encoded record before which to return records

*Note* that for cursor-based pagination, Zendro follows Facebook's Relay specification for [GraphQL Cursor Connections](https://relay.dev/graphql/connections.htm). Valid combinations are either `first` and optionally `after`, or `last` and optionally `before`.

## Extended API with associations

When a data model is related to one or more data models, extra queries are added to the default API. Let's consider another data model, `Item`, and describe the associations between `Record` and `Item`:

```js
//item.json
{
  "model" : "Item",
  "storageType": "sql",
  "attributes": {
    "name": "String",
    "length": "Int",
    "recordId": "Int"
  },
  "associations": {
    "record": {
      "type": "many_to_one",
      "implementation": "foreignkeys",
      "reverseAssociation": "items",
      "target": "Record",
      "targetKey": "recordId",
      "keyIn": "Item",
      "targetStorageType": "sql"
    }
  }
}
```

```js
//Record.json
{
  ...,
  "associations": {
    "items": {
      "type": "one_to_many",
      "implementation": "foreignkeys",
      "reverseAssociation": "record",
      "target": "Item",
      "targetKey": "recordId",
      "keyIn": "Item",
      "targetStorageType": "sql"
    }
  }
}

```
### Extra query fields to fetch associations

##### Extra query fields for the `Record` model:

* `itemsFilter(search, order, pagination): [Items]` - Given one record, filters all the items associated with it.

* `countFilteredItems(search): Int` - Returns the number of associated items matching the search argument.

 Example:
```python
query {
  records(search: {field: name, value: { value: "%test%"}, operator: like}, pagination: {limit: 100}) {
    name
    description
    countFilteredItems(search: {field: name, value: { value: "%test%"}, operator: like})
    itemsFilter(pagination: {offset: 5, limit: 10}) {
      length
    }
  }
}
```

##### Extra query fields for the `Item` model:

* `record : Record` - Given one item, accesses the data of the record associated with it.

 Example:
```python
query {
  readOneItem(id: 23) {
    name
    length
    record {
      name
      description
    }
  }
}
```

### Extra mutation fields to update or create associations

To manipulate associations, a pair of fields is added to create and update mutations: `addName_of_association` and `removeName_of_association`.

For a `to-one` association, the parameters expect only an `ID` — the associated record to add or remove — or `null` to remove the association.

Continuing the Items-Record example, the sample mutations:

```python
# create
mutation {
  addItem(name: "testItem" addRecord: 14) {
    name
    record {
      name
      description
    }
  }
}
```

```python
# update
mutation {
  updateItem(id: 2 removeRecord: 14) {
    name
    record {
      name
      description
    }
  }
}
```

For a `to-many` association, the parameters expect an array of `IDs` representing the associated records to add or remove.

From the `Record` side of our example:

```python
# create
mutation {
  addRecord( name: "testRecord" addItems: [3, 5, 7] ) {
    name
    itemsFilter {
      name
      length
    }
  }
}
```

```python
# update
mutation {
  updateRecord( id: 1 addItems:[2,4] removeItems: [5,7]) {
    name
    itemsFilter {
      name
      length
    }
  }
}
```
