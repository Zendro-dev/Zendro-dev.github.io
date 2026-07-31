---
layout: default
title: Installation and the Zendro CLI
parent: Getting started
nav_order: 1
permalink: /getting-started/installation
---

# Installation and the Zendro CLI
{: .no_toc }

Every Zendro project, whether you set it up with the [Quickstart]({% link quickstart.md %}) shortcut or the full [Getting started]({% link getting-started/index.md %}) walkthrough, starts from the same CLI installation. This page covers that shared groundwork, plus updating and uninstalling Zendro, so the other guides can just link back here instead of repeating it.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Requirements

* [NodeJS](https://nodejs.org/en/) version 18+ is required.
* [docker](https://docs.docker.com/get-docker/) and the [docker compose plugin](https://docs.docker.com/compose/install/#install-compose) (if not already included in your docker installation) are recommended for setting up Zendro.

We strongly recommend following [this guide](https://docs.docker.com/engine/install/linux-postinstall/) to use docker without `sudo`.

## Recommendations

* We strongly recommend using Zendro on Linux, with or without docker.
* On Windows, we recommend using Zendro through the Windows Subsystem for Linux (WSL). However, inside WSL consider operating on native paths, not the mounted windows. There is high IO latency otherwise.
* On Mac, we recommend using Zendro without docker.

## Install the Zendro CLI

```bash
npm install -g --allow-git=all git+https://github.com/Zendro-dev/zendro.git
```

`--allow-git=all` is needed on npm 12+, which disables installing dependencies from git by default; it's a harmless no-op on older npm.

The `zendro` command is now available anywhere on your system. See the [CLI reference]({% link cli-reference.md %}) for the full list of commands, or continue to the [Quickstart]({% link quickstart.md %}) or [Getting started]({% link getting-started/index.md %}) guide to create your first project.

If you'd rather work from a local, editable checkout (e.g. to contribute to the CLI itself), clone and link it instead:

```bash
git clone https://github.com/Zendro-dev/zendro.git
cd zendro
npm install
npm link
```

## Updating Zendro

If you installed with `npm install -g git+...`, just run that same command again — a global install has no lockfile pinning it to the commit you started with, so it always re-resolves and picks up the latest changes:

```bash
npm install -g --allow-git=all git+https://github.com/Zendro-dev/zendro.git
```

If you used the local `git clone` + `npm link` method instead, pull the latest changes and refresh the CLI's own git-based dependencies. Unlike the CLI's own code, these three are pinned by your local `package-lock.json`, so a plain `npm install` alone won't move them past the commit they were first resolved at — `npm update` re-resolves them properly:

```bash
git pull
npm update graphql-server-model-codegen zendro-bulk-create ZendroStarterPack
```

## Uninstallation

### Remove a project

```bash
zendro rm <name>
```

This deletes the project's directory along with its docker containers, images and volumes. Add `-f`/`--force` to skip the confirmation prompt.

### Uninstall the Zendro CLI

If installed via `npm install -g`:

```bash
npm uninstall -g zendro
```

If installed via the local `git clone` + `npm link` method:

```bash
npm unlink -g zendro
rm -r "path/to/zendro"
```
