[ &larr; back](api_root.md)
<br/>
Concrete requests that can be sent to backend server are model dependent, therefore from now on let's assume that our first data model is called `record` and is described as follows:

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
* `records(search, order, pagination) : [Records]` - Check user authorization and return certain number, specified in pagination argument, of records that holds the condition of search argument, all of them sorted as specified by the order argument. For more information about `search`, `order` and `pagination` argument see this [section below](#general-filter-arguments). Example:
```
query{
  records(search: {field: name, value:{ value: "%test%"}, operator: like}, order: [{field: name, order: ASC}]){
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

* `addRecord(record): Record` - Check user authorization and creates a new record with data specified in the record argument. Example:
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

* `bulkAddRecordCsv: String` - Load csv file of records
In this mutation the csv file should be attached in the request.


### General Filter Arguments
When retrieving a set of records of any data model, there is specific arguments that can help to select only
certain records. Two of the general arguments that the user can specify as input are pagination and order. The description about how to use these arguments is as follows:

#### Search argument
This argument type depends on the data model name. Assuming our data model is calle `Record` then the graphql type of this argument is called
`searchRecordInput` and it is an object which contains the next fields:

name | Type | Description
------- | ------- | --------------
*field* | String | Can be any record's attribute name. Can be also understood as the column by which the records will be filtered.
*value* | Object | Value used to filter the records, can be type `String` or type `Array` (default type is String) and the actual value should be also specified. Example: `value:{ type: String, value: "%string_to_filter%"}`
*operator* | String | Operator used to filter the records. Example: `eq`, `like` ...
*search* | [searchRecordInput] | Recursively the user can spefify another search argument.

Although the search argument type depends on the data model name, the argument name will be always the same, _search_.

EXAMPLE : Let's say we want to filter all the records which name has the substring *'test'*. The proper query to perform this action would be:

```
query {
  records(search: {field: name, value:{ value: "%test%"}, operator: like}){
    name
    description
  }
}

```
#### Order argument
The order argument type also depends on the data model name. With our data model `Record` the order argument will be called `orderRecordInput` and it is an object  which contains the name of the attribute to sort and the order that will be used, order can be ascendent `ASC` or descendant `DESC`.
When retrieving a set of records the user pass an array or order arguments, one for each attribute that will be sorted.
Although the order argument type depends on the data model name, the argument name will be always the same, _order_.

EXAMPLE: Let's say we want to sort all records alphabetically by the name column. The proper query to perform this action would be:
```
query{
  records(order: [{field: name, order: ASC}]){
    name
    description
  }
}
```

#### Pagination argument
The pagination argument is generic for all data model and the purpose of this argument is to control the maximum number of records that can be retrieved. The name for the argument is `pagination` and it is an object which contains the number of records to retrieve and the offset from where to start counting the records.

attribute | Type  | Description
------ | ------- | --------
limit | Integer | Number of records to retrieve
offset | Integer | Starting point for retrieving records

EXAMPLE: Considering the `Record` data model for retrievin the second 10 records, the proper query to perfrom this action would be:
```
query{
  records( pagination: {offset: 11, limit: 10}){
    name
    description
  }
}
```

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
        "type": "to_one",
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
      "type": "to_many",
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

* `countFilteredItems(search): Int` - Return the number of associated items which holds the search argument coditions.

 Example:
```
query{
  records(search: {field: name, value:{ value: "%test%"}, operator: like}){
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

In order to manipulate associations, a couple of fields in create and update mutations will be added: `addName_of_association` and `removeName_of association`

In the case of `to-one` association the parameters expect to receive only and `ID`, representing the associated item to be added or removed,  or `null` value(to remove association).

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
