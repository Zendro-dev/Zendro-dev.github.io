---
layout: default
title: Zendro CLI
nav_order: 9
---

# Zendro CLI
{: .no_toc }

A CLI for ZendroStarterPack.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Installation

See [Installation and the Zendro CLI]({% link getting-started/installation-and-cli.md %}) for how to install and update the CLI. To customize the version of each repository, edit the `zendro_dependencies.json` file in your local Zendro CLI repository.

## Commands
### Start a new zendro application
```
Usage: zendro new [options] <your_application_name>

Options:
  -d, --dockerize: Keep Docker files (default: false).
```
Hints:
1. If you don't have a local database, or want to dockerize the Zendro app, keep the docker files — these are examples for dockerizing a Zendro app.
2. To modify environment variables or the database configuration, edit the corresponding docker-compose file and the following files:
* without docker setup: `./graphql-server/config/data_models_storage_config.json`
* with docker setup: `./config/data_models_storage_config.json`
* `./graphql-server/.env`
* SPA in development mode: `./single-page-app/.env.development`
* SPA in production mode: `./single-page-app/.env.production`
* GraphiQL in development mode: `./graphql-server/.env.development`
* GraphiQL in production mode: `./graphql-server/.env.production`

Note: by default, SQLite3 is used for data storage. To use other storage types, reuse the two example files illustrating the configuration of all supported storage types with the docker setup: `./config/data_models_storage_config_example.json` and `./docker-compose-dev-example.yml`.

### Generate code for graphql-server
```
Usage: zendro generate [options]

Options:
  -f, --data_model_definitions: Input directory or a JSON file (default: current directory path + "/data_model_definitions").
  -o, --output_dir: Output directory (default: current directory path + "/graphql_server").
  -m, --migrations: Generate migrations (default: false).
```

### Dockerize Zendro App with example docker files
```
Usage: zendro dockerize [options]

Options:
  -u, --up: Start docker service (default: false).
  -d, --down: Stop docker service (default: false).
  -p, --production: start or stop GQS and SPA with production mode (default: false).
  -v, --volume: remove volumes (default: false).
```

### Start Zendro service
```
Usage: zendro start [options] [service...]

Options:
  -p, --production: start GQS and SPA with production mode (default: false).
```
Hints:
1. Starts all services by default.
2. To start a specific service, use one of these abbreviations:
* gqs: graphql-server
* spa: single-page-app
* giql: graphiql

### Stop Zendro service
```
Usage: zendro stop [service…]

Options:
  -p, --production: stop GQS and SPA with production mode (default: false).
```
Hints:
1. Stops all services by default.
2. Can stop a specific service.

### Generate migration code for graphql-server
```
Usage: zendro migration:generate [options]

Options:
  -f, --data_model_definitions: Input directory or a JSON file (default: current directory path + "/../data_model_definitions").
  -o, --output_dir: Output directory (default: current directory path + "/migrations").
```
Note: all generated migrations are stored in a directory called `migrations`.

### Execute migrations
```
Usage: zendro migration:up
```
Note: executes migrations generated after the last executed migration. The last executed migration is recorded in `zendro_migration_state.json`, and the migration log is in `zendro_migration_log.json`.

### Drop the last executed migration
```
Usage: zendro migration:down
```

### Upload a file
```
Usage: zendro bulk-create [options]

Options:
  -f, --file_path: File path. Supported file format: CSV, XLSX, JSON
  -n, --model_name: Model name.
  -s, --sheet_name: Sheet name for XLSX file. By default process the first sheet.
  -r, --remote_server: Upload to a remote server (default: false).
```

See [How to import and export data]({% link guides/data-import-export.md %}) for the data format requirements and step-by-step examples.

### Download records
```
Usage: zendro bulk-download [options]

Options:
  -f, --file_path: File path.
  -n, --model_name: Model name.
  -r, --remote_server: Download from a remote server (default: false).
```

See [How to import and export data]({% link guides/data-import-export.md %}) for step-by-step examples.

### Set up a quick sandbox
```
Usage: zendro set-up [options] <name>

Options:
  -d, --dockerize: Keep Docker files (default: false).
```

### Create empty or default plots
```
Usage: zendro create-plot [options]

Options:
  -p, --default_plots: Create default plots (default: false).
  -f, --plot_name: Customized plot name.
  -t, --type: The visualization library (options: "plotly", "d3").
  -m, --menu: The location of the plot menu (options: "none", "top", "left").
  -n, --menu_item_name: The item name in the plot menu (default value is the plot name).
```
Hints:
The meaning of the "menu" (-m) options:
* "top": the plot's navigation is located in the top menu bar, with the menu item name
* "left": the sub-menu for the plot is generated in the left menu, with the menu item name
* "none": no navigation

## Setting up a Zendro instance

For a quick sandbox with default settings, see the [Quickstart]({% link quickstart.md %}) guide. For a full walkthrough of setting up a Zendro instance from scratch, see [Getting started]({% link getting-started/index.md %}).

