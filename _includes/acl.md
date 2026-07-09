Zendro can check access rights for every single GraphQL query, based on the currently logged-in user identified by the token. The user is decoded and the corresponding roles are loaded to check access rights. This step is carried out by the [NPM acl package](https://www.npmjs.com/package/acl). The respective access rights can, and must, be declared in the file `./graphql-server/acl_rules.js`.

You can run Zendro with or without this access control check. The default is to run it without checking access rights.

To switch the access rights check on, uncomment the command line switch `acl` (`//,'acl'`) in `./graphql-server/startServer.js`.

You can also whitelist certain roles to own all user permissions, using the `WHITELIST_ROLES` environment variable. For example, to whitelist reading actions, add this to `./graphql-server/.env`:

```
WHITELIST_ROLES="reader"
```

You can add as many roles as you wish, separated by commas.
