# Distributed Data Models

In(at?) Zendro is possible to integrate data which follows a same data model description stored in diferent servers. For example, let say there is two research groups working in the same topic, each one of them stores their data in their own servers. Distrubuted data models will allow them to access both data sets from one same access point created with Zendro.
The description for this type of models follows the same description here(insert link), with some extra fields marked in bold in the following table, but also we will need to specify an adapter for each one of the data sets.  

Name | Type | Description
------- | ------- | --------------
*model* | String | Name of the model (it is recommended to use snake_case naming style to obtain nice names in the auto-generated GraphQL API). The string here can not contain spaces.
*storageType* | String | Type of storage where the model is stored. In this case the storage type should be _distributed-data-model_
*__registry__* | Array | Array of strings, each string is the name of one of the adapters(insert link here to adapters description) that the distributed data model  is integrating.
*attributes* | Object |  The key of each entry is the name of the attribute and there are two options for the value . It can be either a string indicating the type of the attribute or an object where the user indicates the type of the attribute(in the _type_ field) together with an attribute's description (in the _description_ field). See the [table](#supported-data-types) below for allowed types. Example of option one: ```{ "attribute1" : "String", "attribute2: "Int" }``` Example of option two: ``` { "attribute1" : {"type" :"String", "description": "Some description"}, "attribute2: "Int ```
*associations* | Object | The key of each entry is the name of the association and the value should be an object describing the corresponding association. See [Associations Spec](#associations-spec) section below for details.
*internalId* | String | This string corresponds to the name of the attribute that uniquely identifies a record. If this field is not specified, an _id_, default attribute, will be added.


### Adapters

 An adapter in Zendro is a json file which describes how and which data can be accessed from a _distributed data model_ instance of Zendro.
 The user needs one adapter for each server and for each model that the _distributed data model_ instance will integrate. The adapter's json file also follows the description for data models (insert link), with some extra fields marked in bold in the following table.

 Name | Type | Description
 ------- | ------- | --------------
 *model* | String | Name of the model (it is recommended to use snake_case naming style to obtain nice names in the auto-generated GraphQL API). The string here can not contain spaces.
 *__storageType__* | String | Type of storage where the remote or local data is stored. So far there is 4 possible types of adapters: <ul><li> __sql-adapter__ - for data stored locally  </li> <li>__ddm-adapter__ - for data stored remotely in another distributed data models zendro instance</li> <li>__zendro-webservice-adapter__ - for data stored remotely in a general zendro instance </li> <li>__generic-adapter__ - for data stored in a non-zendro server </li></ul>
 *__adapterName__* | String | Name of the adapter, this name will be used in the __registry__ (insert link) of a *distributed data model* zendro instance.
  *__regex__* | String | Regular expresion, this regular expresion is used by the *distributed data model* zendro instance in order to know which adapter will be the one in charge of performing the operation with a given registry or set of registries. Each registry accessed through this adapter has in its id this regular expression.
  *__url__* | String | Url where data, that this adapter is responsible to integrate, can be fetched.
 *attributes* | Object |  The key of each entry is the name of the attribute and there are two options for the value . It can be either a string indicating the type of the attribute or an object where the user indicates the type of the attribute(in the _type_ field) together with an attribute's description (in the _description_ field). See the [table](#supported-data-types) below for allowed types. Example of option one: ```{ "attribute1" : "String", "attribute2: "Int" }``` Example of option two: ``` { "attribute1" : {"type" :"String", "description": "Some description"}, "attribute2: "Int ```
 *associations* | Object | The key of each entry is the name of the association and the value should be an object describing the corresponding association. See [Associations Spec](#associations-spec) section below for details.
 *internalId* | String | This string corresponds to the name of the attribute that uniquely identifies a record. If this field is not specified, an _id_, default attribute, will be added.

## TO DO
 - [ ] Mention that only cursor based pagination works in this case
 - [ ] _Read all_ is not implemented, only via "Connections"
 - [ ] Add description of an example with a image representing the topology of the example
 - [ ] Add json description of the above
 - [ ] API examples
