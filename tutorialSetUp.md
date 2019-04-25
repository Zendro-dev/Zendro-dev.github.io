[ &larr; back](README.md)
<br/>
# A new ScienceDB project from scratch

ScienceDB consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app-codegen__ and a __single-page-app__. The first pair is responsible for the back-end [GraphQL](https://graphql.org/learn/) service that can be accessed on the default port 3000. The second pair of projects act as a client of the GraphQL server and create a simple generic web-based GUI for this server on the 8080 port. As it can be derived from the names of these projects, the *codegen* suffix means that to pull up corresponding server it is required to generate some code first. Among inter-database transparency, code generation step also permits for ScienceDB user to minimize routinary ORM programming that depends on data scheme.

 <br/>
 
 __Project Requirements:__
 * [NodeJS](https://nodejs.org/en/)
 * [NPM](https://www.npmjs.com/get-npm)
 * [Vue CLI](https://cli.vuejs.org/)
 <br/><br/>

* * *
* _**Step 1 (required): Data scheme definition**_

To generate a model-dependent code that is initially missing within the servers it is necessary to define your data scheme first. The data scheme can be defined through a set of data models (tables). 
 <br/>
 
 [ > Data Models](dataModels.md)
<br/><br/>
* _**Step 2 (required): Setup backend GraphQL server**_

The GraphQL server by default provides basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step. To start up with this server first it is needed to generate scheme-dependent code into the server root folder. Afterwards you can start the server and access it's console from your web browser at `http://<your_host>:3000/graphql/`. 
<br/>

[ > Backend Setup](backendSetUp.md)
<br/><br/>
* _**Step 3 (optional): Setup Web Interface application**_

At least for the first time it is recommended to install generic WebBased GUI server that permits to manage your data in a simple user-friendly web interface. Here we purpose the Single Page Application (SPA) approach where all HTML views are constructed on the client side using VueJS framework. An application would communicate with GraphQL server using regular POST requests. As a rule, all operations that are accessible from GUI are also supported on the GraphQL server console directly.

Likewise as in the GraphQL server it will be required to generate a data scheme dependent code into the web server folder, and run the server afterwards. By default web server page is accessible at `http://<your_host>:8080/`. 
<br/>

[ > Web GUI Setup](guiSetUp.md)
<br/><br/>
* _**Step 4 (optional): Customizing your project**_ 

 A couple of basic extensions are suggested to be introduced directly into the GraphQL server code. These are: *access permissions*, *data validation logic* and *custom GraphQL queries/mutations*. To introduce the logic of these extensions it is suggested some programming, however this step is well confined and described.
 <br/>
 
 [ > Advanced code customizing](projectCustomizing.md)



