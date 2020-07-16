
![Zendro logo](figures/Zendro_logo_horizontal.png)

# Zendro

Zendro is a software tool to quickly create a data warehouse tailored to your specifications. You tell Zendro what the structure of your data is, in the form of models, and where the data is or shall be stored. Zendro will then automatically create two standardized interfaces for your data models. Both interfaces provide access to the standard CRUD, create, read, update, and delete functions, available for each of the defined data models. One of the two interfaces is an intuitive graphical browser based single page application implemented in Google's standard material design. The other is an exhaustive application programming interface built with Facebook's efficient GraphQL framework, enabling a connection to your data warehouse from any programming language or data analysis pipeline with utmost ease, simply by sending HTTP requests to your GraphQL server. Data can be distributed over several databases and servers without losing the relationships between your data records, even if they are not stored in the same place.

Zendro consists of two main components, backend and frontend. The backend component has its [base project](https://github.com/ScienceDb/graphql-server) and a [code generator](https://github.com/ScienceDb/graphql-server-model-codegen). The frontend of SPA (Single Page Application) also has its [base project](https://github.com/ScienceDb/single-page-app) and a [code generator](https://github.com/ScienceDb/single-page-app-codegen).
See the guides below on how to use Zendro.

### HOW-TO GUIDES:

* [How to set up a Zendro instance](setup_root.md). A step-by-step guide on how to create a new Zendro project from scratch, aimed at software developers and system administrators.
* [How to define data models: for developers](setup_data_scheme.md). Detailed technical specifications on how to define data models for Zendro, aimed at software developers and system administrators.
* [How to define data models: for non-developers](non-developer_documentation.md). A brief, illustrated guide of data model specifications, data formatting and data uploading options, aimed at data modelers or managers to facilitate collaboration with developers.
* [How to query and extract data](fromGraphQlToR.html). A concise guide on how to use the Zendro API from R to extract data and perform queries, aimed at data managers or data scientists.
* [API documentation](api_root.md).

### REPOSITORIES:

* [GraphQL server](https://github.com/ScienceDb/graphql-server)
* [GraphQL server model generator](https://github.com/ScienceDb/graphql-server-model-codegen)
* [Single page application](https://github.com/ScienceDb/single-page-app)
* [Single page application model generator](https://github.com/ScienceDb/single-page-app-codegen)
