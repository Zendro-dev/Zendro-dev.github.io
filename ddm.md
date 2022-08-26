# Distributed Data Models

Zendro provides the ability to build a network of zendro graphql-servers, able to talk to and request data from each other, as long as the graphql schema for a given model is identical to all zendro instances inside the cloud.

To do so it is necessary to define your data-model as storageType "distributed-data-model" and provide zendro with adapters to either a local storage database or another zendro instance to be queried for data. Since Zendros data-model-definitions aim to be atomic and independent of any other model, those adapters can be defined as separate data models.