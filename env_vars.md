---
layout: default
title: Environment variables
parent: Getting started
nav_order: 1
permalink: /setup_root/env_var
---

# Environment variables
{: .no_toc }
In this file you can find an explanation of all the environment variables that are used in Zendro for GraphQL server, the Single page app and the GraphiQL-auth services.

The most convenient way to use them is to setup a .env file in the root directory of the respective sub-project and set the variables as needed. Be aware that when using zendro with docker, the docker-compose file also expects some sensible environment variables that can be set in a .env file in the root directory. Also see the `.env.example` files for inspiration.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
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

### Optional
* `NEXT_PUBLIC_ZENDRO_BASEPATH` - custom basepath for the application
* `RECORD_DELIMITER` - File delimiter to differentiate between records. Default is '\n'
* `FIELD_DELIMITER` - File delimiter to differentiate between record fields. Default is ','
* `ARRAY_DELIMITER` - File delimiter to differentiate between Array-field elements. Default is ';'

### OAuth2
* `OAUTH2_ISSUER` - OAuth2 Issuer URL.
* `OAUTH2_AUTH_URI` - Endpoint OAuth2 auth service. Can be used instead of OAUTH2_ISSUER in case you want to use a non-OIDC custom provider. See https://next-auth.js.org/configuration/providers/oauth#using-a-custom-provider for more information.
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service.
* `OAUTH2_LOGOUT_URI` - Logout Endpoint of OAuth2 server. If not provided the OAUTH2_ISSUER is used to use the standard OIDC logout route.
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID.
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret.
* `NEXTAUTH_URL` - When deploying to production, set the NEXTAUTH_URL environment variable to the canonical URL of your site.
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token).

**Note**: `OAUTH2_CLIENT_ID` and `OAUTH2_CLIENT_SECRET` vars will be added automatically if you run the default keycloak migration.

## Graphiql-auth
### Mandatory
* `NEXT_PUBLIC_ZENDRO_GRAPHQL_URL` - GraphQL endpoint address. Used to send data queries and mutations.
* `NEXT_PUBLIC_ZENDRO_METAQUERY_URL` - GraphQL meta-query endpoint address. Used to send meta- queries and mutations.

### Optional
* `NEXT_PUBLIC_ZENDRO_BASEPATH` - custom basepath for the application

### OAuth2
* `OAUTH2_ISSUER` - OAuth2 Issuer URL.
* `OAUTH2_AUTH_URI` - Endpoint OAuth2 auth service. Can be used instead of OAUTH2_ISSUER in case you want to use a non-OIDC custom provider. See https://next-auth.js.org/configuration/providers/oauth#using-a-custom-provider for more information.
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service.
* `OAUTH2_LOGOUT_URI` - Logout Endpoint of OAuth2 server. If not provided the OAUTH2_ISSUER is used to use the standard OIDC logout route.
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID.
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret.
* `NEXTAUTH_URL` - When deploying to production, set the NEXTAUTH_URL environment variable to the canonical URL of your site.
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token).

**Note**: `OAUTH2_CLIENT_ID` and `OAUTH2_CLIENT_SECRET` vars will be added automatically if you run the default keycloak migration.