---
layout: default
title: Authentication
nav_order: 6
has_children: true
---

# Authentication and Authorization
{: .no_toc }
When using Zendro in a default setup, Zendro implements authorization and authentication of users and roles via [OAuth2](https://oauth.net/2/), using third-party libraries to provide a secure and stable experience.
{: .fs-6 .fw-300 }

As an OAuth2 backend, Zendro sets up a [keycloak](https://www.keycloak.org/) server to handle all authentication and user management, which also supports [OpenID Connect](https://openid.net/connect/). Zendro itself, when started with authorization rules enabled, only requires a valid `access_token` to be present in the request header.

On the front-end web interfaces, Zendro uses [NextAuth.js](https://next-auth.js.org/) to communicate with the Keycloak backend, using the recommended OAuth2 [Authorization Code grant](https://oauth.net/2/grant-types/authorization-code/) flow to exchange an authorization code for an access token.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
## Keycloak

When setting up a new Zendro instance using the Zendro CLI and the recommended docker setup, a default Keycloak migration sets up the necessary infrastructure on the Keycloak server. This includes creating a default user, default roles, and the OAuth2 clients for the different Zendro interfaces (graphql-server, graphiql-auth, single-page-app). This migration runs automatically on the first startup of any Zendro installation, unless manually removed from the `graphql-server/migrations` folder. For more detail, see the [Keycloak documentation](https://www.keycloak.org/documentation).

### User management
All user management is handled by Keycloak. To create new users, set their permissions, reset passwords, etc., use the [Keycloak admin-cli](https://www.keycloak.org/docs/latest/server_admin/#admin-cli), available by default at:

```
<your-url>:8081/auth
```

Also refer to the Keycloak documentation on [how to manage users](https://www.keycloak.org/docs/latest/server_admin/#assembly-managing-users_server_administration_guide).

#### Third-party identity providers
Keycloak can be [configured to delegate authentication to one or more identity providers](https://www.keycloak.org/docs/latest/server_admin/#_general-idp-config) — social login via Facebook or Google is one example of identity provider federation. You can also hook Keycloak up to delegate authentication to any other OpenID Connect or SAML 2.0 identity provider.

### Role management
By default, Zendro exposes three different roles for its users — see [Roles]({% link authentication/roles.md %}) for the full breakdown of `administrator`, `editor` and `reader`, and how roles are decoded from the access token.

### Environment variables
To configure the Zendro interfaces to correctly connect to Keycloak (or any other OAuth2 endpoint), a set of environment variables is exposed to the user — see [Environment variables]({% link getting-started/environment-variables.md %}) for the full reference. The most relevant ones here:

#### GraphQL server
* `OAUTH2_TOKEN_URI` - Endpoint of the OAuth2 token service
* `OAUTH2_PUBLIC_KEY`- OAuth2 service public key used to encrypt / verify tokens
* `OAUTH2_CLIENT_ID`- GraphQL server OAuth2 Client ID
* `GRAPHIQL_REDIRECT_URI`- RedirectURI of the graphiql-auth client, used to migrate the default Keycloak OAuth2 service
* `SPA_REDIRECT_URI`- RedirectURI of the single-page-app client, used to migrate the default Keycloak OAuth2 service

#### Web interfaces (GraphiQL & Single-page-app)
* `OAUTH2_ISSUER` - OAuth2 Issuer URL
* `OAUTH2_TOKEN_URI` - Endpoint of the OAuth2 token service
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret
* `NEXTAUTH_URL` - When deploying to production, set this to the canonical URL of your site
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token)

## Using a different authorization service
If you do *not* want to use Keycloak as the authorization service for your Zendro installation, set the environment variables above to your needs. For the two web interfaces, depending on your configuration needs, you might need to [manually define your provider](https://next-auth.js.org/configuration/providers/oauth#using-a-custom-provider). NextAuth also supports a long list of [built-in providers](https://next-auth.js.org/configuration/providers/oauth#built-in-providers) ready to use.

You can customize your providers in the `/src/pages/api/auth/[...nextauth]` files — either reconfigure the default `zendro` Keycloak provider, or add your custom provider to the `providers` list:

```javascript
providers: [
  // ...add more providers here
  {
    id: 'zendro',
    // ...
  },
  {
	// add your custom provider
  }
],
```

## Requesting an access token for third-party requests

To support requesting an access token from the GraphQL server, the OAuth2 graphql-server client supports the [password grant type](https://oauth.net/2/grant-types/password/). Those tokens can then be used by any third-party script — e.g. Python, R, curl — to request data via Zendro's GraphQL API.

Example using curl:

Request a token:
```bash
curl -X POST --url <oauth2-token-url> -d 'Content-Type: application/x-www-form-urlencoded' -d grant_type=password -d client_id=<graphql-server-client_id> -d username=<username> -d password=<password>
```

Send it in the header:
```bash
curl --url <graphql-server>/graphql -X POST -H 'Content-Type: application/json' -H 'Authorization: Bearer <access_token>' -d '{"query": "{ ...<your query> }" }'
```
