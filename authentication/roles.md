---
layout: default
title: Roles
parent: Authentication
nav_order: 1
permalink: /authentication/roles
---

# Role management
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

By default, Zendro exposes three different roles for its users.

## Decoding the roles from the token
To evaluate roles on the GraphQL server, Zendro decodes the `access_token` and reads the roles out of it. Keycloak sends the user roles as part of the token by default; if any other OAuth2 service is used, that function has to be adapted accordingly. Zendro exposes a `/getRolesForOauth2Token` endpoint on the GraphQL server to read the roles from the decoded token. A `getRoles()` utility function can be adapted to the user's needs.

## administrator
Users with administrator permissions have access to the Keycloak admin-cli and all its functionality, including user management and other Keycloak configuration.

## editor
Users with editor permissions can use Zendro's write API functions (create, update, delete) through the web interfaces or directly via the GraphQL API.

## reader
Users with reader permissions can use Zendro's read API functions (count, read-one, read-many, search, sort, paginate) through the web interfaces or directly via the GraphQL API.
