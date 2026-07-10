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
* On Windows, we recommend using Zendro through the Windows Subsystem for Linux (WSL).
* On Mac, we recommend using Zendro without docker.

## Install the Zendro CLI

```bash
git clone https://github.com/Zendro-dev/zendro.git
cd zendro
npm install
sudo npm link
```

In Windows Subsystem for Linux, `sudo npm` may not work; try `sudo -E env "PATH=$PATH" npm` instead. The same applies to the docker-related command `zendro dockerize`, since docker requires elevated permissions.

Once linked, the `zendro` command is available anywhere on your system. See the [CLI reference]({% link cli-reference.md %}) for the full list of commands, or continue to the [Quickstart]({% link quickstart.md %}) or [Getting started]({% link getting-started/index.md %}) guide to create your first project.

## Updating Zendro

To update the Zendro CLI itself, go to your `zendro` folder and run:

```bash
git pull
rm -r package-lock.json node_modules
npm install
```

## Uninstallation

### Remove a project

```bash
sudo rm -r "path/to/<name>"
docker rmi -f $(docker images -a -q "<name>*")
docker volume rm $(docker volume ls -q | grep "^<name>")
```

### Uninstall the Zendro CLI

```bash
sudo npm unlink -g zendro
sudo rm -r "path/to/zendro"
```
