---
layout: default
title: Customize Zendro
parent: Getting started
nav_order: 3
permalink: /setup_root/customize
---
# Customize Zendro
{: .no_toc }
Zendro offers multiple different ways to customize it to your specific needs.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Graphql server
### Custom validations
It is possible to add custom asynchronous validation functions to validate your data. For more information see:  

[> custom validations]({% link setup_data_scheme.md %}#custom-validator-function-for-ajv)
### Patches
Custom patches allow the user to monkey patch functions and properties of the generated backend code. When running the code generator a skeleton patch file is automatically created for every data-model in `/patches`. Implementing functions there will automatically override default behaviour. An example patch to override `<function_name>` could look something like this:

```
data.prototype.<function_name> = function(...) {...}
```

## Single page application
### Custom Pages

Routes in the application automatically mirror the structure of the pages folder. The default static site contains a dynamic `[model]` route, with a home page ([`index.tsx`](./src/pages/[model]/index.tsx)) to display an interactive table of records, and one child route ([`item.tsx`](./src/pages/[model]/item.tsx)) to display data for a single record.

Overriding a model route with a custom page only requires to provide an appropriately named file within the pages folder. Because in Next.js predefined routes take precedence over dynamic routes, all requests for that model will now point to the new page.

In the example below, a custom `books.tsx` page is overriding the default `/books` route that would be otherwise provided by `[model]/index.tsx`.

```
pages
├── [model]
│   ├── index.tsx
│   └── [item].tsx
├── books.tsx
├── index.tsx
└── login.tsx
```

### Next.js Resources

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

## Add a new storage type
If current storage types cannot fulfill user' requirements, it is possible to add a new storage type in Zendro. Here is the guidance for that.

### Workflow
1. Define some new **data models** according to the specification: [Data Models]({% link setup_data_scheme.md%}). If user would like to introduce associations among data models, new data model definitions should also include definitions of associations. For distributed models and their adapters, please read this [specification]({% link ddm.md %}). 
2. Modify the `storageType` in new data model definitions. In specifically, according to the kinds of accesses in the new storage type, user can choose one similar existing `storageType` and replace the new `storageType` with that (replace `targetStorageType` for defined associations if necessary). For example, if the current storage type supports both read and write accesses, user can choose `mongodb` as `storageType`. And in templates of `mongodb` all APIs would be implemented. However, if the new storage type only supports the read access, then an existing storageType `trino` could be used. In this situation, respective APIs would be disabled. For instance, if user calls APIs related to write access in `trino`, error information like **'Not supported by Trino'** would be thrown. 
3. Create a sandbox with [zendro CLI](https://github.com/Zendro-dev/zendro#a-running-example). Please following the steps 1-3 in the CLI page.
4. Add [connection](#connection) in Zendro by the node driver of the new storage type.
5. Design the [**basic CRUD** operations](#basic-crud-operations) for a single record and [operations for **associations**](#operations-for-associations) between records in generated templates of existing `storageType`. 
6. User can extend `single-page-app` for the new storage type as in this [section](#extension-in-single-page-application).
7. (optional) If user would like to submit a pull request for the new storage type to the main branch of Zendro, the modified templates should be translated into [EJS templates](#ejs-templates-in-code-generator) in `graphql-server-model-codegen`. Besides, corresponding [tests](#test) in backend should also be added.

How to add connection, APIs, EJS templates, SPA extension and tests in detail is described below:
### Connection
1. In `connection.js` file from graphql-server repository user can introduce the corresponding node driver and implement a setup function for the new storage type. Afterwards, user can extend `addConnectionInstances` method with the implemented setup function. Besides, user need extend `checkConnections` function, which is used for checking the connectivity for all storage types. 
2. In `models/index.js` user can add the new storage type object in the object definition of `models`. And this object is used for storing models according to different storage types. Moreover, user should create a new folder under `models` for new models in the new storage type and move all new models into the new folder. And it is necessary to correct the `storageType` in the object `definition` of each new model, because these models are generated based on other existing storage types. In addition, if there are defined associations, `targetStorageType` should also be updated to the new storage type. 
3. If user would like to use distributed models and adapters, it is necessary to modify the `index.js` file in the `models/adapters` folder. Similarly, user need add the new storage type object in the object definition of `adapters`. Besides, user need correct the `storageType` in the object `definition` of each new adapter, because these adapters are generated based on other existing storage types. 
4. In `utils/helper.js` user need extend functions `initializeStorageHandlersForModels` and `initializeStorageHandlersForAdapters`. By these two functions, the connected node driver, namely storage handler in Zendro context, would be assigned to corresponding models or adapters. Moreover, it is recommended to extend the `createIndexes` function. By indexes, the speed of fetching/writing records would be greatly improved.
5. Execute `zendro start gqs` for local databases. Or execute `zendro dockerize -u` for dockerizing Zendro APP. Because currently user only needs to test the backend, user can comment `zendro-spa` service and add container information for new storage type in the `docker-compose-dev.yml` file. If there is no error in `./logs/graphql-server.log` (local databases) or in the console (dockerized App), then the connection between models/adapters and node driver is successful.
6. Create a new branch for the new storage type in graphql-server repository, then only commit changes for connection. Note, please do not commit model files, adapter files, schema files, validation files and patch files. Because these files are generated from code generators.

### Basic CRUD operations

#### Schema
In Zendro, different storage types have various sets of operators for search. In specifically, there are four existing schemas of operators, namely `GenericPrestoSqlOperator`, `MongodbNeo4jOperator`, `CassandraOperator` and `AmazonS3Operator`. Meanwhile, considering [StorageType compatability]({% link api_graphql.md%}#storagetype-compatability), user can choose one similar operator schema and substitute that into `operator` field in new schema files, which are stored in the folder `schemas`. And it is also possible to add your own set of search operators.

#### Resolver & Model
In general, it is unnecessary to modify new resolvers in `resolvers` folder. Besides, user need modify some relevant methods in new model files of `models/new_storage_type` folder. 

In specifically, according to the kinds of accesses, APIs are classified into two categories, namely read access and write access. Moreover, two brief lists of APIs in different categories are in the following. And corresponding methods in the model level should be implemented.

* Read access

| API in resolver level| method in model level | note|
| ------------- |:-------------:|:-------------:|
| readOne`<model>` | readById | [data loader implementation]({% link setup_data_scheme.md%}#data-loader)|
| count`<models>` | countRecords ||  
| `<models>` | readAll | limit-offset based pagination |
| `<models>`Connection | readAllCursor | cursor based pagination |

* Write access

| method in resolver level| method in model level |
| ------------- |:-------------:|
| add`<model>` | addOne | 
| delete`<model>` | deleteOne |  
| update`<model>` | updateOne | 
| bulkAdd`<model>`Csv | bulkAddCsv |

To implement most methods related to read access, it is recomended to familiarize pagination, order and search arguments in Zendro. And the relevant link is [here]({% link api_graphql.md%}#general-filter-arguments). Afterwards, it is necessary to add some helper functions for parsing these arguments into statements or objects used in the node driver for the new storage type. For example, the method `searchConditionsToMongoDb` in `readAllCursor` method for `mongodb` storageType would parse the search arguments into a object which can be accepted by the MongoDB node driver. Similarly, `orderConditionsToMongoDb` and `cursorPaginationArgumentsToMongoDb` methods would parse order and pagination arguments into respective objects for query. And by these parsed objects, MongoDB node driver can fetch corresponding records from database and send them back to Zendro. 

#### Validation
In `validations` folder there are some files for validating records in new data models. By default, user can keep existing validation files. If user would like to add more validation rules, the guidance is in this [link]({% link setup_data_scheme.md%}#custom-validator-function-for-ajv).

### Operations for associations
When user would like to introduce operations for associations among data models, some functions in the model level should be implemented according to the type of association and the new storage type. In specifically, according to the type of associations, there are four lists about association resolvers and corresponding methods in the model level.

#### One to one
If foreign key stores in current model, modify corresponding methods in the model level:

| method in resolver level| method in model level |
| ------------- |:-------------:|
|`<model>`.prototype.add_unique_`<assoc>`|add_`<assocID>`| 
|`<model>`.prototype.remove_unique_`<assoc>`| remove_`<assocID>`|  
|bulkAssociate`<model>`With`<assocID>` | bulkAssociate`<model>`With`<assocID>`| 
|bulkDisAssociate`<model>`With`<assocID>`| bulkDisAssociate`<model>`With`<assocID>`|

Other relevant methods in corresponding associated models would be automatically generated.

#### Many to one / One to many
It is necessary to implement the following methods in the model level for models with `many_to_one` association type:

| method in resolver level| method in model level |
| ------------- |:-------------:|
|`<model>`.prototype.add_`<assoc>`|add_`<assocID>`| 
|`<model>`.prototype.remove_`<assoc>`|remove_`<assocID>`|  
|bulkAssociate`<model>`With`<assocID>`|bulkAssociate`<model>`With`<assocID>`| 
|bulkDisAssociate`<model>`With`<assocID>`|bulkDisAssociate`<model>`With`<assocID>`|

For models with `one_to_many` association type, relevant methods would be automatically generated. 

#### Many to many
For all models with `many_to_many` association type, the following methods in the model level should be implemented:

| method in resolver level| method in model level |
| ------------- |:-------------:|
|`<model>`.prototype.add_`<assoc>`|add_`<assocIDs>`| 
|`<model>`.prototype.remove_`<assoc>`|remove_`<assocIDs>`|

### Distributed models and adapters
Firstly, code for distributed models and adapters with `ddm-adapter` storageType would be automatically generated. Then user needs implement methods for adapters with new storage type, e.g. `mongodb-adapter`. And the path for adapters is `models/adapters`. In addition, the methods should be implemented for adapters and models are approximately the same. Hence, the lists of models' methods in [Basic CRUD operations](#basic-crud-operations) and [Operations for associations](#operations-for-associations) could also be the reference for new adapters. And only `readAll` method would not be implemented for adapters. Because Zendro only supports cursor based pagination for adapters, namely `readAllCursor` method.

### Extension in single page application
1. add new storage type in type `StorageType` from `src/types/models.ts` file, e.g. `mongodb` and `mongodb-adapter`
2. add configuration for new storage type in `getModelApiPrivileges()` function from `src/utils/models.ts` file. In specifically, the configuration would include the following options:
  * operators ([String]): allowed operators in new storage type.
  * textSearch (boolean): enable searchbar. By default, Zendro would use `ilike` operator for search. If your storage type does not support `ilike`, please disable testSearch.
  * backwardPagination (boolean): enable backward pagination. 
  * sort (boolean): enable sorting by columns
  * create (boolean): enable function for creating a record
  * update (boolean): enable function for updating a record
  * delete (boolean): enable function for deleting a record
  * bulkAddCsv (boolean): enable function for bulk adding records from CSV file
3. test the extended SPA. For local databases, user can set up SPA by `zendro start spa`. If user use dockerized App, it is necessary to shut down current dockerized App by `zendro dockerize -d`. Then user needs to uncomment `zendro_spa` in the `docker-compose-dev.yml` file and dockerize Zendro App by `zendro dockerize -u`. If the SPA works well, user can commit the changes made in Step 1 and Step 2 in a new branch.

### EJS templates in code generator
If user would like to generate code from new data model definitions, it is recommended to implement some EJS templates in `graphql-server-model-codegen`. And the guidance is as the following:

1. create a template file named `create-models-<storageType>` in `views` folder, e.g. `create-models-mongodb`
2. migrate basic CRUD methods from modified model files into `create-models-<storageType>`. Namely, replace the concrete model name and other model specific variables into EJS variables. Please consider the common EJS variables defined in `getOptions` method from `funks.js` file. In addition, `create-models-mongodb` could be an example of the concrete implementation.
3. (optional) if in the new storage type user implements methods for associations, user should create a template file named `create-models-fieldMutations-<storageType>` in `views/includes` folder, e.g. `create-models-fieldMutations-mongodb`.
4. (optional) migrate basic association related methods from modified model files into `create-models-fieldMutations-<storageType>`. In specifically, basic association related methods are the methods for associations without bulk operation. Similarly, common EJS variables are also defined in `getOptions` method from `funks.js` file. And the example template would be `create-models-fieldMutations-mongodb`.
5. (optional) extend template `views/includes/bulkAssociations-models` with bulk operation methods for associations from modified model files. 
6. (optional) if user implements adapters in the new storage type, it is necessary to create `create-<storageType>-adapter` template, e.g. `create-mongodb-adapter`. 
7. (optional) migrate basic CRUD methods from modified adapter files into `create-<storageType>-adapter`.
8. (optional) if user implements methods for associations in adapters, it is necessary to migrate these methods in `create-adapter-fields-mutations` and `bulkAssociations-models` templates.
9. update the `lib/generators-aux.js` file. In this file, user can add the default database key for the new storage type.
10. update the `lib/operators-aux.js` file. In this file, user can specify the operator schema in the new storage type.
11. update the `funks.js` file. By this file code generator could match the storage types with corresponding templates. Hence, user needs add code for the new storage type. For example, user can search `mongodb` to locate correct locations for adding  code. And the code to be implemented for new storage type should be similar as the matched code for `mongodb` storage type.

### Test
If user would like to contribute to Zendro main branch, it is necessary to commit new templates and add tests before submitting pull requests in Zendro repositories.

The test frameworks for backend are `mocha` and `chai`.
#### Unit test  
For Zendro, unit tests mean that code generater can generate expected code according to the given templates. Hence, it is necessary to add data model definitions to be tested in `graphql-server-model-codegen/test/unit_test_misc` folder and expected code in `graphql-server-model-codegen/test/unit_test_misc/test-describe` folder. Apart from that, user needs add test cases for new storage type in file `graphql-server-model-codegen/test/mocha_unit.test`. And user can execute unit tests by the command `npm run test-unit`. In addition, the required unit tests in the model level would be the following:
* __Basic CRUD Methods__: readById, countRecords, readAll,  readAllCursor, addOne, deleteOne, updateOne, bulkAddCsv
* __Association Methods__:
  * OneToOne: add one association, remove one association, bulkAssociation, bulkDisassociation
  * ManyToMany: add associations, remove associations
  * ManyToOne: add one association, remove one association, bulkAssociation, bulkDisassociation

#### Integration test  
In integration tests functionality of APIs would be tested. Before the test, data model definitions should be added in the folder `test/integration_test_misc/integration_test_models_instance2`. Besides, user needs add the container configuration for new storage type in `test/integration_test_misc/docker-compose-test.yml` file, because the setup of integration tests is based on the `docker` and `docker-compose`. In addition, user needs update `test/integration_test_misc/data_models_storage_config2.json` file, which is the configuration file for default connection, for new storage type. Moreover, if the `bulkAddCsv` method would be tested, the corrsponding CSV files should be added in the folder `test/integration_test_misc`. Afterwards user needs create a new integration test file as `test/mocha_integration_<storageType>.test.js`, e.g. `mocha_integration_mongodb.test.js`. Besides, user needs update the `test/testenv_cli` file. Namely, user needs add the new integration file into the CLI file. For example, user can search `mocha_integration_mongodb.test.js` in the CLI file and add new integration file below the same `if` condition. To run the integration-test suite, user can execute the command `npm run test-integration [-- OPTIONS]`. To view the different integration-test commands and some examples, user can execute `npm run test-integration -- -h`. In addition, the list of concrete tests would be in the following:
* __Basic CRUD Operations__: add operation, update operation, readOne operation, delete operation, CSV bulkUpload functionality, limit-offset pagination, cursor-based pagination, sort functionality, table template acquisition, data loader functionality, search functionality
* __Operators__: like , notLike, iLike, notILike, regexp, notRegexp, iRegexp, notIRegexp, in, notIn, contains, notContains
* __Association__:
  * __ManyToOne__: add one association, read one associated record, remove one association, bulkAssociation, bulkDisassociation
  * __ManyToMany__: add associations, read one associated record, remove associations
  * __OneToOne__: add one association, read one associated record, remove one association, bulkAssociation, bulkDisassociation

Note: If new storage type does not have write access or associations, it is unnecessary to add relevant tests. Apart from that, if user adds the distributed models and adapters, the similar tests should also be added. And tests for `readAll` method in adapters should not be added, because only cursor based pagination would be implementd in adapters, namely `readAllCursor` method.