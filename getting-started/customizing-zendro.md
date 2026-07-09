---
layout: default
title: Customize Zendro
parent: Getting started
nav_order: 3
permalink: /getting-started/customize
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
## GraphQL server
### Custom validations
It is possible to add custom asynchronous validation functions to validate your data. For more information see:

[> custom validations]({% link data-models/validation.md %})

### Patches
Custom patches allow you to monkey-patch functions and properties of the generated backend code. When running the code generator, a skeleton patch file is automatically created for every data model in `/patches`. Implementing functions there automatically overrides the default behaviour. An example patch overriding `<function_name>` could look like this:

```
data.prototype.<function_name> = function(...) {...}
```

## Single page application
### Custom pages

Routes in the application automatically mirror the structure of the pages folder. The default static site contains a dynamic `[model]` route, with a home page (`index.tsx`) to display an interactive table of records, and one child route (`item.tsx`) to display data for a single record.

Overriding a model route with a custom page only requires providing an appropriately named file within the pages folder. Because in Next.js predefined routes take precedence over dynamic routes, all requests for that model will now point to the new page.

In the example below, a custom `books.tsx` page overrides the default `/books` route that would otherwise be provided by `[model]/index.tsx`.

```
pages
├── [model]
│   ├── index.tsx
│   └── [item].tsx
├── books.tsx
├── index.tsx
└── login.tsx
```

### Next.js resources

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

## Add a new storage type
If none of the current storage types fulfill your requirements, it is possible to add a new one to Zendro.