## Example for migrations
If you have new data model definitions, the Zendro CLI is a convenient way to deal with migrations:
1. In the `graphql-server` folder, execute `zendro migration:generate -f <data_model_definitions>`. The migrations are automatically generated in the `/graphql-server/migrations` folder. By default, every migration file has two functions, `up` and `down`: `up` creates a table, `down` deletes the existing table. These functions can be further customized.
2. In the `graphql-server` folder, execute `zendro migration:up` to perform the migrations generated after the last executed one. This updates the last executed migration and the migration log.
3. In the `graphql-server` folder, execute `zendro migration:down` to drop the last executed migration. This updates the latest successful migration and adds the dropped operation to the migration log. If there are remaining records and associations in the table, an error is thrown by default; to forcefully drop the table anyway, set the `DOWN_MIGRATION` environment variable to `true` in `/graphql-server/.env` and re-execute the down-migration.

Note: for all `up` and `down` functions, there is a default argument called `zendro`, giving access to the different APIs in Zendro's layers (resolvers, models, adapters) and enabling GraphQL queries. At the model and adapter level, `zendro` also exposes the storage handler, which interacts with the corresponding database management system. Examples for model `movie` and adapter `dist_movie_instance1`:
```js
await zendro.models.movie.storageHandler;
await zendro.models.movie.countRecords();
await zendro.adapters.dist_movie_instance1.storageHandler;
await zendro.adapters.dist_movie_instance1.countRecords();
```
At the resolver level, `zendro` exposes the corresponding API functions, e.g. `readOneMovie`, `countMovies`, and so on. These functions expect a `context`, provided like in the example below, including an event emitter to collect any occurring errors. Example using `countMovies`:
```js
const {
  BenignErrorArray,
} = require("./graphql-server/utils/errors.js");
let benign_errors_arr = new BenignErrorArray();
let errors_sink = [];
let errors_collector = (err) => {
  errors_sink.push(err);
};
benign_errors_arr.on("push", errors_collector);
const res = await zendro.resolvers.countMovies(
  { search: null },
  {
    request: null, // by default the token is null
    acl: null,
    benignErrors: benign_errors_arr, // collect errors
    recordsLimit: 15,
  }
);
```
It's also possible to execute GraphQL queries or mutations via `execute_graphql(query, variables)`, where `query` is the query string and `variables` represents dynamic values for that query. By default, queries are executed without a token; in a distributed setup with ACL rules, a token is necessary, obtained from Keycloak using the `MIGRATION_USERNAME` and `MIGRATION_PASSWORD` environment variables. Example:
```js
await zendro.execute_graphql("{ countMovies }");
```

## Plots
It is possible to generate default or empty plots via the CLI.

### Create empty plots
To create an empty plot, specify the plot name (`-f`) and the visualization library (`-t`). The location of the plot menu (`-m`) and the item name in the plot menu (`-n`) are optional. For example, an empty plot named `barchart` using the `plotly` library, shown in the top menu bar, can be generated by running the following command in the `single-page-app` folder:
```
zendro create-plot -f barchart -t plotly -m top
```
You can then customize the data processing in `single-page-app/src/pages/barchart.tsx`. In the `fetchData` function, pass the required query as an argument to `zendro.request`, process the response `res` into the desired format, and set it as the `data` variable. Pass `data` as a parameter of the `<PlotlyPlot/>` component. The layout and other plot parameters can be set in `single-page-app/src/zendro/plots/barchart.tsx`.

Similarly, to generate a plot named `circle` using the `d3` library in the left menu, run the following in the `single-page-app` folder:
```
zendro create-plot -f circle -t d3 -m left
```
The processed `data` can be passed as a parameter to the `<D3Plot/>` component in `single-page-app/src/pages/circle.tsx`, and the plot can be customized in the corresponding `single-page-app/src/zendro/plots/circle.tsx` file.

### Generate default plots
Running the following command in the `single-page-app` folder generates three default plots:
```
zendro create-plot -p
```
1. **scatter-plot** — only numerical attributes can be selected. The attribute for the Y-axis must be specified; if no X-axis attribute is selected, the Y-axis values are used to generate the plot. Different modes can be selected: `lines`, `markers`, `lines+markers`.

2. **rain-cloud-plot** — only numerical attributes can be selected, and multiple attributes can be visualized within one plot. Specify "The number of numerical attributes" to render the corresponding selectors, then select the attributes across different data models. You can also choose the plot's `Direction` (horizontal by default, or vertical), specify the `Tickangle`, use the `Autoscale` button for better alignment, and select the `Span mode` (default: soft) — see the [Plotly violin reference](https://plotly.com/javascript/reference/violin/#violin-spanmode) for details on span mode.

3. **boxplot** — set up very similarly to a rain-cloud-plot, but without a `Span mode` option.
