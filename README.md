<div align="center">

<img src="https://raw.githubusercontent.com/MikaPST/unraid-community-apps/main/icon.png" alt="MikaPST Unraid Community Apps" width="160">

# MikaPST — Unraid Community Apps

**Docker application templates for Unraid**

[![Unraid](https://img.shields.io/badge/Unraid-Community%20Apps-blue?logo=unraid)](https://unraid.net/)
[![GitHub](https://img.shields.io/badge/GitHub-MikaPST-black?logo=github)](https://github.com/MikaPST/unraid-community-apps)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)

Self-hosted applications and integrations for the Unraid community.

</div>

---

# Table of Contents

- [Applications](#applications)
  - [Sablier](#sablier)
    - [Installing Sablier on Unraid](#installing-sablier-on-unraid)
    - [Docker network](#docker-network)
    - [Traefik integration](#traefik-integration)
      - [Enable the Sablier plugin](#1-enable-the-sablier-plugin-in-traefik)
      - [Create a Sablier middleware](#2-create-a-sablier-middleware)
    - [Activating Sablier on an application](#activating-sablier-on-an-application)
    - [Custom Sablier themes](#custom-sablier-themes)
    - [Sablier Theme Editor](#sablier-theme-editor)
    - [Sablier architecture](#sablier-architecture)
    - [Sablier useful links](#sablier-useful-links)
  - [DockDash](#dockdash)
    - [Installing DockDash on Unraid](#installing-dockdash-on-unraid)
    - [Security warning](#security-warning)
    - [Authentication](#authentication)
    - [DockDash configuration](#dockdash-configuration)
    - [DockDash update monitoring](#dockdash-update-monitoring)
    - [DockDash architecture](#dockdash-architecture)
    - [DockDash useful links](#dockdash-useful-links)
- [Repository structure](#repository-structure)
- [Support](#support)
- [License](#license)

---

# Applications

This repository provides Docker application templates designed for **Unraid Community Applications**.

The goal is to provide simple, documented and maintainable templates for useful self-hosted applications and integrations.

## Available applications

| Application | Description | Category | License |
|---|---|---|---|
| <img src="https://raw.githubusercontent.com/MikaPST/unraid-community-apps/main/icon-sablier.png" width="40" alt="Sablier"> [Sablier](https://sablierapp.dev/) | Docker-aware middleware for automatically starting and stopping containers based on incoming requests | Docker / FinOps | Apache-2.0 |
| <img src="https://raw.githubusercontent.com/MikaPST/unraid-community-apps/main/icon-dockdash.png" width="40" alt="DockDash"> [DockDash](https://github.com/dougmaitelli/DockDash) | Docker dashboard for monitoring containers, resources, image updates and GitHub release notes | Docker / Monitoring | AGPL-3.0 |

---

<div align="center">
<img src="https://raw.githubusercontent.com/MikaPST/unraid-community-apps/main/icon-sablier.png" width="100" alt="Sablier">
# Sablier
</div>

[Sablier](https://sablierapp.dev/) is a lightweight Docker-aware middleware that can automatically start and stop containers based on incoming requests.

It is designed to work with reverse proxies such as **Traefik**, helping reduce resource consumption by stopping applications that are not being used.

- Docker image: `sablierapp/sablier:1.15.0`
- Default HTTP/API port: `10000`
- Docker provider: enabled
- Reverse proxy integration: Traefik, Caddy and other compatible proxies
- License: Apache-2.0

## Installing Sablier on Unraid

The `templates/sablier.xml` file is intended for use with **Unraid Community Applications**.

The template configures:

- Sablier Docker image
- Docker socket access
- Sablier configuration file
- Optional custom themes directory
- HTTP port `10000`
- Traefik labels
- `start --provider.name=docker` startup command

### Docker network

The template uses the Docker network:

```text
traefik
```

by default, because this is a common configuration when Sablier is used with Traefik.

If you use another reverse proxy, you can change the Docker network to the network used by your reverse proxy.

Sablier itself is not limited to Traefik.

---

## Traefik integration

Sablier can be integrated with Traefik through the official Sablier Traefik plugin.

### 1. Enable the Sablier plugin in Traefik

Add the following configuration to your `traefik.yml`:

```yaml
# =========================
# PLUGINS
# =========================
experimental:
  plugins:
    sablier:
      moduleName: github.com/sablierapp/sablier-traefik-plugin
      version: v1.3.0
```

Restart Traefik after changing this configuration.

The plugin used in this example is **Sablier Traefik Plugin v1.3.0**.

Official Traefik plugin page:

https://plugins.traefik.io/plugins/69104ac3b7d4dd76110a1a09/sablier

### 2. Create a Sablier middleware

Once the plugin is enabled, configure a middleware in your Traefik dynamic configuration, for example in `middlewares.yml`.

Example:

```yaml
# =========================
# SABLIER APP
# =========================
sablier-bentopdf:
  plugin:
    sablier:
      sablierUrl: "http://sablier:10000"
      group: "bentopdf"
      sessionDuration: 5m
      ignoreUserAgent: "kuma"
      dynamic:
        displayName: "BentoPDF"
        showDetails: true
        theme: ghost_fr
        refreshFrequency: 10s
```

### Configuration explained

| Parameter | Description |
|---|---|
| `sablierUrl` | URL used by the Traefik plugin to communicate with Sablier |
| `group` | Sablier group associated with the application |
| `sessionDuration` | How long the application remains active after being requested |
| `ignoreUserAgent` | Optional user-agent to ignore |
| `displayName` | Name displayed by the dynamic Sablier page |
| `showDetails` | Displays additional information on the Sablier page |
| `theme` | Theme used by the Sablier dynamic page |
| `refreshFrequency` | Frequency used to refresh the Sablier page |

The `sablierUrl` assumes that the Sablier container is reachable as `sablier` on the same Docker network as Traefik.

---

## Activating Sablier on an application

Once the middleware has been created, the application that should be managed by Sablier needs the appropriate Traefik and Sablier labels.

For example, for BentoPDF:

```yaml
labels:
  - "traefik.http.routers.bentopdf.middlewares=sablier-bentopdf@file"
  - "sablier.enable=true"
  - "sablier.group=bentopdf"
  - "traefik.docker.allownonrunning=true"
```

The important elements are:

```text
traefik.http.routers.bentopdf.middlewares=sablier-bentopdf@file
```

This attaches the previously created Traefik middleware to the application's router.

```text
sablier.enable=true
```

This enables Sablier management for the container.

```text
sablier.group=bentopdf
```

This associates the container with the same Sablier group configured in the middleware.

```text
traefik.docker.allownonrunning=true
```

This allows Traefik to consider the container even when it is stopped, which is important for Sablier's start-on-request behavior.

Replace `bentopdf` with the name/group appropriate for your application.

---

## Custom Sablier themes

Sablier supports custom themes for its dynamic pages.

The Unraid template provides an optional directory:

```text
/mnt/user/appdata/sablier/themes
```

which is mounted inside the container as:

```text
/etc/sablier/themes
```

For example, custom themes can be stored locally as:

```text
/mnt/user/appdata/sablier/themes/ghost_fr.html
/mnt/user/appdata/sablier/themes/hacker-terminal_fr.html
/mnt/user/appdata/sablier/themes/shuffle_fr.html
```

They can then be referenced from the Traefik middleware configuration:

```yaml
dynamic:
  theme: ghost_fr
```

or:

```yaml
dynamic:
  theme: hacker-terminal_fr
```

This makes it possible to customize the Sablier page displayed while an application is starting.

### Sablier Theme Editor

Sablier provides an official online theme editor that makes it easy to create, customize and preview Sablier themes.

**Official Sablier Theme Editor:**

https://editor.sablierapp.dev/

Once your theme is created, you can export it and place the resulting `.html` file in the Sablier themes directory:

```text
/mnt/user/appdata/sablier/themes/
```

You can then reference the theme from your Traefik middleware configuration using the filename without the `.html` extension.

For example:

```text
ghost_fr.html
```

becomes:

```yaml
dynamic:
  theme: ghost_fr
```

---

## Sablier architecture

A typical Traefik + Sablier setup looks like this:

```text
                         Internet
                             │
                             ▼
                         Traefik
                             │
                 Sablier Traefik Plugin
                             │
                             ▼
                          Sablier
                     ┌───────┴───────┐
                     │ Docker API    │
                     ▼               │
              Docker containers     │
                     │               │
          ┌──────────┼──────────┐    │
          ▼          ▼          ▼    │
       BentoPDF   Shelfmark   Caesium │
```

Traefik receives the request, the Sablier middleware checks the application state, and Sablier can start the associated container when necessary.

---

## Sablier useful links

- **Official website:** https://sablierapp.dev/
- **Official documentation / tutorials:** https://sablierapp.dev/tutorials/
- **GitHub repository:** https://github.com/sablierapp/sablier
- **Traefik plugin:** https://plugins.traefik.io/plugins/69104ac3b7d4dd76110a1a09/sablier

---

<div align="center">
<img src="https://raw.githubusercontent.com/MikaPST/unraid-community-apps/main/icon-dockdash.png" width="100" alt="DockDash">
# DockDash
</div>

[DockDash](https://github.com/dougmaitelli/DockDash) is a self-hosted dashboard for visualizing Docker containers and network services.

It automatically discovers Docker services, monitors their health and resources, detects available container image updates and displays the corresponding GitHub release notes.

DockDash is particularly useful for keeping track of versioned Docker images without having to manually visit the GitHub repository of every application.

- Docker image: `ghcr.io/dougmaitelli/dockdash:latest`
- Default container port: `3001`
- Unraid host port: `4001`
- Docker network: `bridge`
- Docker provider: Docker socket
- Update monitoring: semantic-version aware
- GitHub release notes: supported
- Authentication: **not enforced by default**
- License: AGPL-3.0

---

## Installing DockDash on Unraid

The `templates/dockdash.xml` file is intended for use with **Unraid Community Applications**.

The template configures:

- DockDash Docker image
- Standard Docker `bridge` network
- Docker socket access
- Persistent application data
- HTTP port `4001 → 3001`
- Update monitoring
- Network discovery
- Health monitoring
- Resource monitoring

The default configuration stores DockDash application data in:

```text
/mnt/user/appdata/dockdash
```

and mounts it inside the container as:

```text
/app/data
```

The Docker socket is mounted as:

```text
/var/run/docker.sock
```

---

## ⚠️ Security warning

**DockDash does not enforce authentication by default.**

This is particularly important because DockDash requires access to the Docker socket and provides powerful Docker management features.

Depending on the configuration, DockDash can:

- start, stop and restart containers
- execute commands inside containers
- access container filesystems
- view Docker logs
- monitor resources
- manage Docker services

Access to the Docker socket can effectively provide **root-level control over the Docker host**.

Therefore:

> **Do not expose DockDash directly to the Internet or to an untrusted network without an authentication layer.**

For a typical self-hosted installation, protect DockDash using either its built-in OIDC authentication or an authenticated reverse proxy.

---

## Authentication

DockDash supports two main approaches for protecting the interface.

### Option 1 — Built-in OIDC

DockDash supports OpenID Connect authentication with providers such as:

- Authentik
- Authelia
- Keycloak
- Google
- Other compatible OIDC providers

OIDC can be configured using environment variables such as:

```text
OIDC_ISSUER
OIDC_CLIENT_ID
OIDC_CLIENT_SECRET
SESSION_SECRET
```

Refer to the official DockDash documentation for the complete OIDC configuration.

### Option 2 — Reverse proxy authentication

DockDash can also be placed behind a reverse proxy and protected by an external authentication layer.

Examples include:

- Traefik + Authentik
- Traefik + Authelia
- Caddy + authentication middleware
- Nginx + authentication
- oauth2-proxy
- TinyAuth
- Other trusted authentication solutions

This approach can be particularly convenient in an existing self-hosted infrastructure where authentication is already centralized.

---

## DockDash configuration

The Unraid template exposes the most useful DockDash configuration parameters.

### Basic configuration

| Parameter | Default | Description |
|---|---:|---|
| HTTP Port | `4001 → 3001` | Port used to access the DockDash web interface |
| DockDash Data | `/mnt/user/appdata/dockdash` | Persistent application data |
| Docker Socket | `/var/run/docker.sock` | Docker API access |

### Advanced configuration

| Variable | Default | Description |
|---|---:|---|
| `LOG_LEVEL` | `info` | DockDash logging level |
| `NETWORK_CIDRS` | `192.168.0.1/24` | Network ranges used for network discovery |
| `HEALTH_CHECK_INTERVAL` | `30000` | Health check interval in milliseconds |
| `RESOURCE_MONITOR_INTERVAL` | `5000` | Resource monitoring interval in milliseconds |
| `UPDATE_CHECK_INTERVAL` | `3600000` | Docker image update check interval in milliseconds |

The `NETWORK_CIDRS` value should be adapted to the local network if necessary.

For example:

```text
192.168.0.0/24
```

or another CIDR corresponding to your local network.

---

## DockDash update monitoring

One of DockDash's main features is its **semantic-version-aware Docker image update monitoring**.

Instead of simply reporting that a Docker image has changed, DockDash can compare the version actually running with newer compatible versions published by the registry.

For example:

```text
Running:
1.25.0

Available:
1.26.0
```

DockDash can report:

```text
1.25.0 → 1.26.0
```

It can then resolve the source repository and retrieve the corresponding GitHub release notes.

This makes it possible to see:

```text
Application
Current version → Available version
          │
          ▼
      GitHub Release
          │
          ▼
       Changelog
```

This is particularly useful when managing a large number of self-hosted applications.

Instead of manually checking each GitHub repository, DockDash provides the update information and associated release notes directly from the dashboard.

DockDash also supports digest-based detection for floating tags such as:

```text
latest
stable
dev
```

---

## DockDash architecture

A basic Unraid installation looks like this:

```text
                         Unraid
                            │
                            ▼
                    ┌───────────────┐
                    │    DockDash   │
                    │    :3001      │
                    └───────┬───────┘
                            │
                     Docker Socket
                            │
                            ▼
                     Docker Engine
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
    Sablier             Traefik            Vaultwarden
        │                   │                   │
        ▼                   ▼                   ▼
      Other              Other               Other
    containers          containers          containers
```

DockDash reads Docker information through:

```text
/var/run/docker.sock
```

and provides a centralized view of the Docker environment.

It can monitor:

- container health
- CPU and memory usage
- network and disk I/O
- Docker image versions
- available updates
- GitHub changelogs
- Docker logs
- container filesystem
- container terminal
- service topology

---

## Docker socket hardening

For advanced deployments, the Docker socket can be exposed through a restricted Docker socket proxy rather than mounting the host socket directly.

A solution such as:

```text
tecnativa/docker-socket-proxy
```

can be used to limit the Docker API operations available to DockDash.

This is an advanced configuration and is not required for the standard Unraid template.

---

## DockDash useful links

- **GitHub repository:** https://github.com/dougmaitelli/DockDash
- **Docker image:** https://github.com/dougmaitelli/DockDash/pkgs/container/dockdash
- **Security policy:** https://github.com/dougmaitelli/DockDash/blob/main/SECURITY.md

---

# Repository structure

```text
.
├── ca_profile.xml
├── icon.png
├── icon-sablier.png
├── icon-dockdash.png
├── README.md
├── LICENSE
└── templates/
    ├── sablier.xml
    └── dockdash.xml
```

Each Docker application has its own XML template under:

```text
templates/
```

For example:

```text
templates/sablier.xml
templates/dockdash.xml
```

Application icons are stored at the repository root and referenced directly by their corresponding templates.

---

# Support

For issues concerning an **application itself**, please refer to the official project repository.

### Sablier

https://github.com/sablierapp/sablier

### DockDash

https://github.com/dougmaitelli/DockDash

For issues specifically related to the **Unraid templates maintained in this repository**, please open an issue in this GitHub repository.

---

# License

This repository contains Unraid application templates maintained by MikaPST.

Each application remains subject to its own upstream license:

- **Sablier:** Apache License 2.0
- **DockDash:** GNU Affero General Public License v3.0

The repository itself is distributed under the license specified in `LICENSE`.
