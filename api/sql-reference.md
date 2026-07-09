---
layout: default
title: SQL reference
parent: Zendro API
nav_order: 3
permalink: /api/sql-reference
---

# SQL statements in the data model
{: .no_toc }
This page describes the SQL statements used to implement CRUD functionality for models based on the `sql` storage type.
{: .fs-6 .fw-300 }

Zendro uses the promise-based ORM [Sequelize](https://sequelize.org/) to make the needed database calls. Sequelize provides a [Model](https://sequelize.org/master/class/lib/model.js~Model.html) class to represent tables in a database, with instances of this class representing single rows; in Zendro, model classes are extended from this class.

We'll use the model `event` from the [Breeding API](https://github.com/usadellab/EMPHASIS-Layer/tree/master/data_model_definitions) to see how various Zendro commands translate into SQL.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Case 'Create'

The following GraphQL command is given to Zendro:

```graphql
mutation{addEvent(eventType:"Test Event") {eventType}}
```

This is transformed into the following SQL:

```sql
START TRANSACTION;
INSERT INTO "events" ("eventType","createdAt","updatedAt") VALUES ('Test Event','2020-06-03 13:01:11.715 +00:00','2020-06-03 13:01:11.715 +00:00') RETURNING *;
COMMIT;
```

## Case 'Read'

The following GraphQL command is given to Zendro:

```graphql
{events{eventType}}
```

This is transformed into the following SQL:

```sql
SELECT count(*) AS "count" FROM "events" AS "event";
SELECT count(*) AS "count" FROM "events" AS "event";
SELECT "eventType", "eventDbId", "eventDescription", "eventTypeDbId", "studyDbId", "date", "createdAt", "updatedAt" FROM "events" AS "event" LIMIT 1 OFFSET 0;
```

## Case 'Update'

The following GraphQL command is given to Zendro:

```graphql
mutation{updateEvent(eventType:"Test Event", eventDbId:"1") {eventType eventDbId}}
```

This is transformed into the following SQL:

```sql
START TRANSACTION;
SELECT "eventType", "eventDbId", "eventDescription", "eventTypeDbId", "studyDbId", "date", "createdAt", "updatedAt" FROM "events" AS "event" WHERE "event"."eventType" = 'Test Event';
UPDATE "events" SET "eventDbId"='1',"updatedAt"='2020-06-03 13:02:55.800 +00:00' WHERE "eventType" = 'Test Event'
COMMIT;
```

## Case 'Delete'

The following GraphQL command is given to Zendro:

```graphql
mutation{deleteEvent(eventType:"Test Event")}
```

This is transformed into the following SQL:

```sql
SELECT "eventType", "eventDbId", "eventDescription", "eventTypeDbId", "studyDbId", "date", "createdAt", "updatedAt" FROM "events" AS "event" WHERE "event"."eventType" = 'Test Event';
SELECT count(*) AS "count" FROM "eventParameters" AS "eventParameter" WHERE "eventParameter"."eventDbId" = 'Test Event';
SELECT "eventType", "eventDbId", "eventDescription", "eventTypeDbId", "studyDbId", "date", "createdAt", "updatedAt" FROM "events" AS "event" WHERE "event"."eventType" = 'Test Event';
DELETE FROM "events" WHERE "eventType" IN (SELECT "eventType" FROM "events" WHERE "eventType" = 'Test Event' LIMIT 1)
```
