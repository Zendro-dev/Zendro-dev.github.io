[ &larr; back](zendro_cli.md)
<br/>
# Zendro Command Line Interface (CLI)
## Introduction
A CLI for ScienceDbStarterPack.

## Installation
A quick installation would be this command: `npm install -g Zendro-dev/zendro`.
However, if you would like to customize your Zendro CLI, you can set it up as the following:
```
$ git clone https://github.com/Zendro-dev/zendro.git
$ cd zendro
$ npm install
$ npm link
```
For example, you can customize the version of each repository by editing `zendro_dependencies.json` file in your local Zendro CLI repository.

## Commands
### Start a new zendro application:
```
zendro new <your_application_name>

  Usage: zendro new [options] <your_application_name>

  Options:
    -d, --dockerize                Keep Docker files (default: false).
```
Hints: 
1. If you don't have local database or you want to dockerize Zendro App, you can keep docker files. These are examples for dockerizing a Zendro App.
2. If you want to modify environment variables or database configuration, you can edit the following files:
* without docker setup: ./graphql-server/config/data_models_storage_config.json
* with docker setup: ./config/data_models_storage_config.json
* ./graphql-server/.env
* SPA in development mode: ./single-page-app/.env.development
* SPA in production mode: ./single-page-app/.env.production
* ./graphiql-auth/.env

### Generate code for graphql-server:
```
zendro generate-gqs

  Usage: zendro generate-gqs [options]

  Options:
    -f, --data_model_definitions: Input directory or a JSON file (default: current directory path + "/data_model_definitions").
    -o, --output_dir: Output directory (default: current directory path + "/graphql_server").
    -m, --migrations: Generate migrations (default: false).
```

### Dockerize Zendro App with example docker files:
```
zendro dockerize

  Usage: zendro dockerize [options]

  Options:
    -u, --up: Start docker service (default: false).
    -d, --down: Stop docker service (default: false).
    -p, --production: start or stop SPA with production mode (default: false).
```

### Start Zendro service:
```
zendro start [service...]

  Usage: zendro start [options] [service...]

  Options:
    -p, --production: start SPA with production mode (default: false).
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

  Options:
  -p, --production: stop SPA with production mode (default: false).
```
Hints:
1. stop all service by default
2. stop specified service

## A Running Example
1. create a new application (**test**). Keep docker files (**-d**)  by executing  **`zendro new -d test`**. If you would like to modify some environment variables or database configuration, please edit relevant files, which are also specified in the console.
* without docker setup: ./graphql-server/config/data_models_storage_config.json
* with docker setup: ./config/data_models_storage_config.json
* ./graphql-server/.env
* SPA in development mode: ./single-page-app/.env.development
* SPA in production mode: ./single-page-app/.env.production
* ./graphiql-auth/.env

2. `cd test`

3. add JSON files for model definitions in `./data_model_definitions` folder and generate graphql-server code and migrations by executing **`zendro generate-gqs -m`**.

4. for local database, you can start all service by executing **`zendro start`**. Meanwhile, if you would like to use SPA with production mode, please add `-p` option.
   
5. stop all running service by executing **`zendro stop`**. Besides, if you would like to stop SPA with production mode, please add `-p` option.

6. if you don't have local database or you would like to dockerize example zendro App, then execute **`zendro dockerize -u`**. Moreover, if you would like to use SPA with production mode, please execute **`zendro dockerize -u -p`**. Besides, the default username is `admin@zen.dro` and the corresponding password is `admin`.

7. When you would like to stop docker service, press `CTRL+C` once, then execute **`zendro dockerize -d`** for a full cleanup. In addition, if your SPA is in production mode, please execute **`zendro dockerize -d -p`**.