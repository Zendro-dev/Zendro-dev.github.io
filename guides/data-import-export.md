---
layout: default
title: Import and export data
parent: User guides
nav_order: 4
permalink: /guides/data-import-export
---

# How to import and export data
{: .no_toc }
Data format requirements, and how to upload or download data through the SPA or the Zendro CLI.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Once your data models are generated, you can upload data to a Zendro instance, or download it, either through the graphical interface (SPA) or the [Zendro CLI]({% link cli-reference.md %}). Both use the same file format.

## Data format requirements

Data to populate each model in your schema must be in a separate CSV, XLSX or JSON file, following these requirements:

1. Column names in the first row must correspond to model attributes. For associations, the column name format is `add<associationName>`, e.g. `addCountries` for association name `countries`.
2. Empty values should be represented as `"NULL"`.
3. All fields should be quoted with `"`. However, if the field delimiter and array delimiter do not occur within any String fields — i.e. the fields could be split unambiguously — quoting is not necessary. For example, if the field delimiter is a comma and one String field is `Zendro, excellent!`, that field must be quoted, since without quotes it would be split into two fields.
4. Default configuration: `LIMIT_RECORDS=10000`, `RECORD_DELIMITER="\n"`, `FIELD_DELIMITER=","`, `ARRAY_DELIMITER=";"`. These can be changed via [environment variables]({% link getting-started/environment-variables.md %}).
5. Date and time formats must follow the [RFC 3339](https://tools.ietf.org/html/rfc3339) standard.

If you're using a spreadsheet processor (e.g. Excel) to prepare a CSV file, string (text) records should still be quoted with `"` — for example `"Ingredient A, Ingredient B"` rather than `Ingredient A, Ingredient B` — unless there are no commas within any single record. `"NULL"` should also be quoted.

To get the field names right and check each attribute's type (integer, string, etc.), download the model template first — see [Download the model template](#download-the-model-template) below.

## Uploading data

### Through the graphical interface

Go to the model in the left-side panel of the [SPA]({% link guides/graphical-interface.md %}) and use the import button (the "bold top arrow" icon in the table's top menu). It asks you to select a file from your computer and automatically fills the table. To modify the default configuration for delimiters and batch size, edit `single-page-app/.env.development` or `single-page-app/.env.production` — see [Environment variables]({% link getting-started/environment-variables.md %}).

If there are no errors, your data is uploaded — click the Reload button if you don't see it right away. If there are errors, you'll be told what the problem is; check your data and try again.

**Note:** the maximum allowed upload size and other related settings are configurable when setting up Zendro — see [Environment variables]({% link getting-started/environment-variables.md %}).

### Through the Zendro CLI

1. If the Zendro instance is on your local machine, go into the `graphql-server` folder and execute:

   ```bash
   zendro bulk-create -f <filename> -n <modelname> -s <sheetname>
   ```

   e.g. `zendro bulk-create -f ./country.csv -n country`. CSV, XLSX and JSON are all supported; `sheetname` is only used for XLSX files (if empty, the first sheet is imported by default). The default configuration for delimiters and record limit can be found in `graphql-server/.env`.

2. To upload a file to a remote Zendro server, configure `zendro/.env.migration` accordingly, then execute:

   ```bash
   zendro bulk-create -f <filename> -n <modelname> -s <sheetname> -r
   ```

   e.g. `zendro bulk-create -f ./country.csv -n country -r`.

Note: if record validation fails, a log file is stored in the folder of the uploaded file, named `errors_<uuid>.log`.

See the [CLI reference]({% link cli-reference.md %}#upload-a-file) for the full list of options.

## Downloading data

You can download all data for a model into CSV format, either through the SPA or the CLI. Every attribute is quoted to avoid ambiguity, and remains compatible with Zendro's bulk-creation functionality. Column names for foreign keys follow the same `add<associationName>` format as on upload — for example, an association named `countries` with a foreign key `country_ids` produces the column `addCountries`.

### Through the graphical interface

Click the `Download data` button (the "bold down arrow" icon) to download the current table as a CSV file. You'll be prompted to select a directory and file name. To modify the default configuration for delimiters and record limit, edit `single-page-app/.env.development` or `single-page-app/.env.production`.

Be aware that if you modify the `NEXT_PUBLIC_ZENDRO_MAX_RECORD_LIMIT` environment variable, you should also update `LIMIT_RECORDS` in the backend graphql-server.

This only downloads the data of one table at a time. Complex queries that combine columns from different models into a single table can be done through Zendro's [GraphQL API]({% link guides/graphql-basics.md %}) instead.

#### Download the model template

To check the field names and data types expected for a model, download its template by clicking the "light down arrow" icon in the table's top menu. This downloads a CSV file named after the table (e.g. `river.csv`), with the expected column names in the first row and the expected data type of each column in the second row (e.g. `Int`, `String`). Open it in your favourite spreadsheet processor, replace the second row with your data, and save it as `.xlsx` or `.csv` — it's now ready to upload.

### Through the Zendro CLI

1. If the Zendro instance is installed locally, run, from the `graphql-server` folder:

   ```
   zendro bulk-download -f <filename> -n <modelname>
   ```

   To configure delimiters (`ARRAY_DELIMITER`, `FIELD_DELIMITER`, `RECORD_DELIMITER`) and record limit (`LIMIT_RECORDS`), set the corresponding environment variables in `graphql-server/.env`.

2. If the Zendro instance is remote, configure `zendro/.env.migration` to point to it, then execute:

   ```
   zendro bulk-create -f <filename> -n <modelname> -r
   ```

   to download the records to CSV.

See the [CLI reference]({% link cli-reference.md %}#download-records) for the full list of options.
