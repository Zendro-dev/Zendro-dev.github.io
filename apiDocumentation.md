## CRUD API documentation

Given a data model described within [this specifications and format](dataModels.md), the ScienceDb backend generator will
provide a default CRUD API which can be accessed through the graphql query language.
For more information about queries and mutations in graphql go to this [documentation](https://graphql.org/learn/queries/)


The name of the queries and mutations that will be created depends on the name of the data model. From now on let's assume our data model is called `Record` and it is described as follows:
```
  //Record.json
{
  "model": "Record",
  "storageType": "Sql",
  "attributes": {
    name: String,
    description: String
  }
}
```
Then the correspondant CRUD operations and its retuning values that are automatically created are:

### Queries
* `records(search, order, pagination) : [Records]` - Check user authorization and return certain number, specified in pagination argument, of records that holds the condition of search argument, all of them sorted as specified by the order argument. For more information about `search`, `order` and `pagination` argument see this [section below](#general-filter-arguments).

Example:
```
query{
  records(searchRecordInput: {field: name, value:{ value: "%test%"}, operator: like}, order: [{field: name, order: ASC}]){
    name
    description
  }
}
```

* `readOneRecord(id): Record` - Check user authorization and return one record with the specified id in the id argument.
```
query {
  readOneRecord(id: 23){
    name
    description
  }
}
```

* `countRecords(search): Integer` - Count number of records that holds the conditions specified in the search argument
```
query{
  countRecords( searchRecordInput: {field: name, value:{ value: "%test%"}, operator: like} )
}
```

* `vueTableRecord: vueTableRecord`  - Return table of records as needed for displaying a vuejs table
query{
  vueTableRecord{
    data{
      name
      description
    }
  }
}

### Mutations

* `addRecord(record): Record` - Check user authorization and creates a new record with data specified in the record argument
```
  mutation{
    addRecord(name: "testRecord", description: "testing record" ){
      name
      description
    }
  }
```

* `deleteRecord(id): String` - Check user authorization and delete a record with the specified id in the id argument.
```
mutation{
  deleteRecord(id: 23)
}
```

* `updateRecord(record): Record` - Check user authorization and update the record specified in the input argument
```
mutation{
  updateRecord(id: 23 name: "updated name"){
    name
    description
  }
}
```

* `bulkAddRecordCsv: String` - Load csv file of records
In this mutation the csv file should be attached in the request.


### General Filter Arguments
When retrieving a set of records of any data model, there is specific arguments that can help to select only
certain records. Two of the general arguments that the user can specify as input are pagination and order. The description about how to use these arguments is as follows:

#### Search argument
This argument name depends also on the data model name. Assuming our data model is calle `Record` then the search arguments is called
`searchRecordInput` and it is an object which contains the next fields:

name | Type | Description
------- | ------- | --------------
*field* | String | Can be any record's attribute name. Can be also understood as the column by which the records will be filtered.
*value* | Object | Value used to filter the records, can be type `String` or type `Array` (default type is String) and the actual value should be also specified. Example: `value:{ type: String, value: "%string_to_filter%"}`
*operator* | String | Operator used to filter the records. Example: `eq`, `like` ...
*search* | [searchRecordInput] | Recursively the user can spefify another search argument.

EXAMPLE : Let's say we want to filter all the records which name has the substring *'test'*. The proper query to perform this action would be:

```
query {
  records(searchRecordInput: {field: name, value:{ value: "%test%"}, operator: like}){
    name
    description
  }
}

```
#### Order
The order argument name also depends on the data model name. With our data model `Record` the order argument will be called `orderRecordInput` and it is an object  which contains the name of the attribute to sort and the order that will be used, order can be ascendent `ASC` or descendant `DESC`.
When retrieving a set of records the user pass an array or order arguments, one for each attribute that will be sorted.

EXAMPLE: Let's say we want to sort all records alphabetically by the name column. The proper query to perform this action would be:
```
query{
  records(order: [{field: name, order: ASC}]){
    name
    description
  }
}
```

#### Pagination
The pagination argument is generic for all data model and the purpose of this argument is to control the maximum number of records that can be retrieved. The name for the argument is `painationInput` and it is an object which contains the number of records to retrieve and the offset from where to start counting the records.

attribute | Type  | Description
------ | ------- | --------
limit | Integer | Number of records to retrieve
offset | Integer | Starting point for retrieving records

EXAMPLE: Considering the `Record` data model for retrievin the second 10 records, the proper query to perfrom this action would be:
```
query{
  records( paginatioInput: {offset: 11, limit: 10}){
    name
    description
  }
}
```