### Workflow
1. Define some new **data models** according to the [Data models]({% link data-models/index.md %}) specification. If you would like to introduce associations among data models, the new data model definitions should also include the association definitions. For distributed models and their adapters, see the [Distributed data models]({% link distributed-data-models.md %}) specification.
2. Modify the `storageType` in the new data model definitions. Depending on the kind of access supported by the new storage type, choose one similar existing `storageType` and use that as a starting point (and update `targetStorageType` for defined associations if necessary). For example, if the new storage type supports both read and write access, `mongodb` is a good match, since all its APIs are implemented. If the new storage type only supports read access, `trino` is a better fit — its write-access APIs are disabled and throw an error like *"Not supported by Trino"* when called.
3. Create a sandbox with the [Zendro CLI](https://github.com/Zendro-dev/zendro#a-running-example). Follow steps 1-3 on that page.
4. Add a [connection](#connection) in Zendro using the node driver of the new storage type.
5. Design the [basic CRUD operations](#basic-crud-operations) for a single record, and the [operations for associations](#operations-for-associations) between records, in the generated templates for the existing `storageType` you started from.
6. Extend the `single-page-app` for the new storage type as described in [Extension in single page application](#extension-in-single-page-application).
7. (optional) If you would like to submit a pull request for the new storage type to the main branch of Zendro, translate the modified templates into [EJS templates](#ejs-templates-in-code-generator) in `graphql-server-model-codegen`, and add the corresponding [tests](#test) to the backend.

How to add the connection, APIs, EJS templates, SPA extension and tests in detail is described below.

### Connection
1. In the `connection.js` file of the graphql-server repository, introduce the corresponding node driver and implement a setup function for the new storage type. Then extend the `addConnectionInstances` method with the implemented setup function. Also extend the `checkConnections` function, which checks connectivity for all storage types.
2. In `models/index.js`, add the new storage type object to the `models` object definition, which stores models grouped by storage type. Create a new folder under `models` for the new models and move them there. Update the `storageType` in the `definition` object of each new model (they were generated based on an existing storage type), and, if there are associations, update `targetStorageType` as well.
3. If you would like to use distributed models and adapters, modify the `index.js` file in the `models/adapters` folder similarly: add the new storage type object to the `adapters` definition, and correct the `storageType` of each new adapter.
4. In `utils/helper.js`, extend the `initializeStorageHandlersForModels` and `initializeStorageHandlersForAdapters` functions. These assign the connected node driver (the storage handler, in Zendro's terms) to the corresponding models or adapters. It is also recommended to extend the `createIndexes` function, which greatly improves read/write speed.
5. Execute `zendro start gqs` for local databases, or `zendro dockerize -u` to dockerize the app. Since you only need to test the backend at this point, you can comment out the `zendro-spa` service and add the container information for the new storage type in `docker-compose-dev.yml`. If there are no errors in `./logs/graphql-server.log` (local databases) or the console (dockerized app), the connection between models/adapters and the node driver is working.
6. Create a new branch for the new storage type in the graphql-server repository, and only commit changes related to the connection. Do not commit model files, adapter files, schema files, validation files or patch files — those are generated by the code generator.

### Basic CRUD operations

#### Schema
Zendro has four existing schemas of operators, one per family of storage types: `GenericPrestoSqlOperator`, `MongodbNeo4jOperator`, `CassandraOperator` and `AmazonS3Operator`. Considering [StorageType compatibility]({% link api/graphql-reference.md %}#storagetype-compatibility), choose the operator schema most similar to your new storage type and use it as the `operator` field in the new schema files (stored in the `schemas` folder). You can also define your own set of search operators.

#### Resolver & Model
In general it is unnecessary to modify the resolvers in the `resolvers` folder. You do need to modify some methods in the new model files under `models/new_storage_type`.

APIs are classified into two categories, read access and write access, and the corresponding methods need to be implemented at the model level.

* Read access

| API at the resolver level | Method at the model level | Note |
| ------------- |:-------------:|:-------------:|
| readOne`<model>` | readById | [data loader implementation]({% link data-models/resolver-and-model-layers.md %}#data-loader) |
| count`<models>` | countRecords ||
| `<models>` | readAll | limit-offset based pagination |
| `<models>`Connection | readAllCursor | cursor based pagination |

* Write access

| Method at the resolver level | Method at the model level |
| ------------- |:-------------:|
| add`<model>` | addOne |
| delete`<model>` | deleteOne |
| update`<model>` | updateOne |
| bulkAdd`<model>`Csv | bulkAddCsv |

To implement most read-access methods, familiarize yourself with the pagination, order and search arguments in Zendro — see [General filter arguments]({% link api/graphql-reference.md %}#general-filter-arguments). You'll then need helper functions to parse these arguments into statements or objects for the new storage type's node driver. For example, `searchConditionsToMongoDb` (used by `readAllCursor` for the `mongodb` storage type) parses the search arguments into an object accepted by the MongoDB node driver; `orderConditionsToMongoDb` and `cursorPaginationArgumentsToMongoDb` do the same for order and pagination. The MongoDB node driver then uses these parsed objects to fetch the corresponding records and return them to Zendro.

#### Validation
The `validations` folder contains files for validating records of the new data models. By default you can keep the existing validation files; to add more validation rules, see [Custom Validator Function for AJV]({% link data-models/validation.md %}).

### Operations for associations
To introduce operations for associations among data models, implement some functions at the model level according to the association type and the new storage type.

#### One to one
If the foreign key is stored in the current model, implement the corresponding methods at the model level:

| Method at the resolver level | Method at the model level |
| ------------- |:-------------:|
|`<model>`.prototype.add_unique_`<assoc>`|add_`<assocID>`|
|`<model>`.prototype.remove_unique_`<assoc>`| remove_`<assocID>`|
|bulkAssociate`<model>`With`<assocID>` | bulkAssociate`<model>`With`<assocID>`|
|bulkDisAssociate`<model>`With`<assocID>`| bulkDisAssociate`<model>`With`<assocID>`|

Other relevant methods in the associated models are generated automatically.

#### Many to one / One to many
For models with a `many_to_one` association type, implement the following methods at the model level:

| Method at the resolver level | Method at the model level |
| ------------- |:-------------:|
|`<model>`.prototype.add_`<assoc>`|add_`<assocID>`|
|`<model>`.prototype.remove_`<assoc>`|remove_`<assocID>`|
|bulkAssociate`<model>`With`<assocID>`|bulkAssociate`<model>`With`<assocID>`|
|bulkDisAssociate`<model>`With`<assocID>`|bulkDisAssociate`<model>`With`<assocID>`|

For models with a `one_to_many` association type, the relevant methods are generated automatically.

#### Many to many
For all models with a `many_to_many` association type, implement the following methods at the model level:

| Method at the resolver level | Method at the model level |
| ------------- |:-------------:|
|`<model>`.prototype.add_`<assoc>`|add_`<assocIDs>`|
|`<model>`.prototype.remove_`<assoc>`|remove_`<assocIDs>`|

### Distributed models and adapters
Code for distributed models and adapters with the `ddm-adapter` storage type is generated automatically. You still need to implement the methods for adapters of the new storage type, e.g. `mongodb-adapter`, under `models/adapters`. The methods to implement for adapters and models are approximately the same, so the lists in [Basic CRUD operations](#basic-crud-operations) and [Operations for associations](#operations-for-associations) apply here too — with one exception: `readAll` is never implemented for adapters, since Zendro only supports cursor-based pagination (`readAllCursor`) for them.

### Extension in single page application
1. Add the new storage type to the `StorageType` type in `src/types/models.ts`, e.g. `mongodb` and `mongodb-adapter`.
2. Add the configuration for the new storage type in the `getModelApiPrivileges()` function in `src/utils/models.ts`. This includes the following options:
   * `operators` (`[String]`): allowed operators for the new storage type.
   * `textSearch` (boolean): enables the search bar. By default, Zendro uses the `ilike` operator for search; if your storage type does not support `ilike`, disable `textSearch`.
   * `backwardPagination` (boolean): enables backward pagination.
   * `sort` (boolean): enables sorting by columns.
   * `create` (boolean): enables record creation.
   * `update` (boolean): enables record updates.
   * `delete` (boolean): enables record deletion.
   * `bulkAddCsv` (boolean): enables bulk-adding records from a CSV file.
3. Test the extended SPA. For local databases, run `zendro start spa`. If you're using the dockerized app, first shut it down with `zendro dockerize -d`, then uncomment `zendro_spa` in `docker-compose-dev.yml` and run `zendro dockerize -u`. If the SPA works as expected, commit the changes from steps 1 and 2 on a new branch.

### EJS templates in code generator
If you would like new data model definitions to generate code, implement some EJS templates in `graphql-server-model-codegen`:

1. Create a template file named `create-models-<storageType>` in the `views` folder, e.g. `create-models-mongodb`.
2. Migrate the basic CRUD methods from your modified model files into `create-models-<storageType>`, replacing the concrete model name and other model-specific variables with EJS variables. See the common EJS variables defined in the `getOptions` method of `funks.js`; `create-models-mongodb` is a good concrete example to follow.
3. (optional) If you implemented methods for associations in the new storage type, create a template file named `create-models-fieldMutations-<storageType>` in `views/includes`, e.g. `create-models-fieldMutations-mongodb`.
4. (optional) Migrate the basic association-related methods (i.e. without bulk operations) from your modified model files into `create-models-fieldMutations-<storageType>`. Common EJS variables are again defined in `getOptions` in `funks.js`; `create-models-fieldMutations-mongodb` is a good example.
5. (optional) Extend the `views/includes/bulkAssociations-models` template with bulk-operation methods for associations from your modified model files.
6. (optional) If you implemented adapters for the new storage type, create a `create-<storageType>-adapter` template, e.g. `create-mongodb-adapter`.
7. (optional) Migrate the basic CRUD methods from your modified adapter files into `create-<storageType>-adapter`.
8. (optional) If you implemented methods for associations in adapters, migrate them into the `create-adapter-fields-mutations` and `bulkAssociations-models` templates.
9. Update `lib/generators-aux.js` to add the default database key for the new storage type.
10. Update `lib/operators-aux.js` to specify the operator schema for the new storage type.
11. Update `funks.js` so the code generator can match the new storage type with its templates — search for `mongodb` to find the relevant locations, and add similar code for your new storage type.

### Test
To contribute your storage type to the Zendro main branch, commit the new templates and add tests before submitting a pull request.

The backend test frameworks are `mocha` and `chai`.

#### Unit tests
For Zendro, a unit test means that the code generator produces the expected code for a given set of templates. Add the data model definitions to be tested in `graphql-server-model-codegen/test/unit_test_misc`, the expected code in `graphql-server-model-codegen/test/unit_test_misc/test-describe`, and the test cases in `graphql-server-model-codegen/test/mocha_unit.test`. Run unit tests with `npm run test-unit`. The required unit tests at the model level are:
* **Basic CRUD methods**: readById, countRecords, readAll, readAllCursor, addOne, deleteOne, updateOne, bulkAddCsv
* **Association methods**:
  * OneToOne: add one association, remove one association, bulkAssociation, bulkDisassociation
  * ManyToMany: add associations, remove associations
  * ManyToOne: add one association, remove one association, bulkAssociation, bulkDisassociation

#### Integration tests
Integration tests exercise the API's functionality. Add the data model definitions to `test/integration_test_misc/integration_test_models_instance2`, the container configuration for the new storage type to `test/integration_test_misc/docker-compose-test.yml` (integration tests run via docker and docker-compose), and update `test/integration_test_misc/data_models_storage_config2.json` with the default connection configuration. If `bulkAddCsv` is tested, add the corresponding CSV files under `test/integration_test_misc`. Then create a new integration test file, `test/mocha_integration_<storageType>.test.js`, e.g. `mocha_integration_mongodb.test.js`, and register it in `test/testenv_cli` (search for `mocha_integration_mongodb.test.js` and add your file under the same `if` condition). Run the suite with `npm run test-integration [-- OPTIONS]` (`npm run test-integration -- -h` lists the available commands). The list of concrete tests:
* **Basic CRUD operations**: add, update, readOne, delete, CSV bulk upload, limit-offset pagination, cursor-based pagination, sort, table template acquisition, data loader, search
* **Operators**: like, notLike, iLike, notILike, regexp, notRegexp, iRegexp, notIRegexp, in, notIn, contains, notContains
* **Associations**:
  * **ManyToOne**: add one association, read one associated record, remove one association, bulkAssociation, bulkDisassociation
  * **ManyToMany**: add associations, read one associated record, remove associations
  * **OneToOne**: add one association, read one associated record, remove one association, bulkAssociation, bulkDisassociation

Note: if the new storage type does not have write access or associations, the corresponding tests are unnecessary. If you add distributed models and adapters, add similar tests for them too — except for `readAll`, since adapters only implement cursor-based pagination (`readAllCursor`).
