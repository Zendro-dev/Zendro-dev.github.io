# A new ScienceDB project from scratch

ScienceDB consists of four source-code projects: __graphql-server-model-codegen__, __graphql-server__, __single-page-app-codegen__ and a __single-page-app__. The first pair is responsible for the back-end GraphQL service that can be accessed on the default port 3000. The second pair of projects acts as a client of the GraphQL server and creates a simple generic web-based GUI for this server on the 8080 port. As it can be derived from the names of these projects, the `codegen` suffix means that to pull up the corresponding server it is required to generate some code first. Code generation step is required to separate as much as possible data scheme definition from the routinary ORM programming.

 <br/>
 
 __Project Requirements:__
 * nodejs ([Official site](https://nodejs.org/en/))
 * npm ([Official site](https://www.npmjs.com/get-npm))
 * vue cli ([Official site](https://cli.vuejs.org/))
 
 <br/><br/>

* * *
<br/><br/>
* _**Step 1 (required): Data scheme definition**_

Code generation step is required before you can start any of the ScienceDB servers. To generate a model-dependent code that is missing within the servers it is necessary to define your data scheme first. The data scheme can be defined through a set of data models (tables) as it is described in the ['Data Models'](dataModels.md) section.
 
 Given a set of data models, ScienceDB will allow you to use code generators. 
 Also, given the set of data models you will be able to generate a GUI which will allow you to connect with the graphql server in a more intuitive way.

<br/><br/>
* _**Step 2 (required): Setup backend GraphQL server**_

The GraphQL server by default provides basic create, read, update and delete (CRUD) operations for each data model specified in the scheme from the previous step. To start up with this server first it is needed to generate a scheme-dependent code into the server root folder. Afterwards you can start the server and access to it's console on the `http://<your_host>:3000/graphql/` address. For more details on these operations please refer to the ['Backend setup'](backendSetUp.md) section.
 
 
<br/><br/>
* _**Step 3 (optional): Setup Web Interface application**_ ( [more details](guiSetUp.md) )

At least for the first time is is recommended to install the generic WebBased GUI server to manage your data in a simple user-friendly web interface. Here we purpose the Single Page Application approach where all HTML views are constructed on the client side using VueJS framework. An application would communicate with the GraphQL server using corresponding POST requests. All operations that are accessible from the GUI are also supported on the GraphQL server console directly.

As in the case of the GraphQL server it will be required to generate a data scheme dependent code into the web server folder, and than it will be possible run the server. By default the server page will be accessible on the `http://<your_host>:8080/`. See the ['Web GUI Setup'](guiSetUp.md) section for more details.

<br/><br/>
* _**Step 4 (optional): Customizing the project**_ ( [more details](guiSetUp.md) )

 Some basic extensions are suggested to be introduced directly into the GraphQL server. These are the access permissions, data validation logic and custom GraphQL queries and mutations. The ['Advance code customizing'](projectCustomizing.md) section will guide you through the suggested steps, however as long as you entering now into the coding process - everything is possible.


Unfortunately ScienceDB can support just one scheme per GraphQL server. It means that if you would need to create a multi-scheme project or to implement any strong business logic it is definitely that a new Web GUI server will be needed that could unite a more than one GraphQL server into the single user interface.
