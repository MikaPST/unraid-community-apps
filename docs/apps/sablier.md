<div align="center">

<img src="https://raw.githubusercontent.com/sablierapp/artwork/refs/heads/main/horizontal/sablier-horizontal-color.png" alt="Sablier" width="360">

# Sablier on Unraid

**Start containers on demand and stop them automatically when they are no longer needed**

[![Unraid Community Apps](https://img.shields.io/badge/Unraid-Community%20Apps-0E83CD?logo=unraid)](https://unraid.net/)
[![GitHub Stars](https://img.shields.io/github/stars/sablierapp/sablier?style=flat&logo=github)](https://github.com/sablierapp/sablier/stargazers)
[![Latest Release](https://img.shields.io/github/v/release/sablierapp/sablier?display_name=tag)](https://github.com/sablierapp/sablier/releases)
[![License](https://img.shields.io/github/license/sablierapp/sablier?color=green)](https://github.com/sablierapp/sablier/blob/main/LICENSE)
[![Unraid templates](https://img.shields.io/badge/Unraid%20templates-MikaPST-181717?logo=github)](https://github.com/MikaPST/unraid-community-apps)

[**Official website**](https://sablierapp.dev/) ·
[**Documentation**](https://sablierapp.dev/) ·
[**GitHub repository**](https://github.com/sablierapp/sablier) ·
[**Traefik plugin**](https://plugins.traefik.io/plugins/69104ac3b7d4dd76110a1a09/sablier)

</div>

---

## Table of contents

- [Overview](#overview)
- [How Sablier works](#how-sablier-works)
- [Install on Unraid](#install-on-unraid)
- [Docker network](#docker-network)
- [Integrate Sablier with Traefik](#integrate-sablier-with-traefik)
  - [1. Enable the Sablier plugin](#1-enable-the-sablier-plugin)
  - [2. Create a Sablier middleware](#2-create-a-sablier-middleware)
  - [3. Attach the middleware to an application](#3-attach-the-middleware-to-an-application)
- [Configuration reference](#configuration-reference)
- [Custom Sablier themes](#custom-sablier-themes)
  - [Install a custom theme](#install-a-custom-theme)
  - [Use the Sablier Theme Editor](#use-the-sablier-theme-editor)
- [Architecture](#architecture)
- [Security and reliability notes](#security-and-reliability-notes)
- [Troubleshooting](#troubleshooting)
- [Useful links](#useful-links)
- [Credits and license](#credits-and-license)

---

## Overview

[Sablier](https://sablierapp.dev/) is a Docker-aware middleware that starts applications on demand and stops them automatically after a period of inactivity.

It is particularly useful in self-hosted environments where some services are used only occasionally. When integrated with a reverse proxy such as Traefik, Sablier can wake a stopped application when a request arrives and display a waiting page while the application starts.

This repository provides an Unraid Community Applications template for Sablier. The Sablier software itself is developed and maintained by the upstream project.

### At a glance

| Setting | Value in this repository's template |
|---|---|
| Docker image | `sablierapp/sablier:1.15.0` |
| HTTP/API port | `10000` |
| Default Docker network | `traefik` |
| Docker provider | Enabled |
| Docker socket | Mounted for Docker lifecycle management |
| Reverse proxy integration | Traefik and other supported integrations |
| Optional configuration | Configuration file and custom theme directory |
| Upstream license | Apache-2.0 |

> **Version note:** the image tag above reflects the template configuration documented in this repository. Check `templates/sablier.xml` before assuming the template uses the latest upstream release.

---

## How Sablier works

A typical flow looks like this:

1. A user opens a URL for an application managed by Sablier.
2. Traefik applies the Sablier middleware.
3. Sablier checks whether the relevant application is running.
4. If necessary, Sablier starts the container and serves a waiting page while it becomes ready.
5. The request reaches the application.
6. After the session expires and the application is inactive, Sablier can stop it according to the configured strategy.

Sablier can manage groups of applications together, so multiple containers can be started or stopped as part of the same logical service.

### Typical use cases

- Development, test or demo environments used only at certain times
- Heavy applications that do not need to run continuously
- Self-hosted tools used occasionally
- Reducing idle resource use on a home server

Sablier is not a replacement for an application's own health checks or availability monitoring. Configure health checks for applications that need to be scaled down to zero so Sablier can distinguish a running container from one that is ready to accept requests.

---

## Install on Unraid

1. Open **Apps** in the Unraid web interface.
2. Search for **Sablier**.
3. Select the template provided by **MikaPST — Unraid Community Apps** and click **Install**.
4. Review the Docker network, Docker socket mapping, port and configuration paths.
5. Click **Apply** and wait for the container to start.
6. Confirm the container logs show that Sablier has started successfully.

The Sablier server's HTTP/API port is:

```text
10000/tcp
```

Sablier is normally used internally by a reverse proxy, so you generally do not need to expose its API port publicly. Keep it reachable from Traefik on a shared Docker network, and avoid publishing the port to the Internet.

### What the template configures

The template documented by this repository configures:

- The Sablier Docker image
- Docker socket access for container lifecycle management
- A configuration file
- An optional custom themes directory
- HTTP/API port `10000`
- A startup command using the Docker provider
- The `traefik` Docker network by default

Review the actual XML template if you need to confirm the exact volume paths or image tag used by your current revision.

---

## Docker network

The template uses the following Docker network by default:

```text
traefik
```

This is suitable for a common installation where Traefik and Sablier share a Docker network.

**Important:** the `traefik` network must already exist before the container is started. If your reverse proxy uses a different network, select that network in the Unraid template instead.

Sablier is not intrinsically limited to Traefik, but the middleware and examples in this guide focus on Traefik.

### Check the network

In Unraid, verify that:

- The Traefik container is connected to the selected network.
- The Sablier container is connected to the same network.
- The Sablier service can be resolved from Traefik by its Docker DNS name, such as `sablier`.
- The middleware's `sablierUrl` points to the correct service name and port.

The examples below use:

```text
http://sablier:10000
```

Replace `sablier` with the container's actual DNS name if yours is different.

---

## Integrate Sablier with Traefik

This section follows the file-provider approach: enable the Sablier Traefik plugin in Traefik's static configuration, define a middleware in the dynamic configuration, then attach that middleware to the application router.

### 1. Enable the Sablier plugin

Add the plugin to your Traefik static configuration, for example `traefik.yml`:

```yaml
# =========================
# PLUGINS
# =========================
experimental:
  plugins:
    sablier:
      moduleName: github.com/sablierapp/sablier-traefik-plugin
      version: v1.3.1
```

The version shown is the version listed in the Traefik Plugin Catalog when this README was prepared. Check the [official plugin page](https://plugins.traefik.io/plugins/69104ac3b7d4dd76110a1a09/sablier) before upgrading or pinning another version.

Restart Traefik after modifying the static configuration. Traefik loads plugins during startup.

### 2. Create a Sablier middleware

Once the plugin is enabled, define a middleware in Traefik's dynamic configuration, for example in `middlewares.yml`:

```yaml
# =========================
# SABLIER MIDDLEWARE
# =========================
http:
  middlewares:
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

This is an example configuration for a Sablier group named `bentopdf`. Rename the middleware and group to match your application.

#### Configuration explained

| Parameter | Description |
|---|---|
| `sablierUrl` | URL Traefik uses to communicate with Sablier |
| `group` | Sablier group associated with the application |
| `sessionDuration` | Duration of a session before it expires |
| `ignoreUserAgent` | Optional user-agent that should be ignored by the middleware |
| `displayName` | Name shown on the dynamic waiting page |
| `showDetails` | Displays additional information on the waiting page |
| `theme` | Theme used by the waiting page |
| `refreshFrequency` | How often the waiting page refreshes its status |

The `sablierUrl` assumes that Sablier is reachable as `sablier` on the same Docker network as Traefik.

### 3. Attach the middleware to an application

Add the relevant labels to the Docker container you want Sablier to manage. For example, for BentoPDF:

```yaml
labels:
  - "traefik.http.routers.bentopdf.middlewares=sablier-bentopdf@file"
  - "sablier.enable=true"
  - "sablier.group=bentopdf"
  - "traefik.docker.allownonrunning=true"
```

#### What these labels do

| Label | Purpose |
|---|---|
| `traefik.http.routers.bentopdf.middlewares=sablier-bentopdf@file` | Attaches the middleware defined in the dynamic file configuration |
| `sablier.enable=true` | Enables Sablier management for the container |
| `sablier.group=bentopdf` | Associates the container with the configured Sablier group |
| `traefik.docker.allownonrunning=true` | Allows Traefik to consider a stopped container, which is important for start-on-request behaviour |

Replace `bentopdf` with the router, middleware and group name used by your application. The names must match across the Traefik configuration and Docker labels.

> **Health checks matter:** Sablier needs a reliable way to determine when an application is ready. Configure a suitable Docker health check for workloads that are started from a stopped state.

### Integration checklist

- [ ] Sablier is running and reachable by Traefik.
- [ ] The plugin is enabled in Traefik's static configuration.
- [ ] The dynamic middleware is loaded without errors.
- [ ] The application's labels enable Sablier and use the correct group.
- [ ] The router references the middleware with the correct provider suffix, such as `@file`.
- [ ] The application has a health check where readiness detection is required.
- [ ] You have tested both the stopped-to-started flow and the idle-stop behaviour.

---

## Configuration reference

Sablier can be configured through its configuration file, environment variables and command-line arguments. The supported options depend on the upstream version and provider.

The template documented in this repository uses the Docker provider and starts Sablier with:

```text
start --provider.name=docker
```

The key deployment settings are:

| Setting | Purpose |
|---|---|
| Docker provider | Allows Sablier to discover and manage Docker containers |
| Docker socket | Provides access to Docker lifecycle operations |
| HTTP/API port `10000` | Makes Sablier's API available to Traefik and other permitted clients |
| Configuration file | Holds optional Sablier settings |
| Themes directory | Stores custom waiting-page themes |

Consult the [official Sablier documentation](https://sablierapp.dev/) for version-specific configuration options. Do not copy options from a different Sablier release without verifying they are supported by your deployed image.

---

## Custom Sablier themes

Sablier supports custom themes for its dynamic waiting pages. The Unraid template provides an optional directory for custom theme files.

The host directory used in this repository's documentation is:

```text
/mnt/user/appdata/sablier/themes
```

It is mounted inside the container as:

```text
/etc/sablier/themes
```

Example files:

```text
/mnt/user/appdata/sablier/themes/ghost_fr.html
/mnt/user/appdata/sablier/themes/hacker-terminal_fr.html
/mnt/user/appdata/sablier/themes/shuffle_fr.html
```

The names above are examples of custom theme files. Make sure each file is a valid Sablier theme compatible with your installed version.

### Install a custom theme

1. Create the themes directory on Unraid if it does not already exist.
2. Place your theme `.html` files in that directory.
3. Confirm the directory is mounted in the Sablier container at `/etc/sablier/themes`.
4. Reference the theme name in the Traefik middleware, without the `.html` extension.

For example, a file named:

```text
ghost_fr.html
```

is referenced as:

```yaml
dynamic:
  theme: ghost_fr
```

Sablier's current releases have made changes to custom-theme path handling, so check the [upstream changelog](https://github.com/sablierapp/sablier/blob/main/CHANGELOG.md) if a theme stored in a subdirectory does not load.

### Use the Sablier Theme Editor

Sablier provides an online theme editor for creating, customizing and previewing themes.

**Official editor:** https://editor.sablierapp.dev/

Suggested workflow:

1. Open the theme editor.
2. Customize and preview the waiting page.
3. Export the theme as an HTML file.
4. Copy it to `/mnt/user/appdata/sablier/themes/` on Unraid.
5. Set the theme name in your Traefik middleware, without the `.html` suffix.
6. Test a real start-on-request flow through Traefik.

---

## Architecture

A typical Traefik + Sablier deployment looks like this:

```text
                          Internet
                              |
                              v
                           Traefik
                              |
                     Sablier middleware
                              |
                              v
                           Sablier
                              |
                         Docker API
                              |
                              v
                        Docker Engine
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
          BentoPDF         Shelfmark         Caesium
```

When a request reaches Traefik, the Sablier middleware coordinates with Sablier. If the associated workload is stopped, Sablier can start it and serve a waiting page while it becomes ready. When the configured session expires and inactivity rules allow it, Sablier can stop the workload again.

The applications and example names in the diagram are illustrative. Your actual containers depend on your own setup.

---

## Security and reliability notes

- **Protect access to Traefik and your applications.** Sablier is a lifecycle-management component, not an authentication service.
- **Restrict access to the Sablier API.** It generally needs to be reachable by Traefik and trusted administrative systems, not by the public Internet.
- **Protect the Docker socket.** Sablier needs Docker access to manage containers. Treat the container and its socket permissions as privileged infrastructure.
- **Use application health checks.** A container being in a running state does not necessarily mean its application is ready.
- **Test group names and labels carefully.** A mismatch between the middleware group and application labels can prevent the expected workload from starting.
- **Validate custom themes against your Sablier version.** Theme loading behaviour can change between releases.
- **Keep version compatibility in mind.** Confirm that the Traefik plugin and Sablier server versions are compatible with your deployment.

---

## Troubleshooting

### Traefik cannot contact Sablier

- Confirm Sablier is running.
- Verify that both containers share the expected Docker network.
- Check that `sablierUrl` uses the correct container DNS name and port `10000`.
- Review Sablier and Traefik logs for connection errors.

### Requests do not start a stopped container

- Confirm the container has `sablier.enable=true`.
- Confirm its `sablier.group` matches the `group` setting in the middleware.
- Ensure the Traefik router references the intended middleware.
- Check whether Traefik is allowed to consider stopped containers; for Docker-provider configurations, this may require `traefik.docker.allownonrunning=true`.
- Review Sablier logs for provider or lifecycle errors.

### The waiting page appears, but the application never becomes ready

- Check the application container logs.
- Make sure the container has an appropriate health check.
- Verify that the application listens on the expected port and can become ready.
- Review the session duration and any readiness-related settings supported by your Sablier version.

### A custom theme does not load

- Confirm the host directory exists and is mounted to `/etc/sablier/themes`.
- Verify the file ends in `.html` and is a valid theme.
- Use the theme name without the `.html` extension.
- Check the Sablier logs and consult the upstream changelog for any path-handling changes in your version.

---

## Useful links

- **Official website:** https://sablierapp.dev/
- **Official documentation and tutorials:** https://sablierapp.dev/tutorials/
- **GitHub repository:** https://github.com/sablierapp/sablier
- **Release notes:** https://github.com/sablierapp/sablier/releases
- **Traefik plugin:** https://plugins.traefik.io/plugins/69104ac3b7d4dd76110a1a09/sablier
- **Theme Editor:** https://editor.sablierapp.dev/
- **Unraid template repository:** https://github.com/MikaPST/unraid-community-apps
- **Report an upstream issue:** https://github.com/sablierapp/sablier/issues
- **Report an Unraid template issue:** https://github.com/MikaPST/unraid-community-apps/issues

## Credits and license

Sablier is an open-source project developed and maintained by [Sablier](https://github.com/sablierapp/sablier). The upstream project is licensed under the **Apache License 2.0**. See the [upstream license file](https://github.com/sablierapp/sablier/blob/main/LICENSE).

The Unraid template in this repository is maintained by **MikaPST** as an unofficial community contribution. It is not an official Unraid or Sablier project.
