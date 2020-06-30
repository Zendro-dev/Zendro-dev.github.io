# Vocen

Vocen is a set of modular projects which objective is to provide tools for creation of a standarized platform which will allow the user to store, to manage and to retrieve data in a easy way. For this purpose, as the first step, data models  should be defined ([see more details here](setup_data_scheme.md)) to structure the user information, later this data models will be used as input for ScienceDb tools.

Vocen tools are mainly code generators which can be used with data models in a flexible way, meaning that each time the data model change then the code generator can re-generate the code accordingly.  

Vocen is conformed of two main components, backend and frontend. The backend component has its [base project](https://github.com/ScienceDb/graphql-server) and a [code generator](https://github.com/ScienceDb/graphql-server-model-codegen). The frontend of SPA(Single Page Application) also has its [base project](https://github.com/ScienceDb/single-page-app) and a [code generator](https://github.com/ScienceDb/single-page-app-codegen).
About how to use ScienceDB tools see tutorials below.

### TUTORIALS:

* [How to set up a full project](setup_root.md)
* [API documentation](api_root.md)

### REPOSITORIES:

* [GraphQL server](https://github.com/ScienceDb/graphql-server)
* [GraphQL server model generator](https://github.com/ScienceDb/graphql-server-model-codegen)
* [Single page application](https://github.com/ScienceDb/single-page-app)
* [Single page application model generator](https://github.com/ScienceDb/single-page-app-codegen)
