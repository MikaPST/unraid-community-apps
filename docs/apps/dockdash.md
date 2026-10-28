<div align="center">

<img src="https://raw.githubusercontent.com/MikaPST/unraid-community-apps/main/icons/icon-dockdash.png" alt="DockDash icon" width="128">

# DockDash on Unraid

**A self-hosted dashboard for monitoring Docker containers, services, resources and image updates**

[![Unraid Community Apps](https://img.shields.io/badge/Unraid-Community%20Apps-0E83CD?logo=unraid)](https://unraid.net/)
[![GitHub Stars](https://img.shields.io/github/stars/dougmaitelli/DockDash?style=flat&logo=github)](https://github.com/dougmaitelli/DockDash/stargazers)
[![Latest Release](https://img.shields.io/github/v/release/dougmaitelli/DockDash?display_name=tag)](https://github.com/dougmaitelli/DockDash/releases)
[![License](https://img.shields.io/github/license/dougmaitelli/DockDash?color=green)](https://github.com/dougmaitelli/DockDash/blob/master/LICENSE)
[![Unraid templates](https://img.shields.io/badge/Unraid%20templates-MikaPST-181717?logo=github)](https://github.com/MikaPST/unraid-community-apps)

[**Upstream GitHub project**](https://github.com/dougmaitelli/DockDash) ·
[**Configuration guide**](https://github.com/dougmaitelli/DockDash/blob/master/docs/CONFIGURATION.md) ·
[**Security policy**](https://github.com/dougmaitelli/DockDash/blob/master/SECURITY.md)

</div>

---

## Table of contents

- [Overview](#overview)
- [Features](#features)
- [Install on Unraid](#install-on-unraid)
- [Initial configuration](#initial-configuration)
- [Configuration reference](#configuration-reference)
- [Authentication](#authentication)
  - [Option 1 — Built-in OIDC](#option-1--built-in-oidc)
  - [Option 2 — Authenticated reverse proxy](#option-2--authenticated-reverse-proxy)
- [Docker socket and security](#docker-socket-and-security)
  - [Using a restricted Docker socket proxy](#using-a-restricted-docker-socket-proxy)
- [Image update monitoring](#image-update-monitoring)
- [Architecture](#architecture)
- [Troubleshooting](#troubleshooting)
- [Useful links](#useful-links)
- [Credits and license](#credits-and-license)

---

## Overview

[DockDash](https://github.com/dougmaitelli/DockDash) is a self-hosted dashboard for visualizing Docker containers and network services. It discovers services, tracks their health, monitors resource usage and Docker image updates, and lets you map connections between services on an interactive topology canvas.

This repository provides an Unraid Community Applications template for DockDash. The application itself is developed and maintained by the upstream project.

> **Important security note:** DockDash can manage containers, open interactive terminals and browse or edit files inside containers. Anyone who can access an unprotected instance may gain powerful control over your Docker environment. Configure authentication before allowing access from an untrusted network.

### At a glance

| Setting | Value |
|---|---|
| Docker image | `ghcr.io/dougmaitelli/dockdash:latest` |
| Container port | `3001/tcp` |
| Default Unraid host port | `4001` |
| Default Docker network | `bridge` |
| Persistent data | `/mnt/user/appdata/dockdash` → `/app/data` |
| Docker socket | `/var/run/docker.sock` |
| Authentication | Not enforced by default; configure OIDC or an authenticated reverse proxy |
| Upstream license | AGPL-3.0 |

---

## Features

DockDash brings several Docker management and monitoring functions into one web interface.

- **Docker discovery:** discover containers and services from Docker.
- **Service health:** check service availability and review health history.
- **Resource monitoring:** view CPU, memory, network and disk I/O metrics.
- **Image update monitoring:** check registries for newer container images.
- **Release notes:** display related GitHub release notes when repository metadata can be resolved.
- **Container controls:** start, stop and restart containers.
- **Logs:** view container logs from the web interface.
- **File explorer:** browse container filesystems and view or edit text files.
- **Interactive terminal:** open a shell in a container.
- **Network discovery:** scan configured network ranges for services that are not managed by Docker.
- **Topology canvas:** arrange service nodes and draw connections between them.
- **OIDC authentication:** optionally integrate with an OpenID Connect identity provider.
- **Notifications:** integrate with an Apprise REST API service for supported alerts.

Feature availability can depend on the version deployed and the configured permissions or integrations. Consult the [upstream project](https://github.com/dougmaitelli/DockDash) for current details.

---

## Install on Unraid

1. Open **Apps** in the Unraid web interface.
2. Search for **DockDash**.
3. Select the template provided by **MikaPST — Unraid Community Apps** and click **Install**.
4. Review the settings. The default host port in this template is `4001`, mapped to container port `3001`.
5. Confirm the appdata path and Docker socket mapping.
6. Click **Apply** and wait for the container to start.

Open the web interface at:

```text
http://YOUR_UNRAID_IP:4001/
```

Replace `YOUR_UNRAID_IP` with the IP address of your Unraid server. If you chose a different host port, use that port instead.

### What the template configures

- The DockDash Docker image
- The default Docker `bridge` network
- The web interface port mapping `4001 → 3001`
- Persistent application data at `/mnt/user/appdata/dockdash`, mounted as `/app/data`
- Access to `/var/run/docker.sock` for Docker discovery and management
- Network discovery and monitoring settings

The template mounts the Docker socket to provide DockDash with Docker API access. Review the [Docker socket and security](#docker-socket-and-security) section before exposing the interface to other users.

---

## Initial configuration

### 1. Confirm the network range

DockDash can scan a configured CIDR range to discover services on your network. The Unraid template has historically used `192.168.0.1/24` as its default `NETWORK_CIDRS` value. Review this setting and change it to match your actual LAN.

For example, a common `/24` network range is:

```text
192.168.0.0/24
```

For a different subnet, use the CIDR that matches your network. Do not copy the example blindly.

Network scanning can generate traffic across the configured range. Only scan networks you administer or are authorised to assess.

### 2. Verify persistence

The template stores DockDash data in:

```text
/mnt/user/appdata/dockdash
```

mounted inside the container at:

```text
/app/data
```

This directory is used for persistent application data such as the service inventory, dashboard layout and history. Keep the mapping in place if you want this data to survive container replacement or updates.

### 3. Secure the web interface before sharing it

DockDash does not enforce authentication by default. Before allowing access beyond a trusted personal environment, configure OIDC or place the app behind an authenticated reverse proxy. See [Authentication](#authentication).

---

## Configuration reference

The Unraid template exposes the main settings below. Advanced environment variables may need to be added to the template if they are not present in your current version.

### Basic settings

| Setting | Default | Description |
|---|---|---|
| HTTP Port | `4001 → 3001` | Host port and container port used for the DockDash web interface |
| DockDash Data | `/mnt/user/appdata/dockdash` | Persistent directory mounted as `/app/data` |
| Docker Socket | `/var/run/docker.sock` | Docker API access for discovery and container-management features |

### Advanced monitoring settings

| Variable | Default in the template | Description |
|---|---:|---|
| `LOG_LEVEL` | `info` | Application logging level |
| `NETWORK_CIDRS` | `192.168.0.1/24` | Network range(s) used for service discovery; adapt to your network |
| `HEALTH_CHECK_INTERVAL` | `30000` | Health-check interval, in milliseconds |
| `RESOURCE_MONITOR_INTERVAL` | `5000` | Resource-monitoring interval, in milliseconds |
| `UPDATE_CHECK_INTERVAL` | `3600000` | Image update-check interval, in milliseconds |

Intervals are expressed in milliseconds. For example, `60000` corresponds to one minute.

### OIDC environment variables

DockDash supports OpenID Connect (OIDC) for single sign-on. These are the key settings used for a typical OIDC deployment:

| Variable | Purpose |
|---|---|
| `OIDC_ISSUER` | Identity provider issuer or discovery URL |
| `OIDC_CLIENT_ID` | Client ID registered with the identity provider |
| `OIDC_CLIENT_SECRET` | Client secret registered with the identity provider |
| `SESSION_SECRET` | Secret used to sign session cookies; set a stable, random value for production |
| `OIDC_REDIRECT_URI` | Optional explicit public callback URL, useful with some reverse-proxy configurations |
| `OIDC_SCOPES` | Optional requested scopes; commonly `openid profile email` |

OIDC is enabled when the required issuer and client credentials are configured. Exact options can vary with the upstream version; refer to the [official configuration guide](https://github.com/dougmaitelli/DockDash/blob/master/docs/CONFIGURATION.md).

Treat `OIDC_CLIENT_SECRET` and `SESSION_SECRET` as secrets. Do not publish real values in XML templates, screenshots, logs or source control.

---

## Authentication

**Do not expose DockDash directly to an untrusted network without authentication.** The interface can provide access to powerful Docker operations.

There are two common ways to protect it.

### Option 1 — Built-in OIDC

DockDash supports OpenID Connect providers such as:

- Authentik
- Authelia
- Keycloak
- Google and other compatible OIDC providers

A typical configuration uses the following environment variables:

```dotenv
OIDC_ISSUER=https://auth.example.com/realms/homelab
OIDC_CLIENT_ID=dockdash
OIDC_CLIENT_SECRET=replace-with-your-client-secret
OIDC_REDIRECT_URI=https://dockdash.example.com/auth/callback
SESSION_SECRET=replace-with-a-long-random-secret
OIDC_SCOPES=openid profile email
```

These values are examples only. Replace them with the values from your identity provider. Do not use the example secrets in a real deployment.

In Unraid, add or edit the environment variables in the DockDash container configuration. The exact redirect URI and issuer URL depend on your identity provider and public URL.

Follow the [upstream OIDC documentation](https://github.com/dougmaitelli/DockDash/blob/master/docs/CONFIGURATION.md) for the complete setup.

### Option 2 — Authenticated reverse proxy

You can protect DockDash with a reverse proxy and an authentication layer, for example:

- Traefik with Authentik or Authelia
- Nginx Proxy Manager with an appropriate access-control solution
- Caddy with an authentication integration
- A VPN or private access gateway

A reverse proxy alone does not automatically enable authentication. Make sure an authentication policy is attached to the router or proxy host.

If you use a reverse proxy, avoid leaving another unrestricted route to the DockDash host port. Restrict direct access using your network configuration or firewall where practical.

---

## Docker socket and security

DockDash requires access to Docker to discover containers and provide its management features. The template mounts:

```text
/var/run/docker.sock
```

The Docker socket is highly privileged. Access to it can allow operations that are effectively equivalent to administrator or root control over the Docker host, including starting or stopping containers, opening container terminals, and reading or modifying container files.

This means that access to DockDash should be treated as privileged access to your Unraid server.

### Minimum recommendations

- **Require authentication** before users can access DockDash from an untrusted network.
- **Do not expose DockDash directly to the public Internet** without strong access controls.
- Restrict direct access to the host port when the application is published through a reverse proxy.
- Keep DockDash and its container image updated.
- Store OIDC secrets and other credentials securely.
- Disable features you do not need, where supported by your DockDash version.
- Consider a restricted Docker socket proxy for a more hardened setup.

Mounting the socket with a read-only bind mount does not by itself make the Docker API read-only or remove its privileged capabilities.

### Using a restricted Docker socket proxy

For a more hardened deployment, you can put a Docker socket proxy between DockDash and the host Docker daemon. This lets you limit which Docker API operations are available.

The following is an **illustrative Docker Compose example**, not a direct replacement for the Unraid template. Adapt it to your network and the features you intend to use:

```yaml
services:
  docker-proxy:
    image: tecnativa/docker-socket-proxy
    container_name: docker-proxy
    restart: unless-stopped
    environment:
      CONTAINERS: 1
      IMAGES: 1
      NETWORKS: 1
      INFO: 1
      # Required for container controls, terminal and file explorer:
      POST: 1
      EXEC: 1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - dockdash-internal

  dockdash:
    image: ghcr.io/dougmaitelli/dockdash:latest
    container_name: dockdash
    restart: unless-stopped
    ports:
      - "4001:3001"
    environment:
      DOCKER_HOSTS: tcp://docker-proxy:2375
    volumes:
      - /mnt/user/appdata/dockdash:/app/data
    depends_on:
      - docker-proxy
    networks:
      - dockdash-internal

networks:
  dockdash-internal:
    driver: bridge
```

Only services on the proxy's Docker network should be able to reach the proxy endpoint. Do not publish the proxy's port to the host or Internet.

**Important:** the proxy permissions above allow powerful operations. `POST` and `EXEC` are needed for certain container-management, terminal and file-explorer features. If you disable those features in DockDash, consider reducing the proxy permissions further. Validate the required API access against the versions of DockDash and `docker-socket-proxy` you deploy.

See the upstream project's [Docker socket security guidance](https://github.com/dougmaitelli/DockDash#security) for current recommendations.

---

## Image update monitoring

DockDash can check whether newer images are available for running containers. Where it can resolve the image's source repository and version metadata, it can also display the corresponding GitHub release notes.

For example, a dashboard might show:

```text
Running version:   1.25.0
Available version: 1.26.0
```

The exact information depends on the image tag, registry metadata and repository information available for the container.

### Recommendations

- Prefer versioned image tags where practical for production services.
- Review release notes before updating important services.
- Remember that a moving tag such as `latest` can point to a different image without changing the tag name.
- Use DockDash's update information as a monitoring aid, not as a substitute for reviewing changes and maintaining backups.

---

## Architecture

A typical installation with the standard Docker socket mapping looks like this:

```text
                         Unraid server
                              |
                              v
                    +-------------------+
                    |      DockDash     |
                    |    Web UI :3001   |
                    +---------+---------+
                              |
                       Docker API socket
                    /var/run/docker.sock
                              |
                              v
                       Docker Engine
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
      Containers           Networks          Image registries
          |                                       |
          +---------- health / resources ----------+
                         updates and
                       service discovery
```

When using the standard template, DockDash communicates with the local Docker Engine through the mounted socket. Its persistent application data is stored in `/mnt/user/appdata/dockdash`.

With a restricted socket proxy, Docker API traffic is routed through that proxy instead of mounting the host socket directly into DockDash.

---

## Troubleshooting

### The WebUI does not open

- Confirm the container is running.
- Check the container logs for startup errors.
- Verify that host port `4001` is not already in use.
- Open `http://YOUR_UNRAID_IP:4001/` or use the host port selected in the template.

### DockDash opens, but no containers appear

- Confirm that `/var/run/docker.sock` is mapped correctly in the template.
- Check the container logs for Docker API connection errors.
- If using a Docker socket proxy, confirm DockDash and the proxy share a Docker network and that `DOCKER_HOSTS` points to the correct proxy address.
- Check whether the proxy's permissions allow the Docker API operations required by your selected features.

### Network discovery finds unexpected results or no services

- Review `NETWORK_CIDRS` and set it to the actual LAN subnet.
- Check that the container has network connectivity to the target subnet.
- Remember that host firewalls, VLANs, routing and other network rules can prevent discovery.
- Only scan network ranges you own or are authorised to assess.

### OIDC login fails

- Verify the issuer URL, client ID and client secret.
- Check that the callback/redirect URI registered with the identity provider matches the public URL used to access DockDash.
- If you are behind a reverse proxy, verify forwarded protocol and host headers and consider setting `OIDC_REDIRECT_URI` explicitly.
- Review the DockDash and identity-provider logs.

### Data disappears after recreating the container

Confirm that `/mnt/user/appdata/dockdash` is mounted to `/app/data` and that the host directory remains available.

---

## Useful links

- **Upstream GitHub repository:** https://github.com/dougmaitelli/DockDash
- **Configuration reference:** https://github.com/dougmaitelli/DockDash/blob/master/docs/CONFIGURATION.md
- **Architecture documentation:** https://github.com/dougmaitelli/DockDash/blob/master/docs/ARCHITECTURE.md
- **Security policy:** https://github.com/dougmaitelli/DockDash/blob/master/SECURITY.md
- **Releases:** https://github.com/dougmaitelli/DockDash/releases
- **Report an upstream issue:** https://github.com/dougmaitelli/DockDash/issues
- **Unraid template repository:** https://github.com/MikaPST/unraid-community-apps
- **Report an Unraid template issue:** https://github.com/MikaPST/unraid-community-apps/issues

## Credits and license

DockDash is developed and maintained by [Douglas Maitelli](https://github.com/dougmaitelli). The upstream project is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)**. See the [upstream license file](https://github.com/dougmaitelli/DockDash/blob/master/LICENSE).

The Unraid template is maintained by **MikaPST** as an unofficial community contribution. It is not an official Unraid or DockDash project.
