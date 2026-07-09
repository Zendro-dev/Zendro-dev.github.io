---
layout: default
title: GraphQL basics
parent: User guides
nav_order: 2
permalink: /guides/graphql-basics
---
# Introduction to GraphQL and querying the API
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

GraphQL is a query language for Application Programming Interfaces (APIs), which documents what data is available in the API and lets you query and get exactly the data you want, and nothing more.

This tutorial provides a short introduction to GraphQL, but we recommend exploring the [GraphQL documentation](https://graphql.org/learn/) and other [introductory resources like this one](https://docs.github.com/en/graphql/guides/introduction-to-graphql) to learn more.

In the GraphQL API, queries are written in the GraphQL language, and the result (the data) is given back in [JSON](https://www.w3schools.com/whatis/whatis_json.asp) format. JSON (JavaScript Object Notation) is a standard text-based format for representing structured data, widely used for transmitting data in web applications, and easily reformatted into tables or data frames in programming languages like R or Python.

Zendro provides a GraphQL API web interface called Graph**i**QL, a web browser tool for writing, validating, and testing GraphQL queries.

You can try it live here — the API used in this and other tutorials: [https://zendro.conabio.gob.mx/api/graphql](https://zendro.conabio.gob.mx/api/graphql).

Zendro's GraphQL API allows not only querying data, but also creating, modifying or deleting records (`mutate`). This is only available with authentication (i.e. logging in with edit permissions) and isn't covered in this tutorial — see Zendro's other how-to guides for details on mutations.

## GraphiQL web interface

The GraphiQL API web interface has the following main components:

* A **left panel** where you write your query in GraphQL format.
* A **right panel** where the query result is shown in JSON format.
* A **play button** to execute the query. Its keyboard shortcut is `Ctrl+E`.
* A **Documentation Explorer** side menu, shown or hidden by clicking "Docs" in the top right corner.

![API_parts.png](/figures/API_parts.png)

Data in GraphQL is organised in **types** and **fields** within those types. When thinking about your structured data, you can think of **types as the names of tables**, and **fields as the columns of those tables** — records are the rows of data from those tables. Learn more in the [GraphQL documentation](https://graphql.org/learn/).

A GraphQL service is created by defining types and fields on those types, and providing functions for each field on each type.

The documentation explorer shows what operations (e.g. query, mutation) are allowed for each type. Clicking `Query` opens another view with the details of the operations available to query the data — all types in a given dataset are listed alphabetically, with the operations available for each listed below.

In the image above, the first type is `cities`. Types can contain elements or arguments, specified inside parentheses `()`. Some may be required arguments (marked with `!`), such as `pagination`.

You can extend the bottom panel ("Query variables") to provide dynamic inputs to your query. [Learn more](https://graphql.org/learn/queries/#variables).

## Writing queries

The [GraphQL documentation](https://graphql.org/learn/) has plenty of resources on how to build queries and make the most of GraphQL; below is a short summary, after which we recommend exploring it further. Feel free to try your queries in our [Zendro Dummy API](https://zendro.conabio.gob.mx/dummy_api), set up for tests.

**GraphQL syntax tips:**

* Queries and other operations are written between curly braces `{}`.
* Types can contain elements or arguments, specified inside parentheses `()`.
* Use a colon `:` to set parameter arguments (e.g. `pagination:{limit:10, offset:0}`).
* Use a hashtag `#` to include comments within a query, useful for documenting what you're doing.
* A query should provide at least one type (e.g. `rivers`), at least one field (e.g. `names`), and any mandatory arguments the type has (marked with `!` in the Docs).
* In Zendro, `pagination` is a mandatory argument. It refers to the number of records (`limit`) the output returns, starting from a given `offset`. If you don't specify the offset, it defaults to `offset:0`.

A simple query looks like this:

```
query {
  rivers(pagination:{limit:10, offset:0}){
   # fields you want from the "rivers" type go here
    name
  }
}
```

Copy-pasting and executing the query above in GraphiQL looks like this — we got the names of the first 10 rivers in the data:

![API_query1.png](/figures/API_query1.png)

But how did we know `name` is a field within `rivers`? There are two options:

**Option 1: check the Docs panel**

Click `Query`, look for the type you want (in this example `rivers`), then click `[river]`. This opens the list of available fields, along with their documentation:

![API_docs2.png](/figures/API_docs2.png)

**Option 2: autocomplete while you type**

To see what fields are available for the `rivers` type, hold `ctrl+space` inside the curly braces `{}` after `rivers(pagination:{limit:10, offset:0})`. A menu appears showing all possible fields.

![API_query2.png](/figures/API_query2.png)

Here we can get the fields `river_id`, `name` and `country_ids`. The rest of the list relates to `countries`, since `rivers` is associated with `countries`, letting us build a more complex query.

Let's build a query that returns the fields `river_id`, `name`, `length` and `country_ids` from the `river` type:

```
query {
  rivers(pagination:{limit:10, offset:0}){
    river_id
    name
    length
    country_ids
  }
}
```

For each of the first 10 rivers (`limit:10`) in the data, we get its ID, name, length, and the ID of any country it's associated with:

![API_query3.png](/figures/API_query3.png)

### Extracting data from different types (i.e. dealing with associations)

GraphQL can fetch fields associated with a record across different types, so you only retrieve the variables and records you need from the entire dataset. For example, you can get the name and length of a river, as well as the name and population of the countries it crosses.

Extracting data from associated types depends on whether the association is *one-to-one* (a city belongs to one country) or *one-to-many* (a river can cross many countries).

#### One to one

For a *one-to-one* association, the associated data model appears as just another field — for example, each `city` is associated with one `country`, so `country` is one of the fields available within `cities`.

Looking at the Docs, you'll notice it's not just another field — you need to provide it with an input search.

![API_city](/figures/API_city.png)

Here we want to look up the associated country, and we know the common field (the key) is `country_id`, so the search should look like:

```
query {
  cities(pagination: {limit: 10, offset: 0}) {
    city_id
    name
    population
    country(search: {field: country_id}) {
      name
      population
    }
  }
}
```

#### One to many

For a *one-to-many* association, there's a `Connection` for each association the model has. For example, to see the countries a river is associated with, use `countriesConnection`:

```
query {
  rivers(pagination:{limit:10, offset:0}){
      river_id
      name
      length
      country_ids
      countriesConnection(pagination: {first: 1}) {
      countries{
        name
        population
      }
    }
  }
}
```

Remember to check the Docs for any mandatory argument — here, `pagination` is mandatory. You can check what's expected in its `paginationCursorInput` by clicking on it in the documentation. Also see the [pagination argument]({% link api/graphql-reference.md %}#pagination-argument) documentation for details.

After executing the query, you get the same data as before for each river, plus the data of the country (or countries) it's associated with.

![API_query4.png](/figures/API_query4.png)

In the examples above, all the arguments are inline in the query string. But arguments to fields can also be dynamic — for instance, an application might have a dropdown menu letting the user select which City they're interested in, or a set of filters.

To improve run time, GraphQL can factor dynamic values out of the query and pass them as a separate dictionary — these are called **variables**. Common variables include search, order and pagination. To use variables:

1. Replace the static value in the query with `$variableName`.
2. Declare `$variableName` as one of the variables accepted by the query.
3. Pass `variableName: value` in the separate, transport-specific (usually JSON) variables dictionary.

Check the [official documentation](https://graphql.org/learn/queries/#variables) for examples.

You can write much more complex queries to get exactly the data you want — explore the [GraphQL documentation](https://graphql.org/learn/) or other resources to learn more. The examples above should get you going if you want to pull data into R or Python for analysis.

Before downloading data from R, Python or any other programming language via the GraphQL API, we recommend writing and testing the query in the GraphiQL web interface first, making sure it returns the desired data in the right panel, as shown above.

**Next step?** See [Filtering with jq and JSONPath]({% link guides/filtering-jq-jsonpath.md %}), or the [Python]({% link guides/python-api.md %}) and [R]({% link guides/r-api.md %}) guides for using the GraphQL API from those languages.
