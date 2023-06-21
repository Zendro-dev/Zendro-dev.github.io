---
layout: default
title: ACL
parent: Zendro API
nav_order: 1
permalink: /api_root/acl
---

# Access Control List
{: .no_toc }

Zendro can be used checking access rights for every single GraphQL query received by the currently logged in user identified by the Token. The user is decoded and the corresponding roles are loaded to check access rights. This step is carried out by the [NPM acl package](https://www.npmjs.com/package/acl). Respective access rights can and must be declared in the file `./graphql-server/acl_rules.js`.

You can run Zendro with or without this access control check. The default is to run it without checking access rights.

To switch access right check on, you must uncomment the command line switch acl and change the following line in `./graphql-server/migrateDbAndStartServer.sh`

```
npm start #acl
```
to 
```
npm start acl
```

Moreover, you can whitelist certain roles, which can own all user permissions, by using *WHITELIST_ROLES* environment variable. For example, if you wish to whitelist reading actions, modify in `./graphql-server/.env`:

```
WHITELIST_ROLES="reader"
```

You can add all roles you wish separating them with a comma.