---
layout: default
title: Environment variables
parent: Getting started
nav_order: 2
permalink: /getting-started/environment-variables
---

# Environment variables
{: .no_toc }
This page explains all the environment variables used by Zendro's GraphQL server, the Single Page App and GraphiQL.
{: .fs-6 .fw-300 }

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

# graphql-server's own Keycloak client ID, used to verify incoming bearer
# tokens (with OAUTH2_PUBLIC_KEY above) and read that client's role mappings
# out of them. This is graphql-server acting as a resource server - a
# different job (and a different Keycloak client) from OAUTH2_GRAPHIQL_CLIENT_ID
# below, which is graphql-server acting as an OAuth2 client on GraphiQL's behalf.
OAUTH2_CLIENT_ID=

# Comma-separated list of redirect URIs graphql-server's own auth backend
# (login/callback/logout) is allowed to send users back to after signing in
# via Keycloak - one per SPA or GraphiQL deployment that talks to this
# server, e.g. `http://localhost:8080/api/auth/callback/zendro`.
AUTH_REDIRECT_URI=

# RedirectURI of the single-page-app client, used to migrate the default Keycloak OAuth2 service.
SPA_REDIRECT_URI=

# Registered username in the OAuth2 service, used to fetch a token for a distributed setup.
MIGRATION_USERNAME=

# Registered password in the OAuth2 service, used to fetch a token for a distributed setup.
MIGRATION_PASSWORD=

# Whitelist of roles that are granted all user permissions regardless of ACL rules. E.g. `"reader,editor"`.
WHITELIST_ROLES=
```

### Zendro's own auth backend (for GraphiQL)
Since GraphiQL is a thin client with no Keycloak credentials of its own, graphql-server runs the actual login/callback/logout flow on its behalf (see [Authentication]({% link authentication/index.md %})). These only matter when `AUTH_ENABLED="true"`:

```properties
# Toggles graphql-server's own /auth/* routes (login/callback/logout) used by GraphiQL.
AUTH_ENABLED=false

# Enables GraphiQL's jq/JSONPath filter panel, which POSTs to /meta_query.
GRAPHIQL_FILTER_ENABLED=false

# OAuth2 Client ID graphql-server itself uses when it talks to Keycloak on
# GraphiQL's behalf, as registered in Keycloak. Not used by GraphiQL directly.
OAUTH2_GRAPHIQL_CLIENT_ID=zendro_graphiql

# The matching Client Secret for the ID above.
OAUTH2_GRAPHIQL_CLIENT_SECRET=

# The realm's OIDC issuer, as reachable by browsers - endpoints are
# discovered from "<OAUTH2_GRAPHIQL_ISSUER_URI>/.well-known/openid-configuration".
# Must exactly match what Keycloak itself reports as its issuer.
OAUTH2_GRAPHIQL_ISSUER_URI=

# Only needed when the URI above isn't reachable from this process - e.g.
# Keycloak in Docker Compose, where the issuer above is the host's published
# port but graphql-server must connect via the internal service hostname.
OAUTH2_GRAPHIQL_ISSUER_INTERNAL_URI=

# Encrypts graphql-server's own login session cookie. Set via
# `zendro set-session-secret gqs <secret>` - see the CLI reference.
SESSION_SECRET=
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

# File delimiter to differentiate between records when exporting data. Default is `\n`.
RECORD_DELIMITER=

# File delimiter to differentiate between record fields when exporting data. Default is `,`.
FIELD_DELIMITER=

# File delimiter to differentiate between array-field elements when exporting data. Default is `;`.
ARRAY_DELIMITER=

# Sheet name to use when exporting data to xlsx.
SHEET_NAME=
```

### Mail notifications (optional)
Used to notify users by email when a bulk-add operation finishes. If unset, this feature is silently disabled — a warning is logged on startup, nothing else changes.
```properties
MAIL_SERVICE=
MAIL_HOST=
MAIL_ACCOUNT=
MAIL_PASSWORD=
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

# Sheet name to use when exporting data to xlsx.
SHEET_NAME=

# Logs Redux state changes to the browser console - useful when debugging the SPA itself.
NEXT_PUBLIC_REDUX_LOGGER=false
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
OAUTH2_LOGOUT_URL=

# SPA OAuth2 Client ID.
OAUTH2_CLIENT_ID=

# SPA OAuth2 Client Secret.
OAUTH2_CLIENT_SECRET=

# When deploying to production, set this to the canonical URL of your site.
NEXTAUTH_URL=

# Used to encrypt the NextAuth.js JWT, and to hash email verification tokens:
# https://next-auth.js.org/adapters/models#verification-token
#
# Set via `zendro set-session-secret spa <secret>` - see the CLI reference.
NEXTAUTH_SECRET=
```

**Note**: `OAUTH2_CLIENT_ID` and `OAUTH2_CLIENT_SECRET` are added automatically if you run the default Keycloak migration.

## GraphiQL

GraphiQL is a thin client — it holds no Keycloak credentials of its own and runs no auth logic; graphql-server's `/auth/*` routes (see above) handle login/callback/logout on its behalf.

### Mandatory
```properties
# The graphql-server this GraphiQL deployment talks to. /graphql,
# /meta_query and /auth/* (login/logout) are all derived from this one origin.
ZENDRO_SERVER_URL=
```

### Optional
```properties
# The port GraphiQL listens on.
PORT=7070

# This app's own public origin. Defaults to "http://localhost:<PORT>" if
# unset, so changing PORT alone is enough for local use - only set this
# explicitly when that default is wrong (different host/protocol, e.g.
# behind a reverse proxy). Sent to graphql-server so it knows which origin
# to run login/logout on behalf of - it must be registered there too, in
# graphql-server's own AUTH_REDIRECT_URI.
ORIGIN=

# Enables the jq/JSONPath filter panel (POSTs to <ZENDRO_SERVER_URL>/meta_query).
GRAPHIQL_FILTER_ENABLED=true

# Where graphql-server runs its auth backend - a fixed path, independent of
# graphql-server's own GraphiQL mount. Only worth overriding if something in
# front of graphql-server rewrites it.
ZENDRO_AUTH_PATH=/auth

# Maximum size of the GraphQL request in MB.
POST_REQUEST_MAX_BODY_SIZE=1mb
```
