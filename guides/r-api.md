---
layout: default
title: R API
parent: User guides
nav_order: 6
permalink: /guides/r-api
---
# How to get data from Zendro to R
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Libraries needed for this tutorial:

```r
library(httr)
library(jsonlite)
library(dplyr)
library(stringr)

## The following objects are masked from 'package:stats':
## 
##     filter, lag

## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

This tutorial uses the library `httr` to establish the connection with the GraphiQL API, but there are also other options to interact with GraphQL from R. Please check the R packages: [ghql](https://docs.ropensci.org/ghql/), [gqlr](https://github.com/schloerke/gqlr) and [graphql](https://github.com/ropensci/graphql).

## Introduction to GraphQL API and how to query it

GraphQL is a query language for Application Programming Interfaces (APIs). Queries are written in the GraphQL language, and the result (the data) is given back in [JSON format](https://www.w3schools.com/whatis/whatis_json.asp).

If you are not familiar with GraphQL, we recommend you start by checking the [GraphQL basics]({% link guides/graphql-basics.md %}) guide.

Zendro provides a GraphQL API web interface, called Graph**i**QL, which is a Web Browser tool for writing, validating, and testing GraphQL queries.

For example, try copy-pasting and executing the following query at [https://zendro.conabio.gob.mx/api/graphql](https://zendro.conabio.gob.mx/api/graphql), which is the API that we will be using in this and other tutorials.

```python
query {
  rivers(pagination: {limit: 10 offset: 0}) {
    river_id
    name
    length
  }
}
```

(The example above only gets the first 10 results; further down we explain how to define `pagination` to pull down a given number, or all, of the items in a dataset.)

## Download a small dataset (<1,000 elements)

The function `get_from_graphQL()` defined below queries a GraphQL API and transforms the data from JSON format (the output of GraphQL) into an R data frame you can easily use for further analyses. If you want to know what's going on inside this function, there's a step-by-step description at the end of this document.

To start using `get_from_graphQL()`, first run the code below to load the function into your R environment (you can also keep it in a separate file and use `source()` to run it):

```r
get_from_graphQL<-function(query, url){
### This function queries a GraphiQL API and outputs the data into a single data.frame

## Arguments
# query: a graphQL query. It should work if you try it in graphiQL server. Must be a character string.
# url = url of the server to query. Must be a character string.

## Needed libraries:
# library(httr)
# library(jsonlite)
# library(dplyr)
# library(stringr)

### Function

##  query the server
result <- POST(url, body = list(query=query), encode=c("json"))

## check server response
satus_code<-result$status_code

if(satus_code!=200){
  print(paste0("Oh, oh: status code ", satus_code, ". Check your query and that the server is working"))
}

else{

  # get data from query result
  jsonResult <- content(result, as = "text")

  # check if data downloaded without errors
  # graphiQL will send an error if there is a problem with the query and the data was not downloaded properly, even if the connection status was 200.
  ### FIX this when != TRUE because result is na
  errors<-grepl("errors*{10}", jsonResult)
  if(errors==TRUE){
    print("Sorry :(, your data downloaded with errors, check your query and API server for details")
  }
  else{
  # transform to json
  readableResult <- fromJSON(jsonResult,
                           flatten = T) # this TRUE is to combine the different lists into a single data frame (because data coming from different models is nested in lists)

  # get data
  data<-as.data.frame(readableResult$data[1])

  # rename colnames to original variable names
  x<-str_match(colnames(data), "\\w*$")[,1] # matches word characters (ie not the ".") at the end of the string
  colnames(data)<-x # assign new colnames
  return(data)
    }
  }
}
```

`get_from_graphQL()` allows you to get data of up to 1,000 elements (results of your query) at a time, which is the maximum number allowed by GraphQL for a single batch. In the next section we explain how to use `pagination` to download larger datasets in batches.

To use the `get_from_graphQL()` function, first you have to define a GraphQL query. If you don't know how to do this, start by checking the [GraphQL basics]({% link guides/graphql-basics.md %}) guide.

Once you have a GraphQL query working, you'll need to save it to an R object as a character vector:

```r
my_query <- "
{
  rivers(pagination: {limit: 10, offset: 0}) {
    river_id
    name
    length
  }
}
"
```

Next we use this query as an argument for `get_from_graphQL()`, along with the url of the API, which is the same as the GraphiQL web interface you explored above:

```r
data<-get_from_graphQL(query=my_query, url="https://zendro.conabio.gob.mx/api/graphql")
```

If all went well you will get a data frame with the result of your query:

```r
head(data)
```

| river_id | name | length |
|---|---|---|
| 1 | Acaponeta | 233 |
| 10 | Coatán | 75 |
| 11 | Coatzacoalcos | 325 |
| 12 | Colorado | 160 |
| 13 | Concepción | 335 |
| 14 | Culiacán | 875 |

## Download a dataset with more than 1,000 elements

GraphQL outputs the results of a query in batches of max 1,000 elements. So if the data you want to download is larger than that, you need to **paginate**, i.e. get the data in batches. `pagination` is an argument within GraphQL queries that can be done in two ways:

* *Limit-offset*: indicating the first element to get (`offset`, default 0) and the number of elements to get (`limit`). `limit` can't be larger than `1000`.
* *Cursor-based*: indicating the unique ID (`cursor`) of the element to get first, and a number of elements to get after it.

Zendro uses limit-offset pagination with the syntax:

`pagination:{limit:[integer], offset:[integer]}`

See the [GraphQL documentation](https://graphql.org/learn/pagination/) and this [tutorial on GraphQL pagination](https://daily.dev/blog/pagination-in-graphql) for more details.

In the previous examples we downloaded only 10 elements (`pagination:{limit:10}`) from the rivers type, but the dataset is larger. (Remember, data in GraphQL is organised in **types** and **fields** within those types. When thinking about your structured data, you can think of types as the names of tables, and fields as the columns of those tables. In the example above, `rivers` is a type, and the fields are `river_id`, `name`, `length`, among others.)

To find out how many elements a type has, we can make a query with the `count` function, if it's available for the type we're interested in — check this in the `Docs` panel at the top right of the GraphiQL interface.

For example, `rivers` has the function `countRivers`, so with the query `{countRivers}` we can get the total number of rivers.

Similar to how we got data before, we can use this simple query in the `get_from_graphQL` function to get the number of rivers into R:

```r
# query API with count function
no_records<-get_from_graphQL(query="{ countRivers }", url="https://zendro.conabio.gob.mx/api/graphql")

