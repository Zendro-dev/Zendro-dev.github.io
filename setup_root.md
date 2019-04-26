[ &larr; back](README.md)
<br/>
# A new ScienceDB project from scratch

ScienceDB consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app-codegen__ and __single-page-app__. The first pair is responsible for the back-end [GraphQL](https://graphql.org/learn/) service that can be accessed on the default port 3000. The second pair of projects act as a client of the GraphQL server and create a simple generic web-based GUI for this server on the 8080 port. As it can be derived from the names of these projects, the *codegen* suffix means that to pull up corresponding server it is required to generate some code first. Among inter-database transparency, code generation step also permits for ScienceDB user to automate routinary ORM programming that depends on data scheme.

 <br/>
 
 ###Project Requirements:
 * [NodeJS](https://nodejs.org/en/)
 * [NPM](https://www.npmjs.com/get-npm)
 * [Vue CLI](https://cli.vuejs.org/)
 <br/><br/>

* * *
* _**Step 1 (required): Data scheme definition**_

To generate model-dependent code that is initially missing within both servers it is necessary to define your data scheme first. The data scheme can be defined through a set of data models (it is possible to imagine each data model as a separate table). 
 <br/>
 
 [ > Data Models](setup_data_scheme.md)
<br/><br/>
* _**Step 2 (required): Setup backend GraphQL server**_

The GraphQL server by default provides basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step. To run this server first it is needed to generate scheme-dependent code into the server root folder. Afterwards you can start the server and access it's console from your web browser at `http://<your_host>:3000/graphql/`. 
<br/>

[ > Back-end Server Setup](setup_backend.md)
<br/><br/>
* _**Step 3 (optional): Setup Web Interface application**_

At least for the first time it is recommended to install generic WebBased GUI server that permits to manage your data in a simple user-friendly web interface. Here we purpose a Single Page Application (SPA) approach where all HTML views are constructed on the client side using VueJS framework. This application communicate with GraphQL server using regular POST requests. It is a rule that all operations accessible within this generic GUI are also accessible on the back-end server directly.

Likewise as in the GraphQL server it will be required to generate a data scheme dependent code into the web server folder and run the server afterwards. By default web server page is accessible at `http://<your_host>:8080/`. 
<br/>

[ > Web GUI Server Setup](setup_gui.md)
<br/><br/>
* _**Step 4 (optional): Customizing your project**_ 

 A couple of basic extensions are suggested to be introduced directly into the GraphQL server code. These are: *data validation logic* and *GraphQL query/mutation patches*. To implement custom logic of these extensions some programming is required, however this step is well confined and described.
 <br/>
 
 [ > Advanced code customizing](setup_customize.md)



