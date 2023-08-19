---
layout: default
title: Quick start
nav_order: 3
---

# Quick start
{: .no_toc }

This is a quickstart guide on how to create a new Zendro project with default parameters. It uses pre-defined datamodels, database and environment variables. 
{: .fs-6 .fw-300 }
If you want to know more about Zendro or a detailed explanation on how to set up Zendro from scratch, check [this]({% link setup_root.md %}).
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Project Requirements:
 * [NodeJS](https://nodejs.org/en/) version 18+ is required.
 * [docker](https://docs.docker.com/get-docker/)
 * [docker-compose](https://docs.docker.com/compose/install/#install-compose)

 We strongly recommend to follow [this guide](https://docs.docker.com/engine/install/linux-postinstall/) to use docker without sudo.
 <br/><br/>

* * *
## Recommendations:
  * We strongly recommend you to use Zendro in Linux with or without docker.
  * If you prefer to use Zendro in Windows, we recommend you to use it with Windows Subsystem for Linux (WSL).
  * If you prefer to use Zendro in Mac, we recommend you to use it without docker.

 <br/>

* * *
### Step 1: Installation

Execute this commands to install Zendro:

```
$ git clone https://github.com/Zendro-dev/zendro.git
$ cd zendro
$ npm install
$ sudo npm link
```

### Step 2: Setup a new Zendro project

The easiest way to set up Zendro is using the [Zendro CLI tool](https://github.com/Zendro-dev/zendro) with minimal steps and configuration. 

Go out from the previusly created `zendro` directory 

```
$ cd ..
```

and execute:

```
$ zendro set-up -d <name>
```

where `<name>` is the name of your new project.

By default, three data models with associations will be used for this instance:
* city
* country
* river

Also a default SQLite database will be used. You can find the database on *graphql-server* folder.

### Step 3: Edit environment variables

Go inside the new project you just created named `<name>` and edit *NEXTAUTH_SECRET* to your expected secret word in the following files. Remember that dotfiles are usually treated as hidden files, so make sure you can view hidden files:

* **SPA in development mode:** ./single-page-app/.env.development
* **SPA in production mode:** ./single-page-app/.env.production
* **GraphiQL in development mode:** ./graphiql-auth/.env.development
* **GraphiQL in production mode:** ./graphiql-auth/.env.production

If you want to know more about the enviroment variables, you can check [this]({% link env_vars.md %}).

### Step 4: Start up your Zendro instance

#### Development mode

To start Zendro in development mode run

```
$ zendro dockerize -u
```

This will start Zendro in development mode. All servers are listening to live changes you make in the files. Especially the SPA and graphiql-auth web-services will be slow to use since they compile pages on demand when opening them. To avoid that either change the `docker-compose-dev.yml` to compile and deploy the webservices (see `docker-compose-prod.yml`) or start Zendro in production mode.

In development mode there is no reverse proxy to map the docker-services. Instead this is done by exposing the ports.

**Note**: We recommend to use Linux system for development mode.

*If you are having problems starting zendro in development mode due to "mandatory OAuth2 variables are not being set" error in SPA or GraphiQL, please run `zendro dockerize -d -v` to stop the services and then `zendro dockerize -u` to start services again. This happens because graphql-server should write the OAuth2 variables in .env files before SPA and GraphiQL load, but SPA and GraphiQL are loading faster than graphql-server.*

#### Production mode
Execute this command to start Zendro in production mode.

```
$ zendro dockerize -u -p
```

This command will create docker containers for each Zendro component:
* [Keycloak]({% link oauth.md %}): manage users and roles
* [Single Page App (SPA)](https://github.com/Zendro-dev/single-page-app): graphical interface to send CRUD requests to a Zendro GraphQL endpoint
* [API](https://github.com/Zendro-dev/graphql-server): CRUD API that can be accessed through a GraphQL query language
* [GraphiQL interface](https://github.com/Zendro-dev/graphiql-auth): An implementation of the GraphQL IDE with Zendro login and advanced filter functionalities.

You can check docker containers by:
```
$ docker ps
```

You can check docker logs by:
```
$ docker logs -f <container name>
```

> ***Please wait until logs indicate the app is running on XXXX port to access Zendro services.***

In default config, the running containers will be:

* Keycloak: 
    * http://localhost:8081/auth
    
      * The default keycloak username is *admin* and the password is *admin*.

  ![Keycloak example](figures/kc1.png)
  ![Keycloak example](figures/kc2.png)

* SPA: 
    * http://localhost:8080

      * The default zendro username is *zendro-admin* and the password is *admin*.

  ![spa example](figures/login.png)
  ![spa example](figures/spa.png)

* GraphQL API: 
    * http://localhost:3000/graphql

  ![api example](figures/graphql.png )

* GraphiQL interface with filter functionality: 
    * http://localhost:7070

      * The default zendro username is *zendro-admin* and the password is *admin*.

  ![api example](figures/login.png)
  ![api example](figures/graphiql.png)


Also, for the default database you can install *sqlite3* with:

```
$ sudo apt install sqlite3
```

Then, go to *graphql-server* folder and run:

```
$ sqlite3 data.db
```

You can see tables and do querys inside sqlite by:
```
sqlite> .tables
sqlite> SELECT * FROM <table>;
sqlite> .exit
```


### Step 5: Stop your Zendro instance

Execute this command to stop Zendro and remove all volumes.

```
# Production
$ zendro dockerize -d -p -v

# Development
$ zendro dockerize -d -v
```

**Note**: If you want to persist your data, that includes user data as well as other data, remove the `-v` flag from the above command.

* * *
## Updating Zendro

To update Zendro enter your zendro folder and execute:

```
$ git pull
$ npm ci
```

* * *
## Uninstallation

### Remove Project

Execute the following to remove a project:

```
$ rm -r "path/to/<name>"
$ docker rmi -f $(docker images -a -q "<name>*")
```

### Uninstall Zendro

To uninstall Zendro, execute the following:

```
$ sudo npm unlink -g zendro
$ sudo rm -r "path/to/zendro"
```