# change to vector, we don't need a df
no_records<-no_records[1,1]
no_records
```
```
## [1] 50
```

In this case we have 50. Technically we could download all the data in a single batch, since it's under 1000, but for demonstration purposes we'll download it in batches of 10.

The following code calculates the number of pages needed to get a given number of records, assuming a desired limit (the size of each batch). It then runs `get_from_graphQL()` in a loop for each page, until it has fetched the total number of records desired.

```r
# Define desired number of records and limit. Number of pages and offset will be estimated based on the number of records to download
no_records<- no_records # this was estimated above with a query to count the total number of records, but you can also manually change it to a custom desired number
my_limit<-10 # max 1000.
no_pages<-ceiling(no_records/my_limit)

## Define offset.
# You can use the following loop:
# to calculate the offset automatically based on
# on the number of pages needed.
my_offset<-0 # start in 0. Leave like this
for(i in 1:no_pages){ # loop to
  my_offset<-c(my_offset, my_limit*i)
}

# Or you can define the offset manually
# uncommenting the following line
# and commenting the loop above:
# my_offset<-c(#manually define your vector)

## create object where to store downloaded data. Leave empty
data<-character()

##
## Loop to download the data from GraphQL using pagination
##

for(i in c(1:length(my_offset))){

# Define pagination
pagination <- paste0("limit:", my_limit, ", offset:", my_offset[i])

# Define query looping through desired pagination:
my_query<- paste0("
{
  rivers(pagination: {", pagination, "}) {
    river_id
    name
    length
  }
}
")

# Get data and add it to the already created df
data<-rbind(data, get_from_graphQL(query=my_query, url="https://zendro.conabio.gob.mx/api/graphql"))

#end of loop
}
```

As a result you will get all the data in a single df:

```r
head(data)
```

| river_id | name | length |
|---|---|---|
| 1 | Acaponeta | 233 |
| 10 | Coatán | 75 |
| 11 | Coatzacoalcos | 325 |
| 12 | Colorado | 160 |
| 13 | Concepción | 335 |
| 14 | Culiacán | 875 |

```r
summary(data)
```
```
##    river_id             name               length
##  Length:50          Length:50          Min.   :  65.0
##  Class :character   Class :character   1st Qu.: 150.0
##  Mode  :character   Mode  :character   Median : 283.0
##                                        Mean   : 347.1
##                                        3rd Qu.: 402.5
##                                        Max.   :1521.0
##                                        NA's   :6
```

## `get_from_graphQL()` explained step by step

The following is a step-by-step example explaining in more detail how the `get_from_graphQL()` function used above works.

First, once you have a GraphQL query working, save it to an R object as a character vector:

```r
my_query<- "{
rivers(pagination:{limit:10, offset:0}){
      river_id
      name
      length
   }
}
"
```

Next, define as another character vector the url of the API, which is the same as the GraphiQL web interface you explored above:

```r
url<-"https://zendro.conabio.gob.mx/api/graphql"
```

Now we can query the API using a POST request:

```r
# query server
result <- POST(url, body = list(query=my_query), encode = c("json"))
```

The result we get is the `http` response. Before checking whether we got the data, it's good practice to verify the connection was successful by checking the status code — a `200` means all went well, any other code means trouble. See [this reference](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).

```r
# check server response
result$status_code
```
```
## [1] 200
```

We now need to extract the data to be able to manipulate it. If everything went well, the `http` response will contain a `data` attribute, which will itself contain an attribute named after the query — in this case, `rivers`.

```r
result
```
```
## Response [https://zendro.conabio.gob.mx/api/graphql]
##   Date: 2022-07-27 23:15
##   Status: 200
##   Content-Type: application/json; charset=utf-8
##   Size: 983 B
## {
##   "data": {
##     "rivers": [
##       {
##         "river_id": "1",
##         "name": "Acaponeta",
##         "length": 233
##       },
##       {
##         "river_id": "10",
## ...
```

If the query is not written properly, or if there's any other error, the `data` attribute won't exist, and instead we'll get an `errors` attribute listing the errors found.

If all went well, we can proceed to extract the content of the results with:

```r
# get data from query result
jsonResult <- content(result, as = "text")
```

The result will be in JSON format, which we can convert into an R object (a list). In this list, the results are nested within each type used in the query. The `flatten` argument collapses the list into a single data frame combining the data from different types.

```r
# transform to json
readableResult <- fromJSON(jsonResult,
                         flatten = T)
