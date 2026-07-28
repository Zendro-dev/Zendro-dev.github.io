---
layout: default
title: Quick start
nav_order: 2
---

# Quick start
{: .no_toc }

This is a quickstart guide on how to create a new Zendro project with default parameters. It uses pre-defined data models, database and environment variables.
{: .fs-6 .fw-300 }
If you want to know more about Zendro or a detailed explanation on how to set up Zendro from scratch, check the [Getting started]({% link getting-started/index.md %}) guide instead.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Hotstart

For the impatient: paste one of the blocks below into a terminal (Linux, or Windows via WSL) to get a full local Zendro sandbox running with the same demo data used throughout these guides. Each step is explained in detail further down this page and in [Installation and the Zendro CLI]({% link getting-started/installation-and-cli.md %}) — come back here any time as a quick reference.

<details open markdown="1">
<summary>With docker</summary>

```bash
npm install -g git+https://github.com/Zendro-dev/zendro.git
zendro set-up -d zendro-example
cd zendro-example
zendro dockerize -u
```

</details>

<details markdown="1">
<summary>Without docker</summary>

```bash
npm install -g git+https://github.com/Zendro-dev/zendro.git
zendro set-up zendro-example

# Keycloak (requires Java 11+, https://www.java.com/en/)
wget https://github.com/keycloak/keycloak/releases/download/26.7.0/keycloak-26.7.0.zip
unzip keycloak-26.7.0.zip
wget -O keycloak-26.7.0/conf/keycloak.conf https://raw.githubusercontent.com/Zendro-dev/zendro/master/test/env/keycloak.conf
KC_BOOTSTRAP_ADMIN_USERNAME=admin KC_BOOTSTRAP_ADMIN_PASSWORD=admin ./keycloak-26.7.0/bin/kc.sh start-dev &

cd zendro-example
zendro start
```

</details>

On Mac, we recommend the "Without docker" path. If `npm install -g` needs elevated permissions on your system, prefix it with `sudo`.

Once running, jump to [Step 4](#step-4-start-up-your-zendro-instance) below to see the running services, or skip straight to [how to use the graphical interface]({% link guides/graphical-interface.md %}) or [GraphQL basics]({% link guides/graphql-basics.md %}).

## Step 1: Install Zendro

Follow [Installation and the Zendro CLI]({% link getting-started/installation-and-cli.md %}) to install the `zendro` command line tool and its requirements.

## Step 2: Set up a new Zendro project

The easiest way to set up Zendro is using the `zendro` CLI tool with minimal steps and configuration:

```bash
zendro set-up -d <name>
```

where `<name>` is the name of your new project.

By default, three data models with associations will be used for this instance: `city`, `country` and `river`. A default SQLite database will be used; you can find it in the `graphql-server` folder.

This clones the `latest-stable` tag of each of single-page-app, graphql-server and graphiql-auth by default — add `--spa-ref`, `--gqs-ref` and/or `--giql-ref` to pin a different branch or tag instead, e.g. `zendro set-up -d --gqs-ref my-branch <name>`. See the [CLI reference]({% link cli-reference.md %}#set-up-a-quick-sandbox) for details.

## Step 3: Edit environment variables

Go inside the new project you just created, named `<name>`, and edit `NEXTAUTH_SECRET` to your expected secret word in the following files. Remember that dotfiles are usually treated as hidden files, so make sure you can view hidden files:

* **SPA in development mode:** `./single-page-app/.env.development`
* **SPA in production mode:** `./single-page-app/.env.production`
* **GraphiQL in development mode:** `./graphiql-auth/.env.development`
* **GraphiQL in production mode:** `./graphiql-auth/.env.production`

An easy way to set them in Linux is by using the following command, replacing `<secret>` accordingly:

```bash
sed -i 's/^\(NEXTAUTH_SECRET\)=..$/\1="<secret>"/' graphiql-auth/.env.* single-page-app/.env.*
```

If you want to know more about the environment variables, see [Environment variables]({% link getting-started/environment-variables.md %}).

## Step 4: Start up your Zendro instance

### Development mode

```bash
zendro dockerize -u
```

All servers listen for live changes you make to the files. The SPA and graphiql-auth web services will be slow to use since they compile pages on demand when opening them; to avoid that either change `docker-compose-dev.yml` to compile and deploy the web services (see `docker-compose-prod.yml`) or start Zendro in production mode instead.

In development mode there is no reverse proxy mapping the docker services; ports are exposed directly instead.

**Note**: We recommend using a Linux system for development mode.

*If you get a "mandatory OAuth2 variables are not being set" error in SPA or GraphiQL, run `zendro dockerize -d -v` to stop the services and then `zendro dockerize -u` to start them again. This happens because graphql-server needs to write the OAuth2 variables to the .env files before SPA and GraphiQL load, but they sometimes load faster than graphql-server.*

### Production mode

```bash
zendro dockerize -u -p
```

This creates a docker container for each Zendro component:

* [Keycloak]({% link authentication/index.md %}): manages users and roles
* [Single Page App (SPA)](https://github.com/Zendro-dev/single-page-app): graphical interface to send CRUD requests to a Zendro GraphQL endpoint
* [API](https://github.com/Zendro-dev/graphql-server): CRUD API accessible through the GraphQL query language
* [GraphiQL interface](https://github.com/Zendro-dev/graphiql-auth): an implementation of the GraphQL IDE with Zendro login and advanced filter functionalities

Check the running containers with `docker ps`, and their logs with `docker logs -f <container name>`.

> ***Wait until the logs indicate the app is running on the expected port before accessing Zendro's services.***

With the default configuration, the running containers will be:

* **Keycloak** — `http://localhost:8081/auth/admin/zendro/console`, default user `zendro-admin` / password `admin`

  ![Keycloak admin console](/figures/kc2.png)

* **SPA** — `http://localhost:8080`, default user `zendro-admin` / password `admin`

  ![SPA models overview](/figures/spa.png)

* **GraphQL API** — `http://localhost:3000/graphql`

  {% include theme-img.html light="/figures/graphql.png" dark="/figures/graphql-dark.png" alt="Bare GraphQL API endpoint" %}

* **GraphiQL interface with filter functionality** — `http://localhost:7070`, default user `zendro-admin` / password `admin`

  {% include theme-img.html light="/figures/graphiql.png" dark="/figures/graphiql-dark.png" alt="GraphiQL interface" %}

For a full walkthrough with a screenshot of every step, see the [Getting started]({% link getting-started/index.md %}) guide, [how to use the graphical interface]({% link guides/graphical-interface.md %}), or [GraphQL basics]({% link guides/graphql-basics.md %}).

For the default database, you can also install `sqlite3` to inspect the data directly:

```bash
sudo apt install sqlite3
```

Then, from the `graphql-server` folder, run:

```bash
sqlite3 data.db
```

You can list tables and run queries inside sqlite with:

```
sqlite> .tables
sqlite> SELECT * FROM <table>;
sqlite> .exit
```

## Step 5: Stop your Zendro instance

```bash
# Production
zendro dockerize -d -p -v

# Development
zendro dockerize -d -v
```

**Note**: The `-v` flag also removes all volumes. Drop it if you want to persist your data, including user data, between restarts.

---

Need to update or uninstall Zendro afterwards? See [Installation and the Zendro CLI]({% link getting-started/installation-and-cli.md %}#updating-zendro).
