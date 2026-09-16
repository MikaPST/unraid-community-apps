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

## Applications

### Sablier

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

### Important: Docker network

The template uses the Docker network:

```text
traefik
```

by default, because this is a common configuration when Sablier is used with Traefik.

If you use another reverse proxy, you can change the Docker network to the network used by your reverse proxy.

Sablier itself is not limited to Traefik.

---

# Traefik integration

Sablier can be integrated with Traefik through the official Sablier Traefik plugin.

## 1. Enable the Sablier plugin in Traefik

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

---

## 2. Create a Sablier middleware

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

# Activating Sablier on an application

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

# Custom Sablier themes

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

## Sablier Theme Editor

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

# Example architecture

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

# Useful links

## Sablier

**Official website**

https://sablierapp.dev/

**Official documentation / tutorials**

https://sablierapp.dev/tutorials/

**Sablier GitHub repository**

https://github.com/sablierapp/sablier

## Traefik

**Sablier Traefik plugin**

https://plugins.traefik.io/plugins/69104ac3b7d4dd76110a1a09/sablier

---

# Repository structure

```text
.
├── ca_profile.xml
├── icon.png
├── icon-sablier.png
├── README.md
├── templates/
│   └── sablier.xml
└── plugins/
```

Each Docker application should have its own XML template under:

```text
templates/
```

For example:

```text
templates/sablier.xml
templates/application2.xml
templates/application3.xml
```

---

# Support

For issues concerning the **Sablier application itself**, please refer to the official Sablier project:

https://github.com/sablierapp/sablier

For issues specifically related to the **Unraid template maintained in this repository**, please open an issue in this GitHub repository.

---

# License

This repository contains Unraid application templates maintained by MikaPST.

The Sablier application is distributed under the **Apache License 2.0**.