```

Extract the data:

```r
# get data
data<-as.data.frame(readableResult$data[1])
head(data)
```

| rivers.river_id | rivers.name | rivers.length |
|---|---|---|
| 1 | Acaponeta | 233 |
| 10 | Coatán | 75 |
| 11 | Coatzacoalcos | 325 |
| 12 | Colorado | 160 |
| 13 | Concepción | 335 |
| 14 | Culiacán | 875 |

By default, the name of each type is added at the beginning of each column name:

```r
colnames(data)
```
```
## [1] "rivers.river_id" "rivers.name"     "rivers.length"
```

To keep only the name of the variable as it is in the original data:

```r
x<-str_match(colnames(data), "\\w*$")[,1] # matches word characters (ie not the ".") at the end of the string
colnames(data)<-x # assign new colnames
```

So finally we have the data in a single nice-looking data frame:

```r
head(data)
```

| river_id | name | length |
|---|---|---|
| 1 | Acaponeta | 233 |
| 10 | Coatán | 75 |
| 11 | Coatzacoalcos | 325 |
| 12 | Colorado | 160 |
| 13 | Concepción | 335 |
| 14 | Culiacán | 875 |

Notice that you'll get a data frame like the one above only for one-to-one associations — in other cases you may still get variables that are lists, which you can process in a separate step.
