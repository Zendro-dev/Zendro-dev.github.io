[&larr; back](api_root.md)
<br/>

# SQL statements in the data model

This document describes the SQL statements that are used for implementing CRUD functionality for models that are based on the `sql` storage type.

Zendro uses the promise-based ORM [Sequelize](https://sequelize.org/) to make the needed database calls. Sequelize provides a class [Model](https://sequelize.org/master/class/lib/model.js~Model.html) to represent tables in a database, with the instances of this class being single rows in this table. In Zendro, the model classes are extended from this class.

We will use the model `event` from the [Breeding API](https://github.com/usadellab/EMPHASIS-Layer/tree/master/data_model_definitions) to see how the various Zendro commands are translated into SQL.

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
