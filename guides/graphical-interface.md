---
layout: default
title: Graphical interface
parent: User guides
nav_order: 1
permalink: /guides/graphical-interface
---
# How to use Zendro's graphical user interface
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Zendro's graphical point-and-click user interface is accessible in a web browser, as a single page application (SPA).

To explore how it looks, take a look at the screenshots below, or try it out live: go to [https://zendro.conabio.gob.mx/spa](https://zendro.conabio.gob.mx/spa), a dummy Zendro instance set up for you to try.

Zendro's home graphical interface looks similar to the image below. It can, of course, be customised (by whoever installed Zendro) to show whatever you prefer:

![SPA_home.png](/figures/SPA_home.png)

All data administration functions in the graphical interface are subject to user-based access control: a user only sees the respective icons, buttons, and even model names in the main menu, if they have the required access rights for the corresponding read or write operations.

## Login

Clicking LOGIN prompts you for your username and password. Zendro's graphical interface lets users Create, Read, Update or Delete (CRUD) records, but you decide which users can do what — for instance, only one or two people on a research team may have edit rights to create, update or delete records, while several other members could be allowed to read only.

We created a user with read-only access for you to explore. Log in with the user shown below and the password *reader*.

![SPA_login.png](/figures/SPA_login.png)

## Exploring data

Upon successful sign-in, the user is presented with an overview menu on the left, with one entry per data model — Zendro creates a table for each of the data models provided when setting it up.

In this dummy example, the models are "City", "River" and "Country". The home page is blank by default, but you can use this area for models documentation or a project introduction.

![SPA_models1.png](/figures/SPA_models1.png)

Clicking on a data model name presents the **main data model table**, with a column for each field in the data model (in this example, "city_id", "name" and "population" for the "city" model) and a row for each record.

![SPA_models2.png](/figures/SPA_models2.png)

If you're exploring a table with lots of rows, you can modify the number of rows shown at a time by clicking the number in the lower-left corner — the number of pages adjusts automatically. Try that in the river model:

![SPA_rivers.png](/figures/SPA_rivers.png)

At the bottom right of the table, you can skip forward or backward through pages:

![SPA_paginationmenu.png](/figures/SPA_paginationmenu.png)

You can hide or expand the data models menu by clicking the ">" icon:

![SPA_hidemenu.png](/figures/SPA_hidemenu.png)

Clicking a column name sorts the data by that column in ascending order. Above the table, you can enter search terms, which are matched against any string (text) column, and against numeric columns if the term can be translated to a number.

For each record, the table offers the option to open its detailed view ("eye" icon). Users with edit and delete permissions also see options to edit ("pencil" icon) or delete ("trash" icon) the record:

![SPA_crud.png](/figures/SPA_crud.png)

You may have noticed that only the "eye" icon is visible above — that's because the public user for this tutorial only has reader permissions and cannot modify existing data.

Clicking the "eye" icon opens the detailed view, with full inspection of all details ("ATTRIBUTES" tab) of a single record:

![SPA_models3.png](/figures/SPA_models3.png)

If the user has edit permissions, they also see buttons to open the edit and delete forms — see [Editing data](#editing-data) below.

![SPA_editoptions.png](/figures/SPA_editoptions.png)

In the detailed view, all users also see an "ASSOCIATIONS" tab at the top right, listing one entry per association the record's data model has. In this case, the city "Aachen" is associated with "COUNTRY" and "CAPITALTO".

![SPA_models4.png](/figures/SPA_models4.png)

Clicking an association name lets the user inspect the associated records. For a one-to-many or many-to-many association, a table is shown just like the main table, with the same sorting, search, and pagination functions.

![SPA_models5.png](/figures/SPA_models5.png)

## Editing data

### Modify or delete existing records

For a user with full CRUD permissions, the main table offers options to open the detailed view ("eye" icon), edit ("pencil" icon), or delete ("trash" icon) a given record:

![SPA_crud.png](/figures/SPA_crud.png)

Clicking the "pencil" icon, either in the main table or the detail view, opens the edit form, pre-filled with the record's data. Here you can change the data; a validation error shows up if the data is invalid.

![SPA_editoptions.png](/figures/SPA_editoptions.png)

For example, entering text in the "population" field, which is defined as an integer, prompts you to enter a valid integer instead.

![SPA_validation.png](/figures/SPA_validation.png)

All fields can be modified in this form, except the ID, which links to the record's associations.

To edit associations, on the "ASSOCIATIONS" tab click the name of the data model containing the record to associate, e.g. "country".

This opens a modified version of that data model's main table, with an additional column to associate or dissociate records with the one being edited. Associated records are marked with a "link chain" icon, unassociated ones with a "broken chain link" icon. You can mark several association changes here, to be executed once "save" is clicked.

![SPA_associatons_edit.png](/figures/SPA_associatons_edit.png)

Clicking an associated/unassociated icon flips it to its counterpart — a connected chain link becomes broken, or vice versa. Icons that have been changed are highlighted in color: green connected chain links indicate an association will be established, red broken links indicate the opposite.

![SPA_associations_color.png](/figures/SPA_associations_color.png)

You can paginate through the whole set of records for the association and mark as many changes as you like — they're collected and carried out once "save" is clicked. You can also apply filters, including filtering by already-associated records, or by records marked to be added or removed.

Once the desired associations have been made, click "save" in the top right menu:

![SPA_associatons_save.png](/figures/SPA_associatons_save.png)

Alternatively, if the record you want to associate doesn't exist yet, create it directly by clicking the "+" icon in the top right menu:

![SPA_associations_create.png](/figures/SPA_associations_create.png)

### Add new records

If your user has edit permissions, the top bar of the main data model table shows buttons to Reload the data ("circle arrow"), Add a new record ("+"), Import data ("bold top arrow"), Download data as CSV ("bold down arrow") and Download the model template ("light down arrow"). Read-only users only see the Reload and Download icons.

![SPA_topmenu.png](/figures/SPA_topmenu.png)

These buttons work independently for each data model table — e.g. if you're in the "city" model, you can create or download city records, but to add a country you need to switch to the "country" model in the left side menu.

#### Add a single record manually

The "+" icon lets you **add a single new data record** to the current data table. Clicking it presents a form for entering values for each field.

![SPA_addsingle.png](/figures/SPA_addsingle.png)

If invalid data is entered and you try to save it, validation error messages appear, as in the edit example above.

Once you're done, click the "save" icon. You don't need to fill in every field, but you'll be asked to confirm leaving some fields blank — click "yes" to proceed:

![SPA_addblankok.png](/figures/SPA_addblankok.png)

Your new record is saved. Click the "table" icon at the top to see the table again:

![SPA_addsucess.png](/figures/SPA_addsucess.png)

#### Add several records from a file

Adding single records is sometimes useful, but many users want to add data in bulk — from a table created in MS Excel, recorded with a digital device, or by any other means. You can import an **Excel file** (.xlsx), a **comma separated value file** (.csv) or a **JSON** file. See [How to import and export data]({% link guides/data-import-export.md %}) for the exact format requirements for each option, and how to download a ready-to-fill model template.

To upload your data, click `Import data` ("bold top arrow" icon) in the top right menu, select your file and click **Upload**:

![SPA_addExcel.png](/figures/SPA_addExcel.png)

If there are no errors, your data is uploaded — click Reload if you don't see it right away. If there are errors, you'll be told what the problem is; check your data and try again.

## Download data

To download data, click `Download data` (bold down arrow icon). You'll be prompted to select a directory and file name.

![SPA_download.png](/figures/SPA_download.png)

The data is saved in .csv format, which you can open in Excel or import into statistical software like R. This only downloads the data of one table at a time — see [How to import and export data]({% link guides/data-import-export.md %}) for details, and Zendro's [GraphQL API]({% link guides/graphql-basics.md %}) for how to build complex queries that combine columns from different models.
