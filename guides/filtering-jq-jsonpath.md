---
layout: default
title: Filtering with jq and JSONPath
parent: User guides
nav_order: 3
permalink: /guides/filtering-jq-jsonpath
---
# GraphQL filtering using jq and JSONPath
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What is jq?

[jq](https://stedolan.github.io/jq/) is a lightweight and flexible command-line JSON processor. It extracts and manipulates data from JSON documents with simple and powerful filters, letting you transform JSON data in various ways.

Why use jq?
- **Filtering**: quickly filter, slice, and extract specific parts of JSON.
- **Transformation**: modify JSON structures, useful for cleaning, formatting, or transforming JSON data.
- **Simplicity**: a simple syntax lets you do complex manipulations easily.

### Why jq is used with GraphQL

GraphQL responses are returned in JSON format. Since GraphQL allows flexible queries, the structure of the returned data can vary significantly, making it necessary to manipulate the response for specific use cases. This is where jq comes in: after querying a GraphQL endpoint, you might need to isolate or transform the returned JSON data for processing, and `jq` makes this easy, letting you select only the fields you need from the complex JSON response.

## What is JSONPath?

[JSONPath](https://goessner.net/articles/JsonPath/) is a query language for JSON, similar to XPath for XML. It extracts specific parts of a JSON document based on a path expression, and is often used when interacting with JSON data in various programming environments.

Why use JSONPath?
- **Simplifies JSON querying**: easy navigation and extraction from nested JSON structures using path expressions.
- **Integration with APIs**: many tools and libraries support JSONPath for working with JSON-based APIs.
- **Efficient data extraction**: extract only the data you need, saving time and resources.

### Why JSONPath is used with GraphQL

GraphQL responses are JSON-based and can be deeply nested with numerous fields. JSONPath provides a simple, readable way to extract specific data from this structure, making it an excellent tool for working with GraphQL results.

## Example usage

In real-world scenarios, especially with GraphQL APIs, tools like `jq` and `JSONPath` help you interact with responses more efficiently. Zendro's GraphiQL integrates these filtering techniques directly into its interface.

### Filtering in Zendro

Zendro's GraphiQL interface provides a flexible and interactive way to query the data, applying filters directly to your queries to refine the response. It lets you request only the fields you need, and presents the data in JSON format.

In the results, you can see the JSON output, which can be further processed with `jq` or `JSONPath` for specific tasks, such as:
- **Filtering specific fields**: extract only the fields that matter to you from the GraphQL response.
- **Nested data extraction**: deal with nested data structures, and query deeper levels of the data tree with additional tools like `jq` or `JSONPath`.

Once you have a query, at the top of the interface there are several buttons. Clicking `Filter` opens a section at the bottom of the screen where you can select `jq` or `JSONPath`, write your filter, and see the results.

![Filter.png](/figures/jq1.png)
![Filter_options.png](/figures/jq2.png)

The filters you use are applied to the query results. You can reproduce the results by accessing the [Zendro-BrAPI data warehouse](https://brapi-graphiql.zendro-dev.org/).

### Filtering with jq

For example, if we want the names of the studies in the trial example, calling only 2 (`limit:2`), we expect 2 names:

![Filter_example.png](/figures/jq3.png)

#### Explanation

- `.trials[]`: `.trials` accesses the `trials` property of the root object. The `[]` selects all elements of that array — in `jq`, `[]` iterates over an array to extract all its elements.
- `.studiesFilter[]`: same idea — selects the `studiesFilter` property inside each element of `trials`, again using `[]` to access each element within `studiesFilter`.
- `.studyName`: finally, accesses the `studyName` property inside each `studiesFilter` object.

Or, to get only the variable name and its value, ignoring null values:

```
[.trials[].studiesFilter[].observationsFilter[] | select(.value != null) | { name: .observationVariable.observationVariableName, value: .value }]
```

#### Explanation

- `.trials[]`: accesses the `trials` property of the root object, selecting all elements of the `trials` array.
- `.studiesFilter[]`: within each `trials` element, accesses `studiesFilter` and iterates over its elements.
- `.observationsFilter[]`: within each `studiesFilter` element, accesses `observationsFilter` and iterates over its elements.
- `select(.value != null)`: filters the elements of `observationsFilter`, keeping only those where `value` is not null.
- `{ name: .observationVariable.observationVariableName, value: .value }`: creates a new object for each filtered element, extracting `observationVariableName` from `observationVariable` and `value` from the current object, renamed `name` and `value`.

Result:

```json
{
  "data": [
    {
      "name": "fresh root yield|CO_334:0000013",
      "value": 5
    },
    {
      "name": "germination count|CO_334:0000166",
      "value": 93.3
    },
    {
      "name": "harvest index variable|CO_334:0000015",
      "value": 0.2
    },
    {
      "name": "initial plant vigor assessment 1-5|CO_334:0000220",
      "value": 4
    },
    {
      "name": "plant height measurement in cm|CO_334:0000018",
      "value": 240
    },
    {
      "name": "plant stands harvested counting|CO_334:0000010",
      "value": 9
    },
    {
      "name": "rotten root percentage|CO_334:0000229",
      "value": 0
    },
    {
      "name": "selected variety boolean 0&1|CO_334:0000232",
      "value": 0
    },
    {
      "name": "fresh root yield|CO_334:0000013",
      "value": 9
    },
    {
      "name": "germination count|CO_334:0000166",
      "value": 60
    },
    {
      "name": "harvest index variable|CO_334:0000015",
      "value": 0.37
    },
    {
      "name": "initial plant vigor assessment 1-5|CO_334:0000220",
      "value": 3
    },
    {
      "name": "plant stands harvested counting|CO_334:0000010",
      "value": 9
    },
    {
      "name": "selected variety boolean 0&1|CO_334:0000232",
      "value": 0
    }
  ]
}
```

### Filtering with JSONPath

Now, if we want the IDs of the studies in the trial example, calling only 2 (`limit:2`), we expect 2 IDs:

![Filter_example.png](/figures/jp1.png)

#### Explanation

JSONPath's syntax is very similar to file paths or regular expressions. It navigates through a JSON object or array and extracts specific elements.

- `$`: the dollar sign represents the root object — the starting point of the query.
- `.trials[*]`: accesses the `trials` property of the root object and selects all elements of that array. The asterisk `[*]` selects all elements.
- `.studiesFilter[*]`: selects the `studiesFilter` property within each `trials` object, again selecting all elements.
- `.studyDbId`: finally, after traversing all elements of `studiesFilter`, selects the `studyDbId` property within each.

#### Key differences from jq

JSONPath does not support creating new objects like `jq` does with `{ name: .observationVariable.observationVariableName, value: .value }`. To get both `name` and `value` in a combined result, you'd typically need separate queries and combine the results programmatically.

```js
.trials[*].studiesFilter[*].observationsFilter[?(@.value != null)].observationVariable.observationVariableName

.trials[*].studiesFilter[*].observationsFilter[?(@.value != null)].value
```

#### Explanation

- `$.trials[*]`: accesses the `trials` property of the root object and selects all elements.
- `$.studiesFilter[*]`: within each `trials` element, selects the `studiesFilter` property and iterates over it.
- `$.observationsFilter[?(@.value != null)]`: within each `studiesFilter` element, selects `observationsFilter` and filters elements where `value` is not null (similar to `select(.value != null)` in `jq`).
- `.observationVariable.observationVariableName`: selects the `observationVariableName` property inside `observationVariable`.
- `.value`: selects the `value` property within the filtered `observationsFilter` elements.

---

By combining GraphQL with tools like jq and JSONPath, you can efficiently retrieve and manipulate data, making your API interactions even more powerful.
