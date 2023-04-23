---
layout: default
title: Data validation
parent: Data models
nav_order: 1
grand_parent: Getting started
permalink: /setup_root/data_models/model_resolver_layers
---


# The Resolver Layer and the Model Layer
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# The Resolver Layer and the Model Layer

The Zendro server uses two different layers below the GraphQL schema (which defines the functions that the GraphQL server understands) to process data, the resolver layer and the model layer. The main reason for this is that Zendro supports a growing number of storage types, and the more abstract layer (the resolver) is supposed to be storage type agnostic. There is one exception to this (in the case of distributed data models, one resolver function cannot be supported, see [below](#pagination-types)), but otherwise, it holds true. So the resolver layer provides an interface that the GraphQL schema can use, and (on the other side) an interface that a new storage type model has to fulfill.