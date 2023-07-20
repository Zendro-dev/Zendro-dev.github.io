---
layout: default
title: Resolver and Model Layers
parent: Data models
nav_order: 1
grand_parent: Getting started
permalink: /setup_root/data_models/model_resolver_layers
---

# The Resolver Layer and the Model Layer

The Zendro server uses two different layers below the GraphQL schema (which defines the functions that the GraphQL server understands) to process data, the resolver layer and the model layer. The main reason for this is that Zendro supports a growing number of storage types, and the more abstract layer (the resolver) is supposed to be storage type agnostic. There is one exception to this (in the case of distributed data models, one resolver function cannot be supported, see [below](#pagination-types)), but otherwise, it holds true. So the resolver layer provides an interface that the GraphQL schema can use, and (on the other side) an interface that a new storage type model has to fulfill.

## Pagination types

As explained before, Zendro provides two different types of pagination: Limit-offset based and cursor based. Limit-offset based pagination is the better known one, where the user provides an offset (how many entries to skip) and a limit (how many entries to display on one page).

Limit-offset based pagination is not possible for distributed data models, i.e. models where records of a single table are spread over different servers. This is because the client (who makes the request) does not know how the entries are distributed, so the offset for the different servers cannot be provided. Since this data model is especially useful for Big Data, it is not feasible to request the entire table and paginate the data on the client side.

Instead, a different model has to be used. If the different servers are told which was the last entry "above" the requested page (a cursor), they can deliver the content that would follow up to it and the client can now get the received data in order. This type of pagination works for any kind of data, although it is more complex (the entries for the table are found under `edges[] -> node`).