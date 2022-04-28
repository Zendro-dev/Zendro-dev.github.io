[ &larr; back](README.md)
<br/>
# Quickstart

This is a quickstart guide on how to create a new Zendro project with default parameters. It uses pre-defined datamodels, database and environment variables.

 <br/>

## Project Requirements:
 * [NodeJS](https://nodejs.org/en/) We strongly recommend to install NodeJS v14.17.6
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

Edit *NEXTAUTH_SECRET* to your expect secret in the following files:
* **SPA in development mode:** ./single-page-app/.env.development
* **SPA in production mode:** ./single-page-app/.env.production
* **GraphiQL in development mode:** ./graphql-server/.env.development
* **GraphiQL in production mode:** ./graphql-server/.env.production

If you want to know more about the enviroment variables, you can check [this]().

### Step 4: Start up your Zendro instance

Execute the next command to start Zendro in production mode. 

```
zendro dockerize -u -p
```

### Step 5: Stop up your Zendro instance

Execute the next command to stop Zendro in production mode and remove all volumes.

```
zendro dockerize -d -p -v
```

