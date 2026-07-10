---
layout: default
title: Environment variables
parent: Getting started
nav_order: 2
permalink: /getting-started/environment-variables
---

# Environment variables
{: .no_toc }
This page explains all the environment variables used by Zendro's GraphQL server, the Single Page App and the GraphiQL-auth service.

The most convenient way to set them is via a `.env` file in the root directory of the respective sub-project. Be aware that when using Zendro with docker, the docker-compose file also expects some environment variables, which can be set in a `.env` file in the root directory. See also the `.env.example` files for inspiration.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## GraphQL server

### Mandatory
```properties
# Sets the `Access-Control-Allow-Origin` header to the specified value.
ALLOW_ORIGIN=
```

### Optional (without defaults)
```properties
# Endpoint of OAuth2 token service.
OAUTH2_TOKEN_URI=

# OAuth2 service public key used to encrypt / verify tokens.
OAUTH2_PUBLIC_KEY=

# GraphQL server OAuth2 Client ID.
OAUTH2_CLIENT_ID=

# RedirectURI of the graphiql-auth client, used to migrate the default Keycloak OAuth2 service.
GRAPHIQL_REDIRECT_URI=

# RedirectURI of the single-page-app client, used to migrate the default Keycloak OAuth2 service.
SPA_REDIRECT_URI=

# Registered username in the OAuth2 service, used to fetch a token for a distributed setup.
MIGRATION_USERNAME=

# Registered password in the OAuth2 service, used to fetch a token for a distributed setup.
MIGRATION_PASSWORD=

# Whitelist of roles that are granted all user permissions regardless of ACL rules. E.g. `"reader,editor"`.
WHITELIST_ROLES=
```

### Optional (with sensible defaults)
```properties
# Debug log verbosity. Can be either `verbose` or `compact`.
ERROR_LOG=compact

# Maximum amount of time in milliseconds before the server throws a timeout error when exporting data.
EXPORT_TIME_OUT=3600

# Maximum number of records that each request can return.
LIMIT_RECORDS=10000

# The port where the app is listening.
PORT=3000

# Maximum size of the GraphQL request in MB.
POST_REQUEST_MAX_BODY_SIZE=1mb

# Maximum number of milliseconds a Zendro server will wait to connect with another Zendro server.
MAX_TIME_OUT=2000

# Boolean to toggle whether sign-in is required for the graphql server.
REQUIRE_SIGN_IN=true

# Number of salt rounds when hashing a new password.
SALT_ROUNDS=10

# When `true`, users can drop all data and metadata for a non-empty table;
# otherwise, down migration is only allowed if the table or collection is empty (no records).
DOWN_MIGRATION=false
```

## Single-page-app
### Mandatory
```properties
# GraphQL endpoint address. Used to send data queries and mutations.
NEXT_PUBLIC_ZENDRO_GRAPHQL_URL=

# GraphQL meta-query endpoint address. Used to send meta-queries and mutations.
NEXT_PUBLIC_ZENDRO_METAQUERY_URL=

# Maximum allowed upload size in megabytes.
NEXT_PUBLIC_ZENDRO_MAX_UPLOAD_SIZE=

# Maximum number of records that can be returned per request.
NEXT_PUBLIC_ZENDRO_MAX_RECORD_LIMIT=

# URL of the graphql-server's `getRolesForOAuth2Token` endpoint.
NEXT_PUBLIC_ZENDRO_ROLES_URL=

# Relative path from the root of the directory to your models folder.
ZENDRO_DATA_MODELS=
```

### Optional
```properties
# Custom basepath for the application.
NEXT_PUBLIC_ZENDRO_BASEPATH=

# File delimiter to differentiate between records. Default is `\n`.
RECORD_DELIMITER=

# File delimiter to differentiate between record fields. Default is `,`.
FIELD_DELIMITER=

# File delimiter to differentiate between array-field elements. Default is `;`.
ARRAY_DELIMITER=
```

### OAuth2
```properties
# OAuth2 Issuer URL.
OAUTH2_ISSUER=

# Endpoint of the OAuth2 auth service. Can be used instead of `OAUTH2_ISSUER` if you want to use a non-OIDC custom provider.
# See the NextAuth custom provider docs for more information:
# https://next-auth.js.org/configuration/providers/oauth#using-a-custom-provider
OAUTH2_AUTH_URI=

# Endpoint of the OAuth2 token service.
OAUTH2_TOKEN_URI=

# Logout endpoint of the OAuth2 server. If not provided, `OAUTH2_ISSUER` is used with the standard OIDC logout route.
OAUTH2_LOGOUT_URI=

# SPA OAuth2 Client ID.
OAUTH2_CLIENT_ID=

# SPA OAuth2 Client Secret.
OAUTH2_CLIENT_SECRET=

# When deploying to production, set this to the canonical URL of your site.
NEXTAUTH_URL=

# Used to encrypt the NextAuth.js JWT, and to hash email verification tokens:
# https://next-auth.js.org/adapters/models#verification-token
NEXTAUTH_SECRET=
```

**Note**: `OAUTH2_CLIENT_ID` and `OAUTH2_CLIENT_SECRET` are added automatically if you run the default Keycloak migration.

## Graphiql-auth
### Mandatory
```properties
# GraphQL endpoint address. Used to send data queries and mutations.
NEXT_PUBLIC_ZENDRO_GRAPHQL_URL=

# GraphQL meta-query endpoint address. Used to send meta-queries and mutations.
NEXT_PUBLIC_ZENDRO_METAQUERY_URL=
```

### Optional
```properties
# Custom basepath for the application.
NEXT_PUBLIC_ZENDRO_BASEPATH=
```

### OAuth2
```properties
# OAuth2 Issuer URL.
OAUTH2_ISSUER=

# Endpoint of the OAuth2 auth service. Can be used instead of `OAUTH2_ISSUER` if you want to use a non-OIDC custom provider.
# See the NextAuth custom provider docs for more information:
# https://next-auth.js.org/configuration/providers/oauth#using-a-custom-provider
OAUTH2_AUTH_URI=

# Endpoint of the OAuth2 token service.
OAUTH2_TOKEN_URI=

# Logout endpoint of the OAuth2 server. If not provided, `OAUTH2_ISSUER` is used with the standard OIDC logout route.
OAUTH2_LOGOUT_URI=

# SPA OAuth2 Client ID.
OAUTH2_CLIENT_ID=

# SPA OAuth2 Client Secret.
OAUTH2_CLIENT_SECRET=

# When deploying to production, set this to the canonical URL of your site.
NEXTAUTH_URL=

# Used to encrypt the NextAuth.js JWT, and to hash email verification tokens:
# https://next-auth.js.org/adapters/models#verification-token
#
# set by `zendro set-next-auth-secret`
NEXTAUTH_SECRET=
```

**Note**: `OAUTH2_CLIENT_ID` and `OAUTH2_CLIENT_SECRET` are added automatically if you run the default Keycloak migration.
