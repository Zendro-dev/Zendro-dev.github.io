# How to document zendro
We have two options to document zendro:

1. Modify directly in github interface
    - Go to the file you are going to update
    - Edit file
    - Commit changes directly to the main branch or create a new branch for your commit and start a pull request
    - If you commit directly to the main branch, wait a few minutes before you can see your changes in the [github page](https://zendro-dev.github.io/).
2. Clone the repository, do the changes and push them
    - `git clone https://github.com/Zendro-dev/Zendro-dev.github.io.git`
    - Edit files
    - `git add <file>`
    - `git commit -m "<your-commit-message>"`
    - `git push`
    - Wait a few minutes before you can see your changes in the [github page](https://zendro-dev.github.io/).

In both options we have to know .md files are presented as .html files in the github page. We also have to know that every change we do in the files, it is going to take a while (5~ minutes) to show in the [github page](https://zendro-dev.github.io/).

In order to have the lateral table content, we have to indicate at the beggining of the file the position of the file in the lateral table, for example:

```
---
layout: default
title: For users
nav_order: 2
has_children: true
---
```

Here we are indicating that the title is going to be "For users", it has the 2nd position in the table of content and it has children. As the position has been written manually, if we add some page in the principal menu, we have to update the positions in all principal .md files.

For children files, we have these options:

```
---
layout: default
title: What are data models?
parent: For users
nav_order: 1
permalink: /usage/data_models
--- 
```

Here we are indicating the title, who is its parent, what is the position that it is going to have in the submenu and the link we want to show in the github page. As we can see, the title and the link are related but it is hard to know what name has the .md file, for that reason we did the table of content with the name of the files. Please update this file if you add another page to the lateral menu.

- Home -> `index.md`
- For users -> `usage.md`
    - What are data models? -> `what_are_data_models.md`
    - GraphQL & API -> `GraphQL_intro.md`
    - GraphQL filtering -> `jq_jsonpath.md`
    - Graphical UI -> `SPA_howto.md`
- Quick start -> `quickstart.md`
- Getting started -> `setup_root.md`
    - Environment variables -> `env_vars.md`
    - Data models -> `setup_data_scheme.md`
        - Resolver and Model Layers -> `model_resolver_layers.md`
        - Data validation -> `data_validation.md`
        - Cassandra storageType -> `cassandra_storageType.md`
    - Customize Zendro -> `setup_customize.md`
- Authentication -> `oauth.md`
    - Roles -> `roles.md`
- Zendro API -> `api_root.md`
    - ACL -> `api_acl.md`
    - GraphQL -> `api_graphql.md`
    - SQL -> `api_sql.md`
    - Python -> `Zendro_requests_with_python.md`
- Distributed Data Models -> `ddm.md`
- Zendro CLI -> `zendro_cli.md`

In order to have the Table of contents in the page we are visiting, we have to add this code in the desired position:

```
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}
```

After you can see the changes in the [github page](https://zendro-dev.github.io/), please make sure all the menus are shown as desired according to the table of content we showed previusly.