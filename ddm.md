# Distributed Data Models

Zendro provides the ability to build a network of zendro graphql-servers, able to talk to and request data from each other, as long as the graphql schema for a given model is identical to all zendro instances inside the cloud of graphql-servers.

To do so it is necessary to define your data-model as storageType "distributed-data-model" (ddm) and provide zendro with adapters to either a local storage database or another zendro instance to be queried for data. Since Zendros data-model-definitions aim to be atomic and independent of any other model, those adapters have be defined as separate data models, even though they are linked to the distributed-data-model (ddm). That means that only the ddm is exposed to the graphql-schema, the adapters are then used internally to request the expected data.

## Data model definition

Defining a distributed-data-model (ddm) and its adapters requires some special attributes in the model definition:


| Name | Type | Description |
| ----------------- | ----------------- | ----------------------- |
| registry | String[] | List of registered adapters the ddm should request |
| internalId | String | While optional in basic models, ddms require a String identifier attribute, instead of e.g. the default numeric sequence. |
| adapterName | String | Only used in the adapter definitions. References the name given in the "registry" List | 
| regex | String | Only used in the adapter definitions. Regular expression used to identify the adapter a specific record belongs to (using its internalId). This is used to identify responsible adapters for read-one and write actions | 
| url | String | Only used in ddm-adapter definitions | URL of the graphql-server to forward the requests to. |



Below is an example of a distributed "country" model, with two so-called "adapters", one "local" adapter that directly connects to a relational database and one "remote" adapter that simply forwards the request to the next zendro node in the network, given a URL. Each adapter you want the distributed-data-model to be aware of needs to be defined as such in a separate JSON data-model-definition file.

```json
// Country definition File
{
  "model": "country",
  // set the storage-type to distributed-data-model
  "storageType": "distributed-data-model",
  // Use the "registry" key to register a list of adapters you want this
  // specific model to be aware of. 
  "registry": ["country-server1","country-server2"],
  // distributed data-models have to use an internalId of type String. This is
  // so the model can differentiate between the registered adapters using the id
  // and a regular expression.
  "internalId": "id",
  "attributes": {
    "id": "String",
    "name": "String",
    "population": "Int"
  }
}
```

```json
// Country local adapter file
{
  // Use the same name as above "ddm" definition
  "model": "country",
  // set the storage type to the desired type and add a "-adapter" to signal
  // that this model functions as an adapter.
  "storageType": "sql-adapter",
  // Reference the name given in the "registry" list of the
  // distributed-data-model above
  "adapterName": "country-server1",
  // Regular expression to determine whether a record can be fetched using this
  // adapter. In this example the ids of country-records stored on this local
  // sql-adapter use a "/countries/server1" signature.
  "regex": "/countries/server1",
  // distributed data-models have to use an internalId of type String. This is
  // so the model can differentiate between the registered adapters using the id
  // and a regular expression.
  "internalId": "id",
  // The adapter uses the same attributes defined in the above
  // distributed-data-model definition
  "attributes": {
    "id": "String",
    "name": "String",
    "population": "Int"
  }
}
```

```json
{
  // Use the same name as above "ddm" definition
  "model": "country",
  // The ddm-adapter type is used to signal zendro that this adapter forwards
  // the request to another zendro instance. The other zendro instance also has
  // a registered distributed-data-model which then handles the request
  // accordingly (e.g. ask its registered local mongodb-adapter for data).
  "storageType": "ddm-adapter",
  // The graphql-server URL of the remote zendro instance, used to forward the
  // incoming request.
  "url": "http://<server2>/graphql",
  // Reference the name given in the "registry" list of the
  // distributed-data-model above
  "adapterName": "country-server2",
  // Regular expression to determine whether a record can be fetched using this
  // adapter. In this example the ids of country-records stored on this local
  // sql-adapter use a "/countries/server2" signature.
  "regex": "/countries/server2",
  // distributed data-models have to use an internalId of type String. This is
  // so the model can differentiate between the registered adapters using the id
  // and a regular expression.
  "internalId": "id",
  // The adapter uses the same attributes defined in the above
  // distributed-data-model definition
  "attributes": {
    "id": "String",
    "name": "String",
    "population": "Int"
  }
}
```

## Functionality

Zendros distributed-data-model work efficiently by distributing incoming requests onto the corresponding responsible adapters. When the user sends any graphql request (query or mutation) the distributed-data-model (ddm) will pass along the request to its registered adapters, collect the results and post-process any necessary modifications in-memory (e.g. sorting all results fetched from different adapters).

In case of read-one and write actions the ddm recognizes a responsible adapter via the given regular expression on the `internalId` field. For that each adapter has a simple function to recognize if they are responsible:

```js
/**
 * recognizeId - test whether this adapter is responsible for a given iri (id).
 * It does this via a simple match of a regular expression.
 * @param {string} iri - The iri to test
 * @return {boolean} returns true if match, false otherwise
 */
static recognizeId(iri) {
  return iriRegex.test(iri);
}
```
When the responsible adapter is found the ddm can forward the request to the adapter, collect the result and return it to the user.

In case of a read-many function the ddm will query _all_ registered adapters for the data. It will pass along any query parameters given (search, order, pagination), collect the results and return them to the user. An example of a two-server zendro network is depicted below:  


|![distribute-data-model functionality](./figures/ddm-drawing.png)|
|:--|
|A depiction of a two-server zendro network using distributed-data-models. The green arrows show the flow of a request to "Graphql server 1". The user will request some data and after passing the resolver layer for authorization checks the "distributed-data-model" forwards the request to its 2 registered adapters. One "local" adapter which will immediately query a database and one "remote" adapter which will forward the graphql-request to "Graphql server 2". This will in turn go through the resolver of Graphql Server 2 and again distributed to the registered adapters by the "distributed-data-model". Here zendro automatically prevents a circular request by _not_ sending the request back through the "remote adapter" of server 2. However the "local adapter" of server 2 will in turn query its connected database and return the data back to the user. After collecting all data the distributed-data-model in server 1 will do any necessary post-processing like sorting etc. and return the data to the user.|


### Pagination

## Network types

## Performance and Limitations