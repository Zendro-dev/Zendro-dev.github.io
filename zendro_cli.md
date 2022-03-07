[ &larr; back](README.md)
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
zendro generate

  Usage: zendro generate [options]

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
    -p, --production: start or stop GQS and SPA with production mode (default: false).
```

### Start Zendro service:
```
zendro start [service...]

  Usage: zendro start [options] [service...]

  Options:
    -p, --production: start GQS and SPA with production mode (default: false).
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
  -p, --production: stop GQS and SPA with production mode (default: false).
```
Hints:
1. stop all service by default
2. stop specified service

### Generate migration code for graphql-server:
```
zendro migration:generate

  Usage: zendro migration:generate [options]

  Options: 
  -f, --data_model_definitions: Input directory or a JSON file (default: current directory path + "/../data_model_definitions").
  -o, --output_dir: Output directory (default: current directory path + "/migrations").
```
Note: all generated migrations would be stored in a directory called `migrations`.
### Execute migrations
```
zendro migration:up

  Usage: zendro migration:up
```
Note: execute migrations which are generated after the last executed migration. Moreover, the last executed migration is recorded in `zendro_migration_state.json` and the log of migrations is in the file `zendro_migration_log.json`.

### Drop the last executed migration
```
zendro migration:down

  Usage: zendro migration:down
```


## A Running Example for setting up a Zendro Instance
1. create a new application (**test**). Keep docker files (**-d**)  by executing  **`zendro new -d test`**. If you would like to modify some environment variables or database configuration, please edit relevant files, which are also specified in the console.
* without docker setup: ./graphql-server/config/data_models_storage_config.json
* with docker setup: ./config/data_models_storage_config.json
* ./graphql-server/.env
* SPA in development mode: ./single-page-app/.env.development
* SPA in production mode: ./single-page-app/.env.production
* ./graphiql-auth/.env

2. `cd test`

3. add JSON files for model definitions in `./data_model_definitions` folder and generate graphql-server (GQS) code and migrations by executing **`zendro generate -m`**.

4. for local database, you can start all service by executing **`zendro start`**. Meanwhile, if you would like to use GQS and SPA with production mode, please add `-p` option.
   
5. stop all running service by executing **`zendro stop`**. Besides, if you would like to stop GQS and SPA with production mode, please add `-p` option.

6. if you don't have local database or you would like to dockerize example zendro App, then execute **`zendro dockerize -u`**. Moreover, if you would like to use GQS and SPA with production mode, please execute **`zendro dockerize -u -p`**. Besides, the default username is `admin@zen.dro` and the corresponding password is `admin`.

7. When you would like to stop docker service, press `CTRL+C` once, then execute **`zendro dockerize -d`** for a full cleanup. In addition, if your GQS and SPA is in production mode, please execute **`zendro dockerize -d -p`**.

## Example for Migrations
If a user has new data model definitions, it is convinient to use Zendro CLI for dealing with migrations. And the following procedure shows how to generate, perform or drop migrations:
1. in `graphql-server` folder, execute `zendro migration:generate -f <data_model_definitions>`. The migrations are automatically generated in the `/graphql-server/migrations` folder. By default, every migration file has two functions, namely `up` and `down`. The `up` function creates a table, the `down` function deletes the existing table. Furthermore it is possible to customize the migration functions.
2. in `graphql-server` folder, it is possible to perform new generated migrations, which are generated after the last executed migration, by executing `zendro migration:up`. After that, the last executed migration and migration log are updated.
3. in `graphql-server` folder, the last executed migration can be dropped by executing `zendro migration:down`. This will update the latest successful migration and add the dropped operation to the migration log. If there are some remaining records and associations in the table, by default an error is thrown. To forcefully drop the table, in spite of remaining records, set the environment variable `DOWN_MIGRATION` to `true` in `/graphql-server/.env` file and re-execute this down-migration.