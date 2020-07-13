# How to get started: a Zendro guide for non-developers

This guide is aimed at data modelers, data managers and other data users to facilitate collaboration with developers in designing Zendro-generated database systems. We assume that as a data modeler or manager, you might be responsible for structuring your database, and work with a developer or system administrator to set up the project. In this guide we describe and illustrate the requirements for data models, which are the main input for Zendro, and then follow up to describe data uploading options. If you want to dive deeper into the installation process from scratch, see this [tutorial](https://sciencedb.github.io/setup_root.html) on how to set up a new project.

## Data models

Zendro takes as input a set of data models described in JSON files, from which it automatically generates both the backend and the GUI of a custom data warehouse. To get started, you need a conceptual model or schema of your database, including entities, attributes and their relationships. Each *data model*, that is, each entity with its corresponding attributes and associations to other entities, must be defined as a JSON object in a separate file according to the specifications below. Zendro supports the standard associations One-to-One, One-to-Many, and Many-to-Many, between data models stored either locally or remotely, which we'll also describe in detail. But first, let's introduce an example that will help illustrate model requierements throughout the guide. 

### An example

Let's assume we want to create a small database for a herbarium of medicinal plants. We need data models for specimens, taxonomic information, collection information, and uses (Figure 1). In our case, the taxonomic information is specific to each plant, so there is a One-to-One association between specimen and taxon. Each plant can have many uses, just like there are many plants that can serve the same function, so the association between specimen and uses is Many-to-Many. Finally, a specimen belongs to one collection only, but a collection may store multiple specimens, so there is a One-to-Many association between these models (Figure 1). 

![Figure 1](figures/figure1.jpg)

In this example, information about specimens, taxonomic data and uses is stored in a local server. But we will assume that the information about collections is in a remote database, perhaps one that holds other types of plants, and we would connect to that database to access only the attributes we are interested in. 

Next, we need to list the attributes of each model and their data types (Figure 2). Allowed data types are: String, Int, Float, Boolean, [Date, Time and DateTime](https://github.com/excitement-engineer/graphql-iso-date/blob/HEAD/rfc3339.txt). Each model also requires an attribute that serves as the *primary key* or unique identifier of each record. 

![Figure 2](figures/figure2.jpg)

Foreign keys are also needed to establish the associations to other data models; their location depends on the association type. Note that in Many-to-Many associations, it is necessary to define an additional model for the cross table of foreign-key pairs that define the association. For more details, please read the documentation on associations in [Sequelize](https://sequelize.org/master/manual/assocs.html), on which Zendro is based.

### JSON specs

Each data model must include the following fields. We recommend using *snake_case* naming style for model, associations and attributes, so that you get readable names in the the generated API. Also, remember to use the standard conventions for variable names (no spaces or special characters, etc.).

Field name | Type | Description | Example
----|----|----|------|
model | String | Name of the data model. This must be unique within each project. | `"model": "specimen"`.
storageType | String | Type of storage of the model. One of: <ul> <li>  __sql__ for local relational databases supported by [sequelize](#http://docs.sequelizejs.com/) such as PostgreSql/MySql etc. </li>  <li>__generic__ for remote databases.</li>  <li>__zendro\_server__ for models stored in another instance created with Zendro. </li></ul> | `"storageType": "sql"`
url | String | Required only when `"storageType": "zendro_server"`. URL where the Zendro server storing the model is running.
attributes | Object | A list where keys are model attributes and values are the corresponding data types. Optionally, values can also be objects that store both the data type and a description of the attribute in the form: `{"type": "String", "description": "String"}`. | `"attributes": {"specimen_id": "Int", "date": "Date"}` **Or** `"attributes": {"specimen_id": {"type": "Int", "description": "Unique identifier of each specimen"}, "date": "Date"}`
associations | Object | A list where keys are association names and values are objects describing the association. See the specifications below. | `"associations" : {"taxonomic_information": {"type": "to_one", // See all attributes below}}`
internalId | String | Primary key of the data model. If this field is not specified, a default attribute *id* wil be added. | `"internalId": "specimen_id"`

See a complete example below.

### Associations

Every association of the source model to other models must be described in the `associations` field with the following arguments:

Field name | Type | Description
---- | ---- | ----
*type* | String | Type of association, either `to_one` or `to_many`. The standard associations are defined through combining these types in the source and target models. For example, a One-to-One association results from assigning `"type": "to_one"` in both the source and target models. 
*target* | String | Name of the target model to which the current one will be associated with.
*targetKey* | String | Attribute that serves as a unique identifier for the association. It can be stored in either of the two models involved in the association.
*keyIn* | String | Name of the model where the *targetKey* is stored.
*targetStorageType* | String | Type of storage of the target model.
*label* | String | Attribute in the target model to be used as a display name in the GUI. This will be useful as a preview of the association.
*sublabel* | String | Optional name of an attribute in the target model to be used as a sub-label in the GUI. Also used for a more explicit preview of the association.
*sourceKey* | String | Required only in Many-to-Many associations. Primary key of the source model.
*keysIn* | String | Required only in Many-to-Many associations. Name of the model that serves as a cross table.

## From conceptual model to JSON

Now we can translate the conceptual diagram into JSON, following the previous specifications. We present two complete examples below.

```
// Taxon Model
{
    "model": "taxon",
    
    "storageType": "sql",

    "attributes": {
        "taxon_id": "String",
        "scientific_name": "String",
        "specimen_id": "Int"
    },

    "associations": {
        "specimen_information": { // give your association an informative name
            "type" : "to_one",
            "target": "specimen",
            "targetKey": "specimen_id",
            "keyIn": "taxon",
            "targetStorageType": "sql",
            "label": "common_name",
        },
    },

    "internalId": "taxon_id"
}
```

```
// Collection Model
{
    "model": "collection",
    
    "storageType": "generic",

    "attributes": {
        "collection_id": "String",
        "institution": "String",
        "collection_name": "String"
    },

    "associations": {
        "medicinal_plants": {
            "type" : "to_many",
            "target": "specimen",
            "targetKey": "specimen_id",
            "keyIn": "specimen",
            "targetStorageType": "sql",
            "label": "common_name",
            "sublabel": "date"
        },
    },
```


## Data upload

Once the database is generated, you can upload data from a csv file to a Zendro instance directly through the GUI or the API.

### Data format requirements

Data to populate each model in your schema must be in a separate csv file, following the format requirements below:

1. Column names in the first row must correspond to model attributes.
2. Empty values must be represented as `NULL`.
3. Strings, dates and boolean data types must be inside double quotes `""`. Do not quote NULLs or numeric data types.
4. Date and time formats must follow the [RFC 3339](https://tools.ietf.org/html/rfc3339) standard.

### GUI

To upload the csv file through the GUI, go to the model on the left-side panel and use the import button. It will ask you to select a file from your computer and automatically fill the table. 

### API

To upload the csv file through the API, you can make a request from the terminal to:

`curl -XPOST [URL] -H 'Content-Type: mulipart/form-data' -F 'query=mutation{ bulkAdd[Model name]Csv{[Primary key] } }' -F csv_file=@[file path]`

For example:

`curl -XPOST http:/ Zendrodev.conabio.gob.mx:3000/graphql -H 'Content-Type: mulipart/form-data' -F 'query=mutation{ bulkAddSpecimenCsv{specimen_id } }' -F csv_file=@specimen.csv`