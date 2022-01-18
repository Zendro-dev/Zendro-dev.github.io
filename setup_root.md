[ &larr; back](README.md)
<br/>
# A new Zendro project from scratch

Zendro consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app-codegen__ and __single-page-app__. The first pair is responsible for the back-end [GraphQL](https://graphql.org/learn/) service that can be accessed on the default port 3000. The second pair of projects acts as a client of the GraphQL server and creates a simple generic web-based GUI for this server on the 8080 port. As the names of these projects say, the *codegen* suffix means that to pull up the corresponding server it is required to generate some code first. Among inter-database transparency, the code generation step also allows a Zendro user to automate routinary ORM programming that depends on data scheme.

 <br/>

### Project Requirements:
 * [NodeJS](https://nodejs.org/en/)
 **recommended for setting up zendro using docker**
 * [docker](https://docs.docker.com/get-docker/)
 * [docker-compose](https://docs.docker.com/compose/install/#install-compose)
 <br/><br/>

* * *
* _**Step 1 (required): Setup a new Zendro project**_

The easiest way to set up Zendro is using the [Zendro CLI tool](). With minimal steps and configuration a Zendro warehouse taylored to your data needs can be deployed.

Follow the instruction on how to install and use the CLI to setup a new _skeleton_ project for your Zendro instance.
```
zendro new >my-project>
```

[ > Zendro CLI](zendro_cli.md)


* _**Step 2 (required): Define your data models**_

To generate model-dependent code that is initially missing within both the graphql-server as well as the SPA it is necessary to define your data scheme first. The data scheme can be defined through a set of data models (it is possible to imagine each data model as a separate table in a relation DB). In case you used the CLI (_Step 1_) use the `data_model_definitions` folder in the root directory of your project.
 <br/>

 [ > Data Models](setup_data_scheme.md)
<br/><br/>

* _**Step 3 (required): Configure and generate your data models**_

After setting up your data models use the Zendro CLI to generate the model-specific code and fill your Zendro skeleton project with life.

```
zendro generate
```

This will automatically generate all basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step. Also have a look at how to generate and use [migrations]() using the CLI tool.

You should modify environment variables and database configurations according to your needs.

* _**Step 4: Start up your Zendro instance**_

The recommend way to run your Zendro instance is via docker. This ensures that regardless of your local infrastructure Zendro will behave the same.  
It is also possible to run the zendro services locally, however you might run into unexpected incompatibilities depending on your local system.


* _**Step 3 (required): Setup backend GraphQL server**_

The GraphQL server by default provides basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step. To run this server first, it is needed to generate scheme-dependent code into the server root folder. Afterwards you can start the server and access it's console from your web browser at `http://<your_host>:3000/graphql/`.
<br/>

[ > Back-end Server Setup](setup_backend.md)
<br/><br/>
* _**Step 3 (optional): Setup Web Interface application**_

At least for the first time it is recommended to install generic WebBased GUI server that permits to manage your data in a simple user-friendly web interface. Here we purpose a Single Page Application (SPA) approach where all HTML views are constructed on the client side using VueJS framework. This application communicates with the GraphQL server using regular POST requests. It is a rule that all operations accessible within this generic GUI are also accessible on the back-end server directly.

Likewise as in the GraphQL server, it will be required to generate a data scheme dependent code into the web server folder and run the server afterwards. By default web server page is accessible at `http://<your_host>:8080/`.
<br/>

[ > Web GUI Server Setup](setup_gui.md)
<br/><br/>
* _**Step 4 (optional): Customizing your project**_

 A couple of basic extensions are suggested to be introduced directly into the GraphQL server code. These are: *data validation logic* and *GraphQL query/mutation patches*. To implement custom logic of these extensions some programming is required, however this step is well confined and described.
 <br/>

 [ > Advanced code customizing](setup_customize.md)
