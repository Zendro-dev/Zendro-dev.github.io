---
layout: default
title: Roles
parent: Authentication
nav_order: 1
permalink: /oauth/data_models/validation
---


# Role management
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Role management
By default Zendro exposes three different roles for their users.

### Decoding the roles from the token
To evaluate roles on the GraphQL server Zendro decodes the access_token and reads the roles out it. Keycloak by default sends the user-roles as part of the token, however if any other OAuth2 service is used, that function has to be adapted accordingly. Zendro exposes a `/getRolesForOauth2Token` on the GraphQL server to read the roles from the decoded token. A `getRoles()` utility function can be used to adapt that functionality according to the users needs.

### administrator
Users with administrator permissions have access to the keycloak admin-cli and all its functionalities, including user-management and other keycloak configurations.

### editor
Users with editor permissions can use Zendro's write API functions (create, update, delete) through the web-interfaces or directly via the GraphQL API.

### reader
Users with read permissions can use Zendro's read API functions (count, read-one, read-many, search, sort, paginate)
through the web-interfaces or directly via the GraphQL API.