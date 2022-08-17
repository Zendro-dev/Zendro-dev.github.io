[ &larr; back](setup_root.md)
<br/>
# Enviroment variables

In this file you can find an explanation of all the enviroment variables that are used in Zendro for GraphQL server, the Single page app and the GraphiQL-auth services.

 <br/>

## GraphQL server

### Mandatory
* `ALLOW_ORIGIN` - Sets the `Access-Control-Allow-Origin` header to the specified value.

### Optional (without defaults)
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service.
* `OAUTH2_PUBLIC_KEY`- OAuth2 service public key used to encrypt / verify tokens.
* `OAUTH2_CLIENT_ID`- GraphQL server OAuth2 Client ID.
* `GRAPHIQL_REDIRECT_URI`- RedirectURI of the graphiql-auth client, used to migrate the default keycloak OAuth2 service.
* `SPA_REDIRECT_URI`- RedirectURI of the single-page-app client, used to migrate the default keycloak OAuth2 service.
* `MIGRATION_USERNAME` - Registered username in OAuth2 service, used to fetch token for a distributed setup.
* `MIGRATION_PASSWORD` - Registered password in OAuth2 service, used to fetch token for a distributed setup.

### Optional (with sensible defaults)
* `ERROR_LOG` - Debug logs verbosity. Can be either "verbose" or "compact". Default value is `compact`.
* `EXPORT_TIME_OUT` - Maximum amount of time in milliseconds before the server throws a timeout error when exporting data. Default is `3600`.
* `LIMIT_RECORDS` - Maximum number of records that each request can return, default value is `10000`.
* `PORT` - The port where the app is listening, default value is `3000`.
* `POST_REQUEST_MAX_BODY_SIZE` - Maximum size of the GraphQL request in MB. Default is `1mb`.
* `MAX_TIME_OUT` - Maximum number of milliseconds that a zendro server will wait to connect with another zendro server. Default value is `2000`.
* `REQUIRE_SIGN_IN` - Boolean to toggle the required sign in to the graphql server. Default is `true`.
* `SALT_ROUNDS` - Number of salt rounds when hashing a new password. Default is `10`.
* `WHITELIST_ROLES` - Whitelist of roles to grant permission regardless of user roles. E.g. `"reader,editor"`.
* `DOWN_MIGRATION` - Default is false. User can perform down migration if the table or collection is empty, namely no records. When this env var is true, user can drop all data and metadata for non-empty table. 

## Single-page-app
### Mandatory
* `NEXT_PUBLIC_ZENDRO_GRAPHQL_URL` - GraphQL endpoint address. Used to send data queries and mutations.
* `NEXT_PUBLIC_ZENDRO_METAQUERY_URL` - GraphQL meta-query endpoint address. Used to send meta- queries and mutations.
* `NEXT_PUBLIC_ZENDRO_MAX_UPLOAD_SIZE` - Maximum allowed upload size in megabytes.
* `NEXT_PUBLIC_ZENDRO_MAX_RECORD_LIMIT` - Maximum number of records that can be returned per request.
* `NEXT_PUBLIC_ZENDRO_ROLES_URL` - URL of graphql-server getRolesForOAuth2Token endpoint.
* `ZENDRO_DATA_MODELS` - relative path from the root of the directory to your models folder.

### Mandatory (OAuth2)
* `OAUTH2_ISSUER` - OAuth2 Issuer URL.
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service.
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID.
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret.
* `NEXTAUTH_URL` - When deploying to production, set the NEXTAUTH_URL environment variable to the canonical URL of your site.
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token).

**Note**: `OAUTH2_CLIENT_ID` and `OAUTH2_CLIENT_SECRET` vars will be added automatically if you run the default keycloak migration.

## Graphiql-auth
### Mandatory
* `NEXT_PUBLIC_ZENDRO_GRAPHQL_URL` - GraphQL endpoint address. Used to send data queries and mutations.
* `NEXT_PUBLIC_ZENDRO_METAQUERY_URL` - GraphQL meta-query endpoint address. Used to send meta- queries and mutations.

### Mandatory (OAuth2)
* `OAUTH2_ISSUER` - OAuth2 Issuer URL.
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service.
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID.
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret.
* `NEXTAUTH_URL` - When deploying to production, set the NEXTAUTH_URL environment variable to the canonical URL of your site.
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token).

**Note**: `OAUTH2_CLIENT_ID` and `OAUTH2_CLIENT_SECRET` vars will be added automatically if you run the default keycloak migration.