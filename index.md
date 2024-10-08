---
title: Home
layout: home
nav_order: 1
---

![Zendro logo](./figures/Zendro_logo_horizontal.png)

# Zendro

Zendro is a software tool to quickly create a data warehouse tailored to your specifications. You tell Zendro what the structure of your data is, in the form of models, and where the data is or shall be stored. Zendro will then automatically create two standardized interfaces for your data models. Both interfaces provide access to the standard CRUD, create, read, update, and delete functions, available for each of the defined data models. One of the two interfaces is an intuitive graphical browser based single page application implemented in Google's standard material design. The other is an exhaustive application programming interface built with Facebook's efficient GraphQL framework, enabling a connection to your data warehouse from any programming language or data analysis pipeline with utmost ease, simply by sending HTTP requests to your GraphQL server. Data can be distributed over several databases and servers without losing the relationships between your data records, even if they are not stored in the same place.

Zendro consists of two main components, backend and frontend. The backend component has its [base project](https://github.com/ScienceDb/graphql-server) and a [code generator](https://github.com/ScienceDb/graphql-server-model-codegen). The frontend of SPA (Single Page Application) also has its [base project](https://github.com/ScienceDb/single-page-app).
See the guides below on how to use Zendro.

To see or contribute to our code please visit Zendro-dev on [github](https://github.com/Zendro-dev), where you can find the repositories for:

* [GraphQL server](https://github.com/ScienceDb/graphql-server)
* [GraphQL server model generator](https://github.com/ScienceDb/graphql-server-model-codegen)
* [Single page application](https://github.com/ScienceDb/single-page-app)

If you have any questions or comments, please don't hesitate to contact us via an issue [here](https://github.com/Zendro-dev/Zendro-dev.github.io/issues). Tag your issue as a question or bug and we will try to answer as quick as possible.

## SHOW ME HOW IT LOOKS!

Would you like to see Zendro in action before deciding to learn more? That's fine! We set up a Zendro Instance for you to explore [Zendro's graphical user interface](https://brapi-gui.zendro-dev.org/) and [Zendro's API](https://brapi-graphql.zendro-dev.org/graphql). 

You can find tutorials on how to [use Zendro day to day here](https://zendro-dev.github.io/#using-zendro-day-to-day), so go there to start exploring.

### Installation and sysadmin

To start trying Zendro you can try the [Quickstart tutorial](https://zendro-dev.github.io/quickstart.html) on how to create a new Zendro project with pre-defined datamodels, database and environment variables. Then you can try the [Getting started tutorial](https://zendro-dev.github.io/setup_root.html), a step-by-step guide on how to create a new Zendro project from scratch, aimed at software developers and system administrators.

[![Go to Quickstart](./figures/quick.png)]({% link quickstart.md %})

[![Go to Getting started guide](./figures/gettingstarted.png)]({% link setup_root.md %})

For more sysadmin details also check:

### HOW-TO GUIDES:
* [How to use Zendro command line interface (CLI)]({% link zendro_cli.md %}). A tutorial of Zendro CLI, aimed at software developers.
* [How to setup Authentication / Authorization]({% link oauth.md %}). A concise guide on how to use and setup the Zendro authorization / authentication services.
* [How to setup a distributed cloud of Zendro nodes]({% link ddm.md %}). A brief guide, aimed at software developers and system administrators, on how to use Zendros distributed-data-models.
* [API documentation]({% link api_root.md %}). A summary of how Zendro backend generator implements a CRUD API that can be accessed through GraphQL query language.

### Defining data models
* [How to define data models: for developers]({% link setup_data_scheme.md %}). Detailed technical specifications on how to define data models for Zendro, aimed at software developers and system administrators.
* [How to define data models: for non-developers]({% link what_are_data_models.md %}). A brief, illustrated guide of data model specifications, data formatting and data uploading options, aimed at data modelers or managers to facilitate collaboration with developers.

### Using Zendro day to day
* [How to use Zendro's graphical interface]({% link SPA_howto.md %}). A full guide on how to use Zendro's graphical point and click interface. Aimed to general users and featuring lots of screenshots.
* [Introduction to GraphQL and querying the API]({% link GraphQL_intro.md %}). A friendly intro to how to perform GraphQL queries and use GraphiQL documentation.
* [How to query and extract data from R]({% link fromGraphQlToR.html %}). A concise guide on how to use the Zendro API from R to extract data and perform queries, aimed at data managers or data 
* [How to use the Zendro API with python to make changes to the data]({% link Zendro_requests_with_python.md %}). A concise guide on how to access the API using your user credentials to make CRUD operations on the data using python.

## Zendro users profiles

We designed Zendro to be useful for research teams and institutions that include users with different areas of expertise, needs and type of activities. The table below summarizes how we envision that different users will use Zendro:


| Profile                       | Background      | Expected use  |
|-------------------------------|-----------------|-----------------------|
| General user / scientist      | * Very experienced using Excel for data manipulation, visualization and basic data analysis. <br/> * Data producer. <br/> * Extensive domain knowledge. <br/> * No programming experience. | * Access Zendro SPA to see what data has been uploaded. <br/> * CRUD single records through the SPA or add several records through uploading a csv file. <br/> * Download data through the SPA. <br/> * Simple queries through the SPA. <br/> * Download data through the SPA.|
| Data scientist / data analyst | * Experienced using Excel for data manipulation, visualization and basic data analysis.<br/>* Experienced in data manipulation, visualization and analysis through programming languages like Python or R.<br/>* Some experience accessing and using APIs. | * Access Zendro instances to see what data have been uploaded.<br/>* Add and modify single records through the SPA.<br/>* Complex queries through the GraphQL API.<br/>* Download query results through the GraphQL API.<br/>* Connect external apps through the API (e.g. Shiny apps in R).|
| Data manager                  | * Experienced in data manipulation, visualization and analysis, mainly in programming languages like Python or R, but also in Excel.<br/>* Experienced in data standards.<br/>* Experienced in the management and oversight of an organization’s data lifecycle. | * Design data models (json) for new Zendro instances. <br/>*  Transform raw data to csv files according to specified data models and following Zendro’s data format requirements.<br/>*  Link users’ needs to technical solutions (ie, facilitate communication between general users or analysts, and developers). <br/>*  Download and manipulate data through the API. <br/>* CRUD data through the API, in batch. |
| Sysadmin                      | * Experienced in system administration and server maintenance. <br/>*  Experienced in designing, building and maintaining software and data services. <br/>*  Some experience in front-end and back-end development.| * Solve technical problems that arise in specific Zendro instances and Zendro in general. <br/>* Set up and configure new Zendro instances.<br/>* Customize back-end and front-end components for Zendro instances.<br/>* Check and confirm that data models and csv data files satisfy Zendro requirements and adjust them if necessary. <br/>*  Upload, delete and update data through the API in batch. <br/>*  Zendro maintenance. |

# CONTRIBUTIONS
Zendro is the product of a joint effort between the Forschungszentrum Jülich, Germany and the Comisión Nacional para el Conocimiento y Uso de la Biodiversidad, México, to generate a tool that allows efficiently building data warehouses capable of dealing with diverse data generated by different research groups in the context of the FAIR principles and multidisciplinary projects. The name Zendro comes from the words Zenzontle and Drossel, which are Mexican and German words denoting a mockingbird, a bird capable of “talking” different languages, similar to how Zendro can connect your data warehouse from any programming language or data analysis pipeline.

### Zendro contributors in alphabetical order
Francisca Acevedo<sup>1</sup>, Vicente Arriaga<sup>1</sup>, Vivian Bass<sup>1</sup>, Katja Dohm<sup>3</sup>, Jaime Donlucas<sup>1</sup>, Constantin Eiteneuer<sup>2</sup>, Sven Fahrner<sup>2</sup>, Frank Fischer<sup>4</sup>, Asis Hallab<sup>2</sup>, Alicia Mastretta-Yanes<sup>1</sup>, Roland Pieruschka<sup>2</sup>, Erick Palacios-Moreno<sup>1</sup>, Alejandro Ponce<sup>1</sup>, Yaxal Ponce<sup>2</sup>, Francisco Ramírez<sup>1</sup>, Irene Ramos<sup>1</sup>, Franz-Eric Sill<sup>2</sup>, Bernardo Terroba<sup>1</sup>, Tim Rehberg<sup>3</sup>, Ulrich Schurr<sup>2</sup>, Verónica Suaste<sup>1</sup>, Björn Usadel<sup>2</sup>, David Velasco<sup>2</sup>, Thomas Voecking<sup>3</sup> and Dan Wang<sup>2</sup>

#### Author affiliations
1. CONABIO - Comisión Nacional para el Conocimiento y Uso de la Biodiversidad, México
2. Forschungszentrum Jülich, Germany
3. Auticon - www.auticon.com
4. InterTech - www.intertech.de

#### Author contributions
Conceptualization, management and coordination of the project was done by Asis Hallab and Alicia Mastretta-Yanes. Software design was done by Asis Hallab. Programming, implementation and testing of the computer code was done by Vivian Bass, Katja Dohm, Constantin Eiteneuer, Asis Hallab, Francisco Ramírez, Tim Rehberg, Franz-Eric Sill, Veronica Suaste, David Velasco, Thomas Voecking and Dan Wang. Use case definitions were provided by Frank Fischer, Roland Pieruschka, Irene Ramos, and Björn Usadel. Acquisition of the financial support for the project was contributed by Francisca Acevedo, Vicente Arriaga and Björn Usadel. User experience and application of Zendro on data management projects was carried out by Vivian Bass, Jaime Donlucas, Asis Hallab, Alicia Mastretta-Yanes, Erick Palacios Moreno, Alejandro Ponce, Yaxal Ponce, Irene Ramos, Verónica Suaste and David Velasco. Writing the original draft of the manuscript and software documentation was done by Vivian Bass, Constantin Eiteneuer, Asis Hallab, Alicia Mastretta-Yanes, Irene Ramos, Franz-Eric Sill, Verónica Suaste and Dan Wang. Logo desing was done by Bernardo Terroba.

### Funding
On the Mexican side, Zendro was developed with resources of the Global Environmental Fund though the Mexican Agrobiodiversity Project (GEF Project ID: 9380). On the German side Zendro was developed with resources of the projects [german grants here].