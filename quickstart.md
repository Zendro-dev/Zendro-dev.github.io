[ &larr; back](README.md)
<br/>
# Quickstart

This is a quickstart guide on how to create a new Zendro project with default parameters. It uses pre-defined datamodels, database and environment variables.

If you want to know more about Zendro or a detailed explanation on how to set up Zendro from scratch, check [this](setup_root.md).

 <br/>

## Project Requirements:
 * [NodeJS](https://nodejs.org/en/)
 * [docker](https://docs.docker.com/get-docker/)
 * [docker-compose](https://docs.docker.com/compose/install/#install-compose)
 <br/><br/>

* * *
### Step 1: Installation

Execute this commands to install Zendro:

```
$ git clone https://github.com/Zendro-dev/zendro.git
$ cd zendro
$ npm install
$ npm link
```

### Step 2: Setup a new Zendro project

The easiest way to set up Zendro is using the [Zendro CLI tool](https://github.com/Zendro-dev/zendro) with minimal steps and configuration. Execute:

```
$ zendro set-up -d <name>
```

By default, three data models with associations will be used for this instance:
* city
* country
* river

Also a default SQLite database will be used. You can find the database on *graphql-server* folder.

### Step 3: Edit environment variables

Edit *NEXTAUTH_SECRET* to your expected secret word in the following files:
* **SPA in development mode:** ./single-page-app/.env.development
* **SPA in production mode:** ./single-page-app/.env.production
* **GraphiQL in development mode:** ./graphql-server/.env.development
* **GraphiQL in production mode:** ./graphql-server/.env.production

If you want to know more about the enviroment variables, you can check [this](env_vars.md).

### Step 4: Start up your Zendro instance

Execute the next command to start Zendro in production mode. 

```
$ zendro dockerize -u -p
```

This command will create docker containers for each Zendro component:
* [Keycloak](https://github.com/Zendro-dev/Zendro-dev.github.io/blob/documentation-vb/oauth.md): manage users and roles
* [Single Page App (SPA)](https://github.com/Zendro-dev/single-page-app): graphical interface to send CRUD requests to a Zendro GraphQL endpoint
* [API](https://github.com/Zendro-dev/graphql-server): CRUD API that can be accessed through a GraphQL query language
* [API with authenthication](https://github.com/Zendro-dev/graphiql-auth): An implementation of the GraphQL IDE with Zendro login and advanced filter functionalities.
* [traefik reverse-proxy](): A reverse-proxy using traefik that maps the above docker services.

You can check docker containers by:
```
$ docker ps
```

You can check docker logs by:
```
$ docker logs -f <container name>
```

> ***Please wait until logs indicate the app is running on XXXX port to access Zendro services.***

In default config, the running containers will be on ports:

* Keycloak: http://localhost/auth
   * The default keycloak username is *admin* and the password is *admin*.

  ![Keycloak example](figures/kc1.png)
  ![Keycloak example](figures/kc2.png)

* SPA: http://localhost/spa
    * The default zendro username is *zendro-admin* and the password is *admin*.

  ![spa example](figures/login.png)
  ![spa example](figures/spa.png)

* GraphQL API: http://localhost/api/

  ![api example](figures/graphql.png )

* GraphQL API with authenthication: http://localhost/graphiql
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

Execute the next command to stop Zendro and remove all volumes.

```
$ zendro dockerize -d -p -v
```

**Note**: If you want to persist your data, that includes user data as well as other data, remove the `-v` flag from the above command.

#### Development mode

To start Zendro in development mode run

```
$ zendro dockerize -u
```

This will start Zendro in development mode. All servers are listening to live changes you make in the files. Especially the SPA and graphiql-auth web-services will be slow to use since they compile pages on demand when openening them. To avoid that either change the `docker-compose-dev.yml` to compile and deploy the webservices (see `docker-compose-prod.yml`) or start Zendro in production mode.

In development mode there is no reverse proxy to map the docker-services. Instead this is done by exposing the ports as follows:

* API -`http://localhost:3000`
* GraphiQL - `http://localhost:7000`
* Single Page App (SPA) - `http://localhost:8080`
* Keycloak - `http://localhost:8081`

Stop the instance using

```
$ zendro dockerize -d -v
```

