[ &larr; back](api_root.md)
<br/>
Concrete requests that can be send to the backend server are model dependent. Therefore from now on let's assume that our first data model is called `record` and is described as follows:

```
//Record.json
{
  "model": "record",
  "storageType": "Sql",
  "attributes": {
    name: String,
    description: String
  }
}
```

### GraphQL Queries
* `records(search, order, pagination) : [Records]` - Check user authorization and return certain number of records, specified in pagination argument, that holds the condition of search argument, all of them sorted as specified by the order argument. For more information about `search`, `order` and `pagination` argument see this [section below](#general-filter-arguments). Example:
```
query{
  records(search: {field: name, value:{ value: "%test%"}, operator: like}, order: [{field: name, order: ASC}], pagination: {limit:10}){
    name
    description
  }
}
```

* `readOneRecord(id): Record` - Check user authorization and return one record with the specified id in the id argument. Example:
```
query {
  readOneRecord(id: 23){
    name
    description
  }
}
```

* `countRecords(search): Integer` - Count number of records that holds the conditions specified in the search argument. Example:
```
query{
  countRecords( search: {field: name, value:{ value: "%test%"}, operator: like} )
}
```

* `vueTableRecord: vueTableRecord`  - Return table of records as needed for displaying a vuejs table. Example:
```
query{
  vueTableRecord{
    data{
      name
      description
    }
  }
}
```
### Mutations

* `addRecord(record): Record` - Checks user authorization and creates a new record with data specified in the record argument. Example:
```
  mutation{
    addRecord(name: "testRecord", description: "testing record" ){
      name
      description
    }
  }
```

* `deleteRecord(id): String` - Check user authorization and delete a record with the specified id in the id argument. Example:
```
mutation{
  deleteRecord(id: 23)
}
```

* `updateRecord(record): Record` - Check user authorization and update the record specified in the input argument. Example:
```
mutation{
  updateRecord(id: 23 name: "updated name"){
    name
    description
  }
}
```

* `bulkAddRecordCsv: String` - Load csv file of records. 
In this mutation the csv file should be attached in the request.


### General Filter Arguments
When retrieving a set of records of any data model, there are specific arguments that can help to select only certain records. Two of the general arguments that the user can specify as input are pagination and order. The description about how to use these arguments is as follows:

#### Search argument
This argument type depends on the data model name. Assuming our data model is called `Record` then the graphql type of this argument is called
`searchRecordInput` and it is an object which contains the next fields:

name | Type | Description
------- | ------- | --------------
*field* | String | Can be any record's attributes name. Can be also understood as the column by which the records will be filtered.
*value* | Object | Value used to filter the records, can be type `String` or type `Array` (default type is String) and the actual value should be also specified. Example: `value:{ type: String, value: "%string_to_filter%"}`
*valueType* | enum | One of `Array, String, Int, Float, Boolean, DateTime`
*operator* | String | Operator used to filter the records. Example: `eq`, `like` ...
*search* | [searchRecordInput] | Recursively the user can spefify another search argument.

Although the search argument type depends on the data model name, the argument name will be always the same, _search_.

**EXAMPLE** : Let's say we want to filter the first 100 records which name has the substring *'test'*. The proper query to perform this action would be:

```
query {
  records(search: {field: name, value: "%test%", operator: like}, pagination: {limit: 100}){
    name
    description
  }
}

```

#### Operators
Zendro supports the following list of operators. Depending on the storage type of the model some operators are not supported and hence not exposed in the models schema.

##### Patern matching operators

Operator | Description | Example
--- | --- | ---
`like` | pattern matching with wildcards for the entire string | `value: "%abc_"`
`notLike` | negated `like` | `value: "%abc%"`
`iLike` | case insensitive pattern matching with wildcards for the entire string | `value: "%abc_"`
`notILike` | negated `iLike` | `value: "%abc%"`
`regexp` | pattern matching via regular expression | `value: "^[a\|b\|c]"` 
`notRegexp` | negated `regexp` | `value: "^[a\|b\|c]
`iRegexp` | case insensitive pattern matching via regular expression | `value: "^[a\|b\|c]"`
`notIRegexp` | negated `iRegexp` | `value: "^[a\|b\|c]"`

**Note:** For now Zendro only supports the `i` flag for regular expressions to search case-insesitive via the `iRegexp` / `notIRegexp` operators. Since the syntax for regular expression varies between storagetypes to a certain degree, there might be some unexpected edge cases.

##### Comparative operators

Operator | Description | Example
--- | --- | ---
`eq` | equality | `value: 5`
`ne` | unequality | `value: 4`
`gt` | greater than | `value: 6`
`gte` | greater than equal | `value: 14.2`
`lt` | less than | `value: 10`
`lte` | less than equal | `value: 20`
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
`or` | logical or to combine multiple searches | `{operator: or search:[{<search>}, {<search>}]}`
`and` | logical and to combine multiple searches | `{operator: and search:[{<search>}, {<search>}]}`
`not` | logical not. searches will get combined with `and` | `{operator: not search:[{<search>}, {<search>}]}`

##### StorageType compatability

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
<sup>3</sup> &nbsp; neo4j's implementation of [regular expressions](https://neo4j.com/docs/cypher-manual/current/clauses/where/#query-where-regex) expects to match the whole string. Since Zendro tries to unify the behaviour of the operators a wildcard `.*` is added to the beginning and the end of the pattern except when otherwise specified via `^` and/or `$`.
#### Order argument
The order argument type also depends on the data model name. With our data model `Record` the order argument will be called `orderRecordInput` and it is an object  which contains the name of the attribute to sort and the order that will be used, order can be ascendent `ASC` or descendant `DESC`.
When retrieving a set of records the user passes an array or order arguments, one for each attribute that will be sorted.
Although the order argument type depends on the data model name, the argument name will be always the same, _order_.

EXAMPLE: Let's say we want to sort the first 100 records alphabetically by the name column. The proper query to perform this action would be:
```
query{
  records(order: [{field: name, order: ASC}], pagination: {limit: 100}){
    name
    description
  }
}
```

#### Pagination argument
The pagination argument is generic for all data model and the purpose of this argument is to control the maximum number of records that can be retrieved. Due to efficiency, especially in the realm of big data, the pagination argument is required by the graphql schema. The name for the argument is `pagination` and it is an object which contains the number of records to retrieve and the offset from where to start counting the records. Zendro provides two different types of pagination. Standard limit-offset based and cursor-based pagination. See Section "Pagination types" [here](setup_data_scheme.md#pagination_types) for more details.

**Limit-Offset**

attribute | Type  | Description
------ | ------- | --------
limit | Integer | Number of records to retrieve
offset | Integer | Starting point for retrieving records

`limit` is a mandatory argument, offset is optional.

EXAMPLE: Considering the `Record` data model for retrieving the second 10 records, the proper query to perform this action would be:
```
query{
  records( pagination: {offset: 11, limit: 10}){
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

*Note* that for cursor-based pagination we follow facebooks relay specification on [GraphQL Cursor Connections](https://relay.dev/graphql/connections.htm).  
Either `first` and optionally `after`, or `last` and optionally `before` are valid combinations. 

## Extendend API with associations

When a data model is related with one or more data models, extra queries are added to the default API.
Following with our example, let's consider another data model `Item`:
And we will describe the associations between the models `Record` and `Item`.

```
//item.json
{
    "model" : "Item",
    "storageType": "sql",
    "attributes": {
      "name": String,
      "length": Int,
      "recordId":Int
    },
    "associations":{
      "record":{
        "type": "many_to_one",
        "implementation": "foreignkey",
        "reverseAssociation": "items",
        "target": "Record",
        "targetKey": "recordId",
        "keyIn": "Item",
        "targetStorageType": "sql",
      }
    }
}
```

```
//Record.json
{
...
  "associations":{
    "items": {
      "type": "one_to_many",
      "implementation": "foreignkey",
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

##### The extra query fields for the `Record` model would be:

* `itemsFilter(search, order, pagination): [Items]` - Given one record, the user will be able to filter all the items associated with the current record.

* `countFilteredItems(search): Int` - Return the number of associated items which hold the search argument conditions.

 Example:
```
query{
  records(search: {field: name, value:{ value: "%test%"}, operator: like}, pagination: {limit: 100}){
    name
    description
    countFilteredItems(search: {field: name, value:{ value: "%test%"}, operator: like})
    itemsFilter(pagination:{offset: 5, limit: 10}){
      length
    }
  }
}
```

##### The extra query fields for the `Item` model would be:

* `record : Record` -  Given one item, the user will be able to access the data of the record associated with the current item.

 Example:
```
readOneItem(id: 23){
  name
  length
  record{
    name
    description
  }
}
```

### Extra mutation fields to update or create associations.

In order to manipulate associations, a couple of fields to create and update mutations will be added: `addName_of_association` and `removeName_of association`

In the case of `to-one` association, the parameters expect to receive only an `ID`, representing the associated item to be added or removed,  or `null` value(to remove association).

Continuing with our example of Items-Record, the sample mutations would be:

```
//create
mutation{
  addItem(name: "testItem" addRecord: 14){
    name
    record{
      name
      description
    }
  }
}
```

```
//update
mutation{
  updateItem(id: 2 removeRecord: 14){
    name
    record{
      name
      description
    }
  }
}
```

In the case of `to-many` association the parameters expect to receive an array of `IDs` representing the associated items to be added or removed.

From the side of the `Record` model from our example, the sample mutations would be:

```
//create
mutation{
  addRecord( name: "testRecord" addItems: [3, 5, 7] ){
    name
    itemsFilter{
      name
      length
    }
  }
}
```

```
//update
mutation{
  updateRecord( id: 1 addItems:[2,4] removeItems: [5,7]){
    name
    itemsFilter{
      name
      length
    }
  }
}
```
