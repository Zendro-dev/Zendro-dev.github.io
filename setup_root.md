[ &larr; back](README.md)
<br/>
# A new Zendro project from scratch

Zendro consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app-codegen__ and __single-page-app__. The first pair is responsible for the back-end [GraphQL](https://graphql.org/learn/) service that can be accessed on the default port 3000. The second pair of projects acts as a client of the GraphQL server and creates a simple generic web-based GUI for this server on the 8080 port. As the names of these projects say, the *codegen* suffix means that to pull up the corresponding server it is required to generate some code first. Among inter-database transparency, the code generation step also allows a Zendro user to automate routinary ORM programming that depends on data scheme.

 <br/>

## Project Requirements:
 * [NodeJS](https://nodejs.org/en/)  

 **recommended for setting up zendro using docker**
 * [docker](https://docs.docker.com/get-docker/)
 * [docker-compose](https://docs.docker.com/compose/install/#install-compose)
 <br/><br/>

* * *
## Step-by-Step guide
### Step 1 (required): Setup a new Zendro project

The easiest way to set up Zendro is using the [Zendro CLI tool](https://github.com/Zendro-dev/zendro). With minimal steps and configuration a Zendro warehouse taylored to your data needs can be deployed.

Follow the instructions on how to install and use the CLI to setup a new _skeleton_ project for your Zendro instance.
```
zendro new -d <my-project> # "-d" adds Dockerfiles to fully dockerize running zendro 
```

[> Zendro CLI](zendro_cli.md)


### Step 2 (required): Define your data models

To generate model-dependent code that is initially missing within both the graphql-server as well as the single-page-application it is necessary to define your data scheme first. The data scheme can be defined through a set of data models (it is possible to imagine each data model as a separate table in a relation DB). In case you used the CLI (_Step 1_) use the `data_model_definitions` folder in the root directory of your project to define your models.
 <br/>

 [ > Data Models](setup_data_scheme.md)
<br/><br/>

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
