---
layout: default
title: Getting started
nav_order: 4
has_children: true
---
# Getting started
{: .no_toc }

This is a step-by-step guide on how to create a new Zendro project from scratch, aimed at software developers and system administrators.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
Zendro consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app__ and __graphiql-auth__. The first pair is responsible for the back-end [GraphQL](https://graphql.org/learn/) service that can be accessed on the default url `http://localhost:3000/graphql`. To pull up the corresponding server it is required to generate some code first. The third project acts as a client of the GraphQL server and creates a simple generic web-based GUI for this server on the url `http://localhost:8080`. The last project offers a Zendro specific implementation of the browser based GraphQL IDE [Graphiql](https://github.com/graphql/graphiql). The project is a simple [Next.js](https://nextjs.org/) application. Custom adjustments have been made to accommodate Zendro requirements for authentication of users and enhanced meta searches using [jq](https://stedolan.github.io/jq/) or [JSONPath](https://goessner.net/articles/JsonPath/) statements.

## Project Requirements:
 * [NodeJS](https://nodejs.org/en/) version 16+ is required.

 **recommended for setting up zendro using docker**
 * [docker](https://docs.docker.com/get-docker/)
 * [docker-compose](https://docs.docker.com/compose/install/#install-compose)

* * *
## Step by step guide

### Step 1: Installation

Execute this commands to install Zendro:

```
$ git clone https://github.com/Zendro-dev/zendro.git
$ cd zendro
$ npm install
$ npm link
```

### Step 2: Setup a new Zendro project

The easiest way to set up Zendro is using the [Zendro CLI tool](https://github.com/Zendro-dev/zendro). With minimal steps and configuration a Zendro warehouse taylored to your data needs can be deployed. 

Go out from the previusly created `zendro` directory 

```
$ cd ..
```

and execute:

```
# "-d" adds Dockerfiles to fully dockerize running zendro 
$ zendro new -d <my-project-name>  
```

<!----><a name="envvars"></a>

### Step 3: Edit environment variables

Go inside the new project and modify the selected enviroment variables in the next files. These files have a default configuration, please remember to add your expected secret word in the *NEXTAUTH_SECRET* variable.

* **Without docker setup:** ./graphql-server/config/data_models_storage_config.json
* **With docker setup:** ./config/data_models_storage_config.json
* **SPA in development mode:** ./single-page-app/.env.development
* **SPA in production mode:** ./single-page-app/.env.production
* **GraphiQL in development mode:** ./graphiql-auth/.env.development
* **GraphiQL in production mode:** ./graphiql-auth/.env.production

  If you would like to upload a file to a remote server, please consider the template *.env.migration.sample*, create a new file *.env.migration* and modify relevant environment variables.

If you wish to know more about enviroment variables you can check [this]({% link env_vars.md %}).

**Note**: The `data_models_storage_config.json` file defines the connections to the databases that zendro should be aware of. Currently we have integrated sequelize, neo4j, cassandra, amazon s3, mongodb. For other storage types, user needs to implement the connection part and add a [new storage type]({% link setup_customize.md %}#add-a-new-storage-type), or use generic models. If user wants to use other supported storage types, then user can reuse part of two example files, which illustrate the configuration of all supported storage types with docker setup, namely `./config/data_models_storage_config_example.json` and `./docker-compose-dev-example.yml`.

### Step 4: Define your data models

Add your model definitions in JSON files to `./data_model_definitions` folder. 

If you want to learn more about how to define data models with Zendro, please check [this]({% link setup_data_scheme.md %}). 

Note: by default, indices would be generated for *internalId*. And it is recommended to add indices for attributes which are foreign keys. See the [json specs]({% link setup_data_scheme.md %}#json-specs) for more information.

### Step 5: Generate code and migrations

After setting up your data models use the next command to generate the model-specific code and fill your Zendro skeleton project with life. 

Inside the new project execute:

```
$ zendro generate -m
```

This will automatically generate all basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step.

Also, this will create migration files. By default, every migration file has two functions, namely **up** and **down**. The **up** function creates a table, the **down** function deletes the existing table. Additionally, **up** and **down** functions would add and drop indices for primary keys. Furthermore it is possible to customize the migration functions. Please check [this]({% link zendro_cli.md %}) to learn more about migrations.

### Step 6 (optional): Define your database connection

In case you want to connect Zendro to an already existing database, maybe already populated with data, there is no need for Zendro to run any migrations to create the necessary infrastructure. Leave out the `-m` flag in the above `zendro generate` command.

To define the connection to your local database edit the `config/data_models_storage_config.json` configuration file. You can reference a name given to a connection in your data-model-definition via the *database* attribute. See the [json specs]({% link setup_data_scheme.md %}#json-specs) of data-model-definitions for more information.

In this file you can define any number of database connections, that zendro should be aware of. Depending on the driver used a set of attributes is expected. Details can be found in the [default storage configuration for each storage type](https://github.com/Zendro-dev/graphql-server/blob/master/config/data_models_storage_config.json).

### Step 7: Start up Zendro 

<!----><a name="start"></a>

> ***➡ USING DOCKER***

The recommend way to [run your Zendro instance is via docker]({% link zendro_cli.md %}#dockerize-zendro-app-with-example-docker-files). This ensures that regardless of your local infrastructure Zendro will behave the same.

Execute this command to start Zendro in production mode or without `-p` to start in development mode. 

```
$ zendro dockerize -u -p
```

This command will create docker containers for each Zendro component:
* [Keycloak]({% link oauth.md %}): manage users and roles
* [Single Page App (SPA)](https://github.com/Zendro-dev/single-page-app): graphical interface to send CRUD requests to a Zendro GraphQL endpoint
* [API](https://github.com/Zendro-dev/graphql-server): CRUD API that can be accessed through a GraphQL query language
* [GraphiQL interface](https://github.com/Zendro-dev/graphiql-auth): An implementation of the GraphQL IDE with Zendro login and advanced filter functionalities.
* [traefik reverse-proxy](https://doc.traefik.io/traefik/): A reverse-proxy using traefik that maps the above docker services.

You can check docker containers by:
```
$ docker ps
```

You can check docker logs by:
```
$ docker logs -f <container name>
```

> ***Please wait until logs indicate the app is running on XXXX port to access Zendro services.***

In default config, the running containers will be:

* Keycloak: 
    * http://localhost:8081/auth
    
      * The default keycloak username is *admin* and the password is *admin*.

  ![Keycloak example](figures/kc1.png)
  ![Keycloak example](figures/kc2.png)

* SPA: 
    * http://localhost:8080

      * The default zendro username is *zendro-admin* and the password is *admin*.

  ![spa example](figures/login.png)
  ![spa example](figures/spa.png)

* GraphQL API: 
    * http://localhost:3000/graphql

  ![api example](figures/graphql.png )

* GraphiQL interface with filter functionality: 
    * http://localhost:7000

      * The default zendro username is *zendro-admin* and the password is *admin*.

  ![api example](figures/login.png)
  ![api example](figures/graphiql.png)


If you wish to modify the default ports, adjust the [enviroment variables](#envvars) in the next files:
> Hint: [Stop Zendro Instance](#stop), modify files and [start Zendro Instance](#start) again.

* ./docker-compose-prod.yml
* ./docker-compose-dev.yml
* ./single-page-app/.env.production
* ./single-page-app/.env.development
* ./graphql-server/.env
* ./graphiql-auth/.env.development
* ./graphiql-auth/.env.production

Also, if you wish to modify docker containers name or docker services names, adjust the following files:
> Hint: [Stop Zendro Instance](#stop), modify files and [start Zendro Instance](#start) again.

* ./docker-compose-prod.yml
* ./docker-compose-dev.yml

---

 ***➡ WITHOUT DOCKER***

If you prefer to use local setup with Keycloak, there are a few things to do after running Zendro:

***Requirements***

* Install [Java](https://www.java.com/en/) (from Java 11 forward).
* Install [keycloak](https://www.keycloak.org). We recommend Keycloak 18+. 
  * Go to https://www.keycloak.org/downloads and download *Distribution powered by Quarkus*.
  * After unzip, copy the keycloak configuration file from `zendro/test/env/keycloak.conf` to `keycloak/conf/keycloak.conf`.
  * Two enviroment variables should be configured through command line. In terminal inside keycloak folder execute:
    ```
    $ export KEYCLOAK_ADMIN=admin
    $ export KEYCLOAK_ADMIN_PASSWORD=admin
    ```
  * Start keycloak in dev mode executing `$ ./bin/kc.sh start-dev` in keycloak folder. 
  * Go to http://localhost:8081 to see keycloak running. The keycloak username is *admin* and the password is *admin*.


    * Zendro realm configuration will be done when the migration file is executed after zendro starts.
    <br/><br/>


  * In order to get zendro and keycloak running, you have to do some modifications in your zendro new project `.env` files. 

    * ./single-page-app/.env.production and ./single-page-app/.env.development
    ```
    NEXT_PUBLIC_ZENDRO_ROLES_URL="http://localhost:3000/getRolesForOAuth2Token"
    OAUTH2_ISSUER='http://localhost:8081/realms/zendro'
    OAUTH2_TOKEN_URI='http://localhost:8081/realms/zendro/protocol/openid-connect/token'
    OAUTH2_AUTH_URI='http://localhost:8081/realms/zendro/protocol/openid-connect/auth'
    ```
    * ./graphiql-auth/.env.development and ./graphiql-auth/.env.production
    ```
    OAUTH2_ISSUER='http://localhost:8081/realms/zendro'
    OAUTH2_TOKEN_URI='http://localhost:8081/realms/zendro/protocol/openid-connect/token'
    OAUTH2_AUTH_URI='http://localhost:8081/realms/zendro/protocol/openid-connect/auth'
    ```

    * ./graphql-server/.env
    ```
    OAUTH2_TOKEN_URI="http://localhost:8081/realms/zendro/protocol/openid-connect/token"
    ```

* Start zendro 

  **Development mode**

    ```
    $ zendro start
    ```
    > ***Please wait until logs indicate the app is running on XXXX port to access Zendro services.***

    In default config, zendro services will be on ports:
    * API - http://localhost:3000/graphql
    * GraphiQL - http://localhost:7000
    * Single Page App (SPA) - http://localhost:8080
    * Keycloak - http://localhost:8081/


  **Production mode**
    * Copy the content of `./graphiql-auth/.env.development` to `./graphiql-auth/.env.production`
    * Copy the content of `./single-page-app/.env.development` to `./single-page-app/.env.production`
    * Modify the `OAUTH2_TOKEN_URI` env var in `./graphql-server/.env`:
    `OAUTH2_TOKEN_URI="http://localhost:8081/realms/zendro/protocol/openid-connect/token"`
  * Start
      ```
      $ zendro start -p
      ```
    > ***Please wait until logs indicate the app is running on XXXX port to access Zendro services.***

    In default config, zendro services will be on ports:
      * API - http://localhost:3000/graphql
      * GraphiQL - http://localhost:7000
      * Single Page App (SPA) - http://localhost:8080
      * Keycloak - http://localhost:8081
  
    
* You can find applications logs on `./logs/graphiql.log`, `./logs/graphql-server.log` and `./logs/single-page-app.log`.

<!----><a name="important"></a>
>
> **Important**: If you have problems to connect zendro with keycloak service, we recommend you to modify http://localhost:8081 to http://0.0.0.0:8081 in the next .env files:
>
>  * ./single-page-app/.env.production
>  * ./single-page-app/.env.development
>  * ./graphiql-auth/.env.development 
>  * ./graphiql-auth/.env.production
>  * ./graphql-server/.env


### Step 8: Start up Zendro with access control 
Zendro can be used checking access rights for every single GraphQL query received by the currently logged in user identified by the Token. The user is decoded and the corresponding roles are loaded to check access rights. This step is carried out by the [NPM acl package](https://www.npmjs.com/package/acl). Respective access rights can and must be declared in the file `./graphql-server/acl_rules.js`.

You can run Zendro with or without this access control check. The default is to run it without checking access rights.

To switch access right check on, you must uncomment the command line switch acl and change the following line in `./graphql-server/migrateDbAndStartServer.sh`

```
npm start #acl
```
to 
```
npm start acl
```

Moreover, you can whitelist certain roles, which can own all user permissions, by using *WHITELIST_ROLES* environment variable. For example, if you wish to whitelist reading actions, modify in `./graphql-server/.env`:

```
WHITELIST_ROLES="reader"
```

You can add all roles you wish separating them with a comma.


<!----><a name="stop"></a>
### Step 9: Stop Zendro instance

 ***➡ USING DOCKER***

Execute this command to stop Zendro. To remove all volumes use `-v`.
```
$ zendro dockerize -d -v
```

If you are on production mode execute:
```
$ zendro dockerize -d -p -v
```
---

 ***➡ WITHOUT DOCKER***
 
Execute this command to stop Zendro if you are on production mode:
```
$ zendro stop -p
```

Execute the next command to stop Zendro if you are on development mode:
```
$ zendro stop
```

### Customize your project (optional)

A couple of basic extensions are suggested to be introduced directly into the GraphQL server code. These are: *data validation logic* and *GraphQL query/mutation patches*. To implement custom logic of these extensions some programming is required, however this step is well confined and described.  

Furthermore, the whole codebase used to run zendro is exposed and can be directly customized if needed. That is true for the graphql-server as well as the frontend applications.

[ > Advanced code customizing]({% link setup_customize.md %})

### Add empty or default plots (optional)

Empty or default plots could be generated via zendro CLI. Please see the instruction [here]({% link zendro_cli.md %}#plots)

* * *
## Updating Zendro

To update Zendro enter your zendro folder and execute:

```
$ git pull
$ npm install
```

* * *
## Uninstallation

### Remove Project

Execute the following to remove a project:

```
$ rm -r "path/to/<name>"
$ docker rmi -f $(docker images -a -q "<name>*")
```

### Uninstall Zendro

To uninstall Zendro, execute the following:

```
$ sudo npm unlink -g zendro
$ sudo rm -r "path/to/zendro"
```
