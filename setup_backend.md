[ &larr; back](setup_root.md)
<br/>
# Back-end Server Setup

<br/><br/>
_**Server "skeleton"**_

For installing the backend skeleton we will need a copy of the repository, found in [this link](https://github.com/ScienceDb/graphql-server), installed locally. For this purpose we should run the following lines:

```
$ git clone https://github.com/ScienceDb/graphql-server.git backend-skeleton
$ cd backend-skeleton
$ npm install  
```
From now on, in this document, we will assume that your backend-skeleton is installed in `/your-path/backend-skeleton`.

<br/><br/>
_**Install backend code generator**_

For installing the backend generator we will need a copy of the repository, found in [this link](https://github.com/ScienceDb/graphql-server-model-codegen), installed.
For this purpose we should run the following lines:
```
$ git clone https://github.com/ScienceDb/graphql-server-model-codegen.git backend-generator
$ cd backend-generator
$ npm install -g
```
If you only want to install it locally, then you should run
`$ npm install` instead and then adapt each command accordingly.  

<br/><br/>
_**Generate the code**_

After installed the backend-generator and as long as we have our data models defined, then we can generate the code for the graphql server. For this purpose we should run the following lines:

```
$ code-generator generate <input-json-files> <output-directory>
```
```
INPUT:
<input-json-files> - directory where json models are stored
<output-directory> - directory where the generated code will be written
```

Example:
```
$ code-generator generate /your-path/json-files /your-path/backend-skeleton
```

<br/><br/>
_**Run GraphQL server**_

```
$ cd /your-path/backend-skeleton
$ node_modules/.bin/sequelize db:migrate
$ node server.js
```
`$ node_modules/.bin/sequelize db:migrate` command will create the tables specified in the migrations folder. With credential as in config/config.json file.

*Note:* For the command `$ node_modules/.bin/sequelize db:migrate` a data base should be already configured locally as in `config/config.json` file, which is part of the project. If you followed all the instructions, this file should be in  `/your-path/backend-skeleton/config/config.json`

<br/><br/>
_**Setup environment variables**_

You can also specify some environment variables:

* `PORT` - The port where the app is listening, default value is `3000`
* `ALLOW_ORIGIN` - In development mode we need to specify the header `Access-Control-Allow-Origin` so the SPA application can communicate with the server, default value is `http://localhost:8080`.
* `LIMIT_RECORDS` - Maximum number of records that each request can return, default value is 10000.

Example:
```
$ PORT = 7000 && node server.js
```
Now your server will be listening on PORT 7000.
