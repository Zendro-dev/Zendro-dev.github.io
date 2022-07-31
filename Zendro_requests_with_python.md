# How to use the Zendro API with python to make changes to the data

If you are a user with CRUD credentials you can use Zendro GraphQL API to directly make changes to the data. 

Using the Zendro API its mostly straightforward and similar to how we write queries in [GraphiQL](https://zendro.conabio.gob.mx/graphiql) (check the [GraphQL_intro.md](GraphQL_intro.md)), but with some extra steps.

The first thing we need to do is to get a session token so we can authenticate our queries. To achieve this, we use the python library `requests`. The steps to get a session token and use it with our requests are the following:

1. First we need to write the auth credentials in dictionary to make a successful login, we are going to use those credentials in the POST request.
2. Now we use the library requests to make the POST, specifying the url that gives us the session token we need. In this example this is: [https://zendro.conabio.gob.mx/auth/realms/zendro/protocol/openid-connect/token](https://zendro.conabio.gob.mx/auth/realms/zendro/protocol/openid-connect/token). We also need to pass the dictionary we created earlier in the data parameter.
3. If the request was successful we can obtain the session token from the response, then we create a Session object from the requests library, and update the 'headers' to add the 'Authorization' header with the session token as a Bearer token.


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


With a successful login we can now make the requests to the Zendro API. The first query we are going to make, is a **mutation**. In this mutation we are going to create a new country in the country table, and we are going to retrieve the country id if the mutation was successful. 

To write the query we can use the multiline syntax of python, then we insert this string in a dictionary as a value to the key "query", and we pass it to the requests in the json parameter. 

All the requests we are going to do are addressed to the Zendro API url, which in this example is [https://zendro.conabio.gob.mx/api/graphql](https://zendro.conabio.gob.mx/api/graphql), and it is declared in the first parameter of the post method.

To see if the query ran correctly, we can use the method `json` of the response object.


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



Now we are going to create some cities and relate these cities with the country we just created above. To create a city we need to use a specific type for it, which is `addCity`, but if we intend to use it more than once in a single request, then we need to distinguish them with a different name for each one. To specify a name for our mutations we write it like: `name: mutation_name()`, as in the example below. 

To relate the city to our previous country, we add the `addCountry` param, with the country's id as the value.


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



We can also make a query request, by changing the word `mutation` to `query`. This way we, and also Zendro, knows that the query between the braces is going to be a read query and not a mutation. In the example below we are checking if the previous mutation correctly associated the cities with the country.


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




    {'data': {'readOneCountry': {'citiesFilter': [{'name': 'Osaka'},
        {'name': 'Suwon'}]}}}



We can **update the entries in our table with mutations, for this we need to specify the id of the entry we are going to update. It is also helpful to retrieve in the response the field we just update so we know the mutation ran correctly.

In the next example we update the field  `population` to correct the population of Japan.


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



Finally to **delete** an entry, first we need to disassociate it from its relations, once our entry is no longer related to any other table, we can delete it. 

In the first examples, when we created our cities, we create the city of `Suwon`, which doesn't belong to Japan, so we first disassociate it to the country, and then we can delete it, by typing  its id in the `deleteCity` mutation.

The delete mutations normally just return a string, instead of the fields of the table we are targeting.


```python

# to delete an entry, first we need to disassociate the entry of its relations
delete_city = """
    mutation {
          updateCity(
            city_id: 7,
            removeCountry: "JP"
          ) { city_id country_id }
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
```




    {'data': {'updateCity': {'city_id': '7', 'country_id': None},
      'deleteCity': 'Item successfully deleted'}}


