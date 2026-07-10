---
layout: default
title: Python API
parent: User guides
nav_order: 5
permalink: /guides/python-api
---
# How to use the Zendro API with Python to make changes to the data
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

If you're a user with CRUD credentials, you can use Zendro's GraphQL API to directly make changes to the data.

Using the Zendro API is mostly straightforward and similar to how we write queries in [GraphiQL](https://zendro.conabio.gob.mx/graphiql) (see [GraphQL basics]({% link guides/graphql-basics.md %})), but with some extra steps.

The first thing we need is a session token to authenticate our queries. We use the Python library `requests` for this. The steps to get a session token and use it with our requests:

1. Write the auth credentials in a dictionary, to use in the POST request.
2. Use `requests` to make the POST request to the endpoint that gives us the session token, passing the credentials dictionary in the `data` parameter. In this example the endpoint is [https://zendro.conabio.gob.mx/auth/realms/zendro/protocol/openid-connect/token](https://zendro.conabio.gob.mx/auth/realms/zendro/protocol/openid-connect/token).
3. If the request succeeds, get the session token from the response, create a `Session` object from `requests`, and update its `headers` to add the `Authorization` header with the session token as a Bearer token.

```python
import requests

# I used this library to hide the warnings for not verifying the ssl certificates
# of the url
import warnings
warnings.filterwarnings('ignore')


# credential for login to zendro
auth = {
    "username": "<username>",
    "password": "<password>",
    "client_id": "zendro_graphql-server",
    "grant_type": "password"
}

# make a post to a zendro-keycloak endpoint to retrieve session token
login = requests.post(
    "https://zendro.conabio.gob.mx/auth/realms/zendro/protocol/openid-connect/token",
    verify=False,
    data=auth
)

# if status code in the response is 200, then the request was successful and we have
# the session token we need in the login response
if login.status_code == 200:

    # we create a session object to use it for the requests to zendro api
    session = requests.Session()

    # and store the token we receive in the 'Authorization' header as a Bearer token
    session.headers.update({
        "Authorization": "Bearer " + login.json()["access_token"]
    })

    print("Successful login")
```

    Successful login

With a successful login, we can now make requests to the Zendro API. Word of advice: this token expires after 30 minutes.

The first query we'll make is a **mutation**. It creates a new country in the country table, and retrieves the country ID if the mutation was successful.

To write the query, we use Python's multiline string syntax, insert it into a dictionary as the value of key `"query"`, and pass it to `requests` via the `json` parameter.

All requests here are addressed to the Zendro API URL, in this example [https://zendro.conabio.gob.mx/api/graphql](https://zendro.conabio.gob.mx/api/graphql), declared as the first parameter of the `post` method.

To check whether the query ran correctly, we use the response object's `json` method.

```python
# we define the query to create a country
country_query = """
    mutation {
        addCountry(
            country_id:"JP",
            name: "Japan",
            population: 100000000,
            size: 377975
        ) {
            country_id
        }
    }
"""

# and using the session object we make a POST request to zendro api
# to create the country
country_response = session.post(
    "https://zendro.conabio.gob.mx/api/graphql", verify=False, json={"query": country_query}
)

country_response.json()
```

    {'data': {'addCountry': {'country_id': 'JP'}}}

Now let's create some cities and relate them to the country we just created. To create a city we use the `addCity` mutation, but if we use it more than once in a single request, each call needs a distinct name: `name: mutation_name()`, as shown below.

To relate a city to the previous country, we add the `addCountry` parameter with the country's ID.

```python
# we can group several queries in one request by adding it into the mutation braces
# but if two requests are of the same type, then we have to specify a name for each one
# using the syntax: 'name: mutation_name()' like in the example below
city_query = """
    mutation {
        osaka: addCity(
            city_id: 6,
            name: "Osaka",
            population: 2691000,
            addCountry: "JP"
        ) {
            city_id
        }
            suwon: addCity(
            city_id: 7,
            name: "Suwon",
            population: 1241000,
            addCountry: "JP"
        ) {
            city_id
        }
    }
"""

# and then we make the request as usual
city_response = session.post(
    "https://zendro.conabio.gob.mx/api/graphql", verify=False, json={"query": city_query}
)

city_response.json()
```

    {'data': {'osaka': {'city_id': '6'}, 'suwon': {'city_id': '7'}}}

We can also make a query request by changing `mutation` to `query`. This tells us — and Zendro — that this is a read query, not a mutation. Below, we check whether the previous mutation correctly associated the cities with the country.

```python
# to make a read request and not a mutation we change mutation to query
country_cities = """
    query {
        readOneCountry(country_id: "JP") {
            citiesFilter(pagination: { limit:10 }) {
                name
            }
        }
    }
"""

country_cities_response = session.post(
    "https://zendro.conabio.gob.mx/api/graphql", verify=False, json={"query": country_cities}
)

country_cities_response.json()
```

    {'data': {'readOneCountry': {'citiesFilter': [{'name': 'Osaka'}, {'name': 'Suwon'}]}}}

We can **update** entries in our table with mutations, specifying the ID of the entry to update. It's also helpful to retrieve the field you just updated in the response, to confirm the mutation ran correctly.

In the next example, we correct the population of Japan.

```python
# we can make changes to a table's entry using the specific type queries for that
# in this case we use 'updateCountry' to fix the population of Japan
update_country = """
    mutation {
        updateCountry(
            country_id: "JP",
            population: 125000000
        ) {
            name
            population
        }
    }
"""

update_response = session.post(
    "https://zendro.conabio.gob.mx/api/graphql", verify=False, json={"query": update_country}
)

update_response.json()
```

    {'data': {'updateCountry': {'name': 'Japan', 'population': 125000000}}}

Finally, to **delete** an entry, we first need to disassociate it from its relations — once an entry is no longer related to any other table, it can be deleted.

Earlier, we created the city of `Suwon`, which doesn't actually belong to Japan, so we first disassociate it from the country, then delete it by ID via `deleteCity`.

Delete mutations normally just return a string, instead of the fields of the targeted table.

```python
# to delete an entry, first we need to disassociate the entry of its relations
delete_city = """
    mutation {
        updateCity(
            city_id: 7,
            removeCountry: "JP"
        ) {
            city_id
            country_id
        }
        deleteCity(
            city_id: 7
        )
    }
"""
# the delete query returns a string
delete_response = session.post(
    "https://zendro.conabio.gob.mx/api/graphql", verify=False, json={"query": delete_city}
)

delete_response.json()

# {'data': {'updateCity': {'city_id': '7', 'country_id': None}, 'deleteCity': 'Item successfully deleted'}}
```

You can also pass [variables](https://graphql.org/learn/queries/#variables) to a given query, letting you define the query once and pass in dynamic parameters (search, order, pagination, ...) at run time.

For instance, [cursor-based-pagination.js](https://github.com/Zendro-dev/graphql-server-model-codegen/blob/master/test/unit_test_misc/test-describe/cursor-based-pagination.js), from Zendro's code generator test suite, uses a query with variables:

```js
let query = `
    query booksConnection($search: searchBookInput $pagination: paginationCursorInput! $order: [orderBookInput]) {
        booksConnection(search: $search pagination: $pagination order: $order) {
            edges{
                cursor
                node {
                    id
                    title
                    genre
                    publisher_id
                }
            }
            pageInfo {
                startCursor
                endCursor
                hasPreviousPage
                hasNextPage
            }
        }
    }
`
```

Here the variables are `search`, `pagination` and `order`, which can be passed dynamically. Sending the query and variables via `axios`:

```js
let response = await axios.post(
    remoteZendroURL,
    {
      query: query,
      variables: {search: search, order:order, pagination: pagination},
    },
    opts
);
```
