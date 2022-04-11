# Authentication and Authorization

When using Zendro in a default setup, Zendro implements authorization and authentication of users and roles via [OAuth2](https://oauth.net/2/) using third-party libraries to provide a secure and stable experience.

As OAuth2 backend, Zendro sets up a [keycloak](https://www.keycloak.org/) server to handle all authentication and user management, which also supports [OpenID Connect](https://openid.net/connect/). Zendro itself, if started with authorization rules enabled, only requires a valid access_token to be present in the request header.

On the front-end web interfaces, Zendro uses [NextAuth.js](https://next-auth.js.org/) to communicate with the keycloak backend.

To connect to the web interfaces we use the recommended OAuth2 [Authorization Code grant](https://oauth.net/2/grant-types/authorization-code/) flow to exchange an authorization for an access token

## keycloak

When setting up a new Zendro instance using the Zendro CLI and the recommended docker setup, a default keycloak migration is provided, that sets up the necessary infrastructure on the keycloak server. This includes creating a default user, default roles and the OAuth2 clients for the different Zendro-interfaces (graphql-server, graphiql-auth, single-page-app). This migration is run automatically on the first start-up of any Zendro installation if not manually removed from the `graphql-server/migrations` folder. For further, more detailed information check out the detailed information on the [keycloak documentation](https://www.keycloak.org/documentation). In the background keycloak 

### user-management
All user management is handled by keycloak. To create new users, set their permissions reset passwords, etc.. use the [keycloak admin-cli](https://www.keycloak.org/docs/latest/server_admin/#admin-cli) to do so.It should be available at
```
<your-url>:8081/auth
```
Also refer to the keycloak documentation on [how to manage users](https://www.keycloak.org/docs/latest/server_admin/#assembly-managing-users_server_administration_guide).

#### Third party identity providers
Keycloak can be [configured to delegate authentication to one or more Identity providers](https://www.keycloak.org/docs/latest/server_admin/#_general-idp-config). Social login via Facebook or Google+ is an example of identity provider federation. You can also hook Keycloak to delegate authentication to any other OpenID Connect or SAML 2.0 Identity provider.

### role-management
By default Zendro exposes three different roles for their users.

#### Decoding the roles from the token
To evaluate roles on the GraphQL server Zendro decodes the access_token and reads the roles out it. Keycloak by default sends the user-roles as part of the token, however if any other OAuth2 service is used, that function has to be adapted accordingly. Zendro exposes a `/getRolesForOauth2Token` on the GraphQL server to read the roles from the decoded token. A `getRoles()` utility function can be used to adapt that functionality according to the users needs.

#### administrator
Users with administrator permissions have access to the keycloak admin-cli and all its functionalities, including user-management and other keycloak configurations.

#### editor
Users with editor permissions can use Zendro's write API functions (create, update, delete) through the web-interfaces or directly via the GraphQL API.

#### reader
Users with read permissions can use Zendro's read API functions (count, read-one, read-many, search, sort, paginate)
through the web-interfaces or directly via the GraphQL API.

### environment variables
To configure the Zendro interfaces to correctly connect to the keycloak (or any other OAuth2) endpoint a set of environment variables is exposed to the User.

#### GraphQL server
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service
* `OAUTH2_PUBLIC_KEY`- OAuth2 service public key used to encrypt / verify tokens
* `OAUTH2_CLIENT_ID`- GraphQL server OAuth2 Client ID
* `GRAPHIQL_REDIRECT_URI`- RedirectURI of the graphiql-auth client, used to migrate the default keycloak OAuth2 service
* `SPA_REDIRECT_URI`- RedirectURI of the single-page-app client, used to migrate the default keycloak OAuth2 service

#### Web-interfaces (GraphiQL & Single-page-app)
* `OAUTH2_ISSUER` - OAuth2 Issuer URL 
* `OAUTH2_TOKEN_URI` - Endpoint of OAuth2 token service 
* `OAUTH2_CLIENT_ID` - SPA OAuth2 Client ID
* `OAUTH2_CLIENT_SECRET` - SPA OAuth2 Client Secret
* `NEXTAUTH_URL` - When deploying to production, set the NEXTAUTH_URL environment variable to the canonical URL of your site.
* `NEXTAUTH_SECRET` - Used to encrypt the NextAuth.js JWT, and to hash [email verification tokens](https://next-auth.js.org/adapters/models#verification-token).


## Using a different Authorization service
In case you do *not* want to use keycloak as the authorization service for your Zendro installation, you can do so by setting the above environment variables to your needs. For the two web interfaces, depending on your configuration needs, you might need to [manually define your provider](https://next-auth.js.org/configuration/providers/oauth#using-a-custom-provider). NextAuth also supports a long list of [built-in providers](https://next-auth.js.org/configuration/providers/oauth#built-in-providers) ready to be used.

You can customize your providers in the `/src/pages/api/auth/[...nextauth]` files. Either reconfigure the default `zendro` keycloak provider or add your custom provider to the `providers` list.

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

To support requesting an access token from the GraphQL server the OAuth2 graphql-server client supports granting tokens through the [password grant type](https://oauth.net/2/grant-types/password/). Those tokens can then be used by any thrid-party script, e.g. python, R, curl, etc.. to request Data via Zendros GraphQL API.

To illustrate an example using curl:

Request a token via
```bash
curl -X POST --url <oauth2-token-url> -d 'Content-Type: application/x-www-form-urlencoded' -d grant_type=password -d client_id=<graphql-server-client_id> -d username=<username> -d password=<password>
```

Send it in the header via
```bash
curl --url <graphql-server>/graphql -X POST -H 'Content-Type: application/json' -H 'Authorization: Bearer <access_token>' -d '{"query": "{ ...<your query> }" }'
```