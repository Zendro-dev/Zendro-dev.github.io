[ &larr; back](setup_root.md)
<br/>
# Enviroment variables

In this file you can find an explanation of all the enviroment variables that are used in Zendro for GraphQL server, the Single page app and the GraphiQL-auth services.

 <br/>

## GraphQL server

### Mandatory
* `ALLOW_ORIGIN` - Sets the `Access-Control-Allow-Origin` header to the specified value.

### Optional (without defaults)
* `MAIL_SERVICE` - For bulk add operations, the email service to use for sending progress reports.
* `MAIL_HOST` - Email service host (usually SMTP config).
* `MAIL_ACCOUNT` - Sender email account address.
* `MAIL_PASSWORD` - Sender email account password.
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service
* `OAUTH2_PUBLIC_KEY`- OAuth2 service public key used to encrypt / verify tokens
* `OAUTH2_CLIENT_ID`- GraphQL server OAuth2 Client ID
* `GRAPHIQL_REDIRECT_URI`- RedirectURI of the graphiql-auth client, used to migrate the default keycloak OAuth2 service
* `SPA_REDIRECT_URI`- RedirectURI of the single-page-app client, used to migrate the default keycloak OAuth2 service

### Optional (with sensible defaults)
* `ERROR_LOG` - Debug logs verbosity. Can be either "verbose" or "compact". Default value is `compact`.
* `EXPORT_TIME_OUT` - Maximum amount of time in milliseconds before the server throws a timeout error when exporting data. Default is `3600`.
* `LIMIT_RECORDS` - Maximum number of records that each request can return, default value is `10000`.
* `PORT` - The port where the app is listening, default value is `3000`
* `POST_REQUEST_MAX_BODY_SIZE` - Maximum size of the GraphQL request in MB. Default is `1mb`.
* `MAX_TIME_OUT` - Maximum number of milliseconds that a zendro server will wait to connect with another zendro server. Default value is `2000`.
* `REQUIRE_SIGN_IN` - Boolean to toggle the required sign in to the graphql server. Default is `true`.
* `SALT_ROUNDS` - Number of salt rounds when hashing a new password. Default is `10`.

## Single-page-app
### Mandatory
* `NEXT_PUBLIC_ZENDRO_GRAPHQL_URL` - GraphQL endpoint address. Used to send data queries and mutations.
* `NEXT_PUBLIC_ZENDRO_EXPORT_URL` - Endpoint export address. Used to request table downloads in CSV format.
* `NEXT_PUBLIC_ZENDRO_METAQUERY_URL` - GraphQL meta-query endpoint address. Used to send meta- queries and mutations.
* `NEXT_PUBLIC_ZENDRO_MAX_UPLOAD_SIZE` - Maximum allowed upload size in megabytes.
* `NEXT_PUBLIC_ZENDRO_MAX_RECORD_LIMIT` - Maximum number of records that can be returned per request.
* `ZENDRO_DATA_MODELS` - relative path from the root of the directory to your models folder

### Mandatory (OAuth2)
* `OAUTH2_ISSUER` - OAuth2 Issuer URL 
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service 
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret
* `NEXTAUTH_URL` - When deploying to production, set the NEXTAUTH_URL environment variable to the canonical URL of your site.
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token).

## Graphiql-auth
### Mandatory
* `NEXT_PUBLIC_ZENDRO_GRAPHQL_URL` - GraphQL endpoint address. Used to send data queries and mutations.
* `NEXT_PUBLIC_ZENDRO_METAQUERY_URL` - GraphQL meta-query endpoint address. Used to send meta- queries and mutations.

### Mandatory (OAuth2)
* `OAUTH2_ISSUER` - OAuth2 Issuer URL 
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service 
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret
* `NEXTAUTH_URL` - When deploying to production, set the NEXTAUTH_URL environment variable to the canonical URL of your site.
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token).
