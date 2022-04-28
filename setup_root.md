[ &larr; back](README.md)
<br/>
# Getting started

This is a step-by-step guide on how to create a new Zendro project from scratch, aimed at software developers and system administrators.

Zendro consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app-codegen__ and __single-page-app__. The first pair is responsible for the back-end [GraphQL](https://graphql.org/learn/) service that can be accessed on the default port 3000. The second pair of projects acts as a client of the GraphQL server and creates a simple generic web-based GUI for this server on the 8080 port. As the names of these projects say, the *codegen* suffix means that to pull up the corresponding server it is required to generate some code first. Among inter-database transparency, the code generation step also allows a Zendro user to automate routinary ORM programming that depends on data scheme.

 <br/>

## Project Requirements:
 * [NodeJS](https://nodejs.org/en/) We strongly recommend to install NodeJS v14.17.6 

 **recommended for setting up zendro using docker**
 * [docker](https://docs.docker.com/get-docker/)
 * [docker-compose](https://docs.docker.com/compose/install/#install-compose)
 <br/><br/>

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

The easiest way to set up Zendro is using the [Zendro CLI tool](https://github.com/Zendro-dev/zendro). With minimal steps and configuration a Zendro warehouse taylored to your data needs can be deployed. Execute:

```
# "-d" adds Dockerfiles to fully dockerize running zendro 
zendro new -d <my-project-name> 
```

### Step 3: Edit environment variables

If you want to know more about enviroment variables you can check [this](env_vars.md)

### Step 4: Define your data models

### Step 5: Start up Zendro 

## Using docker
* Indicar cambios que deben hacerse si se quieren modificar los puertos 
## Without docker
* Indicar los cambios que debe haber en las variables de entorno y de donde obtenerlas en keycloak, quizá capturas de pantalla?

### Step 6: Start up Zendro with access control 


### Futuro: cómo hacerlo en un servidor remoto



* * *

### Step 3 (required): Configure and generate your data models

After setting up your data models use the Zendro CLI to generate the model-specific code and fill your Zendro skeleton project with life.

```
zendro generate
```

This will automatically generate all basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step. Also have a look at how to generate and use [migrations](https://zendro-dev.github.io/zendro_cli.html#example-for-migrations) using the CLI tool.

You should modify environment variables and database configurations according to your needs.

### Step 4: Start up your Zendro instance

The recommend way to [run your Zendro instance is via docker](https://zendro-dev.github.io/zendro_cli.html#dockerize-zendro-app-with-example-docker-files). This ensures that regardless of your local infrastructure Zendro will behave the same.
```
zendro dockerize -u
```
It is also possible to [run the zendro services locally](https://zendro-dev.github.io/zendro_cli.html#start-zendro-service), however you might run into unexpected incompatibilities depending on your local system.
```
zendro start
```

### Step 5 (optional): Customize your project

A couple of basic extensions are suggested to be introduced directly into the GraphQL server code. These are: *data validation logic* and *GraphQL query/mutation patches*. To implement custom logic of these extensions some programming is required, however this step is well confined and described.  

Furthermore the whole codebase used to run zendro is exposed an can be directly customized if needed. That is true for the graphql-server as well as the frontend applications.

[ > Advanced code customizing](setup_customize.md)
