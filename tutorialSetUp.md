# A new ScienceDB project from scratch

ScienceDB consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app-codegen__ and a __single-page-app__. The first pair is responsible for the back-end GraphQL service that can be accessed on the default port 3000. The second pair of projects act as a client of the GraphQL server and create a simple generic web-based GUI for this server on the 8080 port. As it can be derived from the names of these projects, the *codegen* suffix means that to pull up the corresponding server it is required to generate some code first. Code generation step is required to separate data scheme definition from the routinary ORM programming.

 <br/>
 
 __Project Requirements:__
 * [nodejs](https://nodejs.org/en/)
 * [npm](https://www.npmjs.com/get-npm)
 * [vue cli](https://cli.vuejs.org/)
 
 <br/><br/>

* * *
<br/><br/>
* _**Step 1 (required): Data scheme definition**_

Code generation step is required before you can start any of the ScienceDB servers. To generate a model-dependent code that is initially missing within the servers it is necessary to define your data scheme first. The data scheme can be defined through a set of data models (tables). Code generation step is required before you can start any of the ScienceDB servers. To generate a model-dependent code that is initially missing within the servers it is necessary to define your data scheme first. The data scheme can be defined through a set of data models (tables). 
 <br/>
 
 [ > Data Models](dataModels.md)
<br/><br/>
* _**Step 2 (required): Setup backend GraphQL server**_

The GraphQL server by default provides basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step. To start up with this server first it is needed to generate scheme-dependent code into the server root folder. Afterwards you can start the server and access it's console from your web browser at `http://<your_host>:3000/graphql/`. 
<br/>

[ > Backend setup](backendSetUp.md)
<br/><br/>
* _**Step 3 (optional): Setup Web Interface application**_

At least for the first time is is recommended to install generic WebBased GUI server that permits to manage your data in a simple user-friendly web interface. Here we purpose the Single Page Application (SPA) approach where all HTML views are constructed on the client side using VueJS framework. An application would communicate with GraphQL server using regular POST requests. As a rule, all operations that are accessible from GUI are also supported on the GraphQL server console directly.

Likewise as in the GraphQL server it will be required to generate a data scheme dependent code into the web server folder, and run the server afterwards. By default web server page is accessible at `http://<your_host>:8080/`. 
<br/>

[ > Web GUI Setup](guiSetUp.md)
<br/><br/>
* _**Step 4 (optional): Customizing your project**_ 

 Some basic extensions are suggested to be introduced directly into the GraphQL server. These are the access permissions, data validation logic and custom GraphQL queries/mutations. Such extensions suggest some programming, however this step is well confined and described. Unfortunately ScienceDB can support just one scheme per GraphQL server. It means that if you would need to create a multi-scheme project or implement any strong business logic it is definitely that a new Web GUI server will be needed.
 <br/>
 
 [ > Advance code customizing](projectCustomizing.md)



