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

```js
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

```js
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

```js
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

In case of read-one and write actions the ddm recognizes a responsible adapter via the given regular expression on the `internalId` field. For that each adapter exposes a simple function to recognize if they are responsible:

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
If desired, this function can be customized separately for each adapter. Instead of using a regular expression to distinguish between adapters any function that returns a `boolean` value given an id can be used.

When the responsible adapter is found the ddm can forward the request to the adapter, collect the result and return it to the user.

In case of a read-many function the ddm will query _all_ registered adapters for the data. It will pass along any query parameters given (search, order, pagination), collect the results and return them to the user. Any errors encountered from querying the different nodes in the network will be passed along and displayed as benign, while data from working nodes is still returned. An example of a two-server zendro network is depicted below:  


|![distribute-data-model functionality](./figures/ddm-drawing.png)|
|:--|
|A depiction of a two-server zendro network using distributed-data-models. The green arrows show the flow of a request to "Graphql server 1". The user will request some data and after passing the resolver layer for authorization checks the "distributed-data-model" forwards the request to its 2 registered adapters. One "local" adapter which will immediately query a database and one "remote" adapter which will forward the graphql-request to "Graphql server 2". This will in turn go through the resolver of Graphql Server 2 and again distributed to the registered adapters by the "distributed-data-model". Here zendro automatically prevents a circular request by _not_ sending the request back through the "remote adapter" of server 2. However the "local adapter" of server 2 will in turn query its connected database and return the data back to the user. After collecting all data the distributed-data-model in server 1 will do any necessary post-processing like sorting etc. and return the data to the user.|


### Cursor-based Pagination

To make sure distributed-data-models can be paginated efficiently they only support cursor-based pagination. See the documentation of the [Pagination Argument](api_graphql.md#pagination-argument) for more information. Since the ddm will fetch data from multiple sources and databases we cannot use the classic limit-offset based pagination. Instead, when requesting paginated data zendro sends along a cursor (a base64 encoded record) and the amount (first or last x records) to all data sources and fetches data after, or before that cursor. After all returned data is collected a final pagination to the first, or last number of requested records is done in memory and send to the user.

## Network topologies

Depending on the use-case several different topologies of "zendro-networks" can be achieved by configuring the data-model-definitions on each participating zendro instance. Below are two examples on how different setups could be achieved. Note that this list is _not_ exhaustive. Zendro makes sure to prevent any circular requests by excluding adapters that already have been queried.


### Example 1: Mesh topology

![mesh-topology](./figures/mesh-ddm.png)

A mesh topology where each participating zendro node has access to all other zendro nodes in the network can be achieved simply by making all nodes aware of each other. In practical terms that means on every zendro instance exists a model of storagetype "distributed-data-model", a ddmadapter for every zendro-node in the network and, if exists, an adapter to a local storage. That means, no matter from which node a user requests data, _every_ node in the network will return the data stored at their local site and the end result will include data from _all_ zendro nodes in the network.
### Example 2: Master-slave topology
![master-slave-topology](./figures/ms-ddm.png)

A master-slave topology where one zendro-node acts as a central controller can be achieved by only making the central node aware of the other instances. To do so, only that node defines a model of storagetype "distributed-data-model" and ddm-adapters for each node in the network. All other secondary nodes do _not_ need a "distributed-data-model" in this case. That means that only when querying the central node, data from all other nodes will be returned. If a user queryies one of the secondary nodes only data specifically from the requested node can be returned.


## Performance and Limitations

Zendro ensures performance of any kind of distributed setup through multiple strategies. 

First and foremost the overhead resulting from distributing the incoming request to multiple adapters is kept at a minimum. Due to the paginated nature of results collected at the distributed-data-model post-processing of the data in memory is usually very fast. Only the collected result has to sorted and paginated once from the individual adapter results. Keep in mind that all parameters (search, oder, pagination) are forwarded to the adapters, which will take care of the execution of the database queries. Requests from the distributed-data-model are send to adapters in parallel, the limiting factor will be the slowest node in the network to return data.

To ensure efficient read of associations it is recommend to use [paired-end foreign keys](setup_data_scheme.md#paired-end-foreign-keys) for any associations among distributed data models. This ensures that associations can be resolved directly from the read record itself instead of the need to query multiple distributed zendro instances for records matching the requested association.

Due to Zendro requiring the use of a pagination argument the user needs to think about what and especially how much data he is requesting. Scaling up a zendro network to include a lot of nodes means that for each of the connected databases a query with the requested pagination will be send and that data has to be collected in memory at the requesting node. This can grow quickly out of proportion if a network and the requested data become very big. 