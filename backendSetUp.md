# Set up backend

### Install backend skeleton
For installing the backend skeleton we will need a copy of the repository, found in [this link](https://github.com/ScienceDb/graphql-server), installed locally. For this purpose we should run the following lines:

```
$ git clone https://github.com/ScienceDb/graphql-server.git backend-skeleton
$ cd backend-skeleton
$ npm install  
```
From now on, in this document, we will assume that your backend-skeleton is installed in `/your-path/backend-skeleton`.

### Install backend generator
For installing the backend generator we will need a copy of the repository, found in [this link](https://github.com/ScienceDb/graphql-server-model-codegen), installed.
For this purpose we should run the following lines:
```
$ git clone https://github.com/ScienceDb/graphql-server-model-codegen.git backend-generator
$ cd backend-generator
$ npm install -g
```
If you only want to install it locally, then you should run
`$ npm install` instead.  

### Generate backend
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
$ code-generator /your-path/json-files /your-path/backend-skeleton
```

### Run graphql server
```
$ cd /your-path/backend-skeleton
$ node server.js
```
