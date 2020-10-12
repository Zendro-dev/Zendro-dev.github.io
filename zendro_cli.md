[ &larr; back](zendro_cli.md)
<br/>
# Zendro Command Line Interface (CLI)
## Introduction
A CLI for ScienceDbStarterPack.

## Installation
Install Zendro CLI
```
$ git clone https://github.com/Zendro-dev/zendro.git
$ cd zendro
$ npm install
$ npm link
```

## Commands
### Start a new zendro application:
```
zendro new <your_application_name>

  Usage: zendro new [options] <your_application_name>

  Options:
    -d, --dockerize                Keep Docker files (default: false).
    -u, --update_code_generators   Update code generators (default: false).
```
Hints: 
1. If you don't have local database or you want to dockerize Zendro App, you can keep docker files. These are examples for building a Zendro App.
2. If you have local database, please edit the storage config file
* `./graphql-server/config/data_models_storage_config.json `
3. If you want to modify environment variables, you can edit the following files:
* `./graphql-server/config/globals.js`
* `./single-page-app/src/config/globals.js`
* `./graphiql-auth/src/config/globals.js`

### Generate code for graphql-server:
```
zendro generate-gqs

  Usage: zendro generate-gqs [options]

  Options:
    -d, --data_model_definitions: Input directory or a JSON file.
    -o, --output_dir: Output directory.
    -m, --migrations: Generate migrations (default: false).
```

### Generate code for single-page-app:
```
zendro generate-spa

  Usage: zendro generate-spa [options]

  Options:
    -d, --data_model_definitions: Input directory or a JSON file.
    -o, --output_dir: Output directory.
    -D, --createBaseDirs: Create base directories (default: false).
```

### Dockerize Zendro App with example docker files:
```
zendro dockerize

  Usage: zendro dockerize [options]

  Options:
    -u, --up: Start docker service (default: false).
    -d, --down: Stop docker service (default: false).
```

### Start Zendro service:
```
zendro start [service...]

  Usage: zendro start [options] [service...]

  Options:
    -i, --install_package: Install packages (default:false).
```
Hints:
1. start all service by default
2. start specified service with the following abbreviations:
* gqs: graphql-server
* spa: single-page-app
* giql: graphiql
  
### Stop Zendro service:
```
zendro stop [service…]

  Usage: zendro stop [service…]
```
Hints:
1. stop all service by default
1. stop specified service with abbreviations

## Running Examples
1. create a new application (**test**). Keep docker files (**-d**) and update the latest stable code generators (**-u**) by executing  `zendro new -d -u test`. If you want to modify some environment variables, please edit relevant files, which are also specified in the console.
* ./graphql-server/config/globals.js
* ./single-page-app/src/config/globals.js
* ./graphiql-auth/src/config/globals.js
  
2. `$ cd test`

3. generate graphql-server code and migrations by executing `zendro generate-gqs -d ../schema -m`

4. generate single-page-app code by executing `zendro generate-spa -d ../schema`

5. if you don't have local database or you want to dockerize example zendro App, then execute `zendro dockerize -u`

6. When you want to stop docker service, press `CTRL+C` once, then execute `zendro dockerize -d`
   
7. for local database, you can edit its config file: *./graphql-server/config/data_models_storage_config.json*. Then install all necessary packages for all service and start all service by executing `zendro start -i`
   
8. stop all running service by executing `zendro stop`