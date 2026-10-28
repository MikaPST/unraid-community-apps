<div align="center">

<img src="https://raw.githubusercontent.com/MikaPST/unraid-community-apps/main/icons/icon-web-check.png" alt="Web-Check icon" width="128">

# Web-Check on Unraid

**On-demand OSINT and website analysis, self-hosted on your Unraid server**

[![Unraid Community Apps](https://img.shields.io/badge/Unraid-Community%20Apps-0E83CD?logo=unraid)](https://unraid.net/)
[![Docker Pulls](https://img.shields.io/docker/pulls/lissy93/web-check?logo=docker)](https://hub.docker.com/r/lissy93/web-check)
[![Docker Image Version](https://img.shields.io/docker/v/lissy93/web-check?sort=semver)](https://hub.docker.com/r/lissy93/web-check)
[![Upstream License](https://img.shields.io/badge/License-MIT-green.svg)](https://github.com/Lissy93/web-check/blob/master/LICENSE)
[![Unraid templates](https://img.shields.io/badge/Unraid%20templates-MikaPST-181717?logo=github)](https://github.com/MikaPST/unraid-community-apps)

[**Web-Check website**](https://web-check.xyz/) ·
[**Upstream GitHub project**](https://github.com/Lissy93/web-check) ·
[**Docker Hub**](https://hub.docker.com/r/lissy93/web-check)

</div>

---

## Table of contents

- [Overview](#overview)
- [What can Web-Check analyse?](#what-can-web-check-analyse)
- [Install on Unraid](#install-on-unraid)
- [First use](#first-use)
- [Configuration](#configuration)
  - [API keys and external services](#api-keys-and-external-services)
  - [Advanced options](#advanced-options)
  - [Restrict or select checks](#restrict-or-select-checks)
  - [Using a reverse proxy](#using-a-reverse-proxy)
- [Security recommendations](#security-recommendations)
- [Troubleshooting](#troubleshooting)
- [Useful links](#useful-links)
- [Credits and license](#credits-and-license)

---

## Overview

[Web-Check](https://web-check.xyz/) is an open-source OSINT tool for inspecting a website or domain and gathering useful information about its public-facing configuration.

This Unraid template deploys the upstream Docker image, `lissy93/web-check`, using a standard Docker `bridge` network. The core functionality works without API keys; optional credentials can enable or enrich certain checks.

> **This repository provides an Unraid installation template.** Web-Check itself is developed and maintained by the upstream project at [Lissy93/web-check](https://github.com/Lissy93/web-check).

### At a glance

| Setting | Value |
|---|---|
| Docker image | `lissy93/web-check:latest` |
| Container port | `3000/tcp` |
| Default Unraid host port | `3000` |
| Default Docker network | `bridge` |
| Persistent volume required | No, not for the standard deployment |
| API keys required | No, optional for additional checks |
| Upstream license | MIT |

## What can Web-Check analyse?

Depending on the check and the target, Web-Check can gather information such as:

- IP address and hosting information
- SSL/TLS certificates
- DNS records and DNS security
- HTTP headers, cookies and redirects
- Domain information and crawl rules
- Open ports and traceroute
- Page performance and technology indicators
- Trackers and associated hostnames
- Additional intelligence from supported third-party services

Not every check is guaranteed to return a result for every target. Results depend on the target, network reachability, third-party services, rate limits and any optional API credentials you configure.

---

## Install on Unraid

1. Open **Apps** in the Unraid web interface.
2. Search for **Web-Check**.
3. Select the template provided by **MikaPST — Unraid Community Apps** and click **Install**.
4. Review the configuration. The default HTTP port is `3000`; change the host port if it is already used by another application.
5. Click **Apply** and wait for the container to start.

Open the web interface at:

```text
http://YOUR_UNRAID_IP:3000/
```

Replace `YOUR_UNRAID_IP` with the IP address of your Unraid server. If you changed the host port during installation, use that port instead.

### What the template configures

- The upstream `lissy93/web-check:latest` Docker image
- The `bridge` network by default
- A port mapping from the chosen Unraid host port to container port `3000`
- Optional API keys and advanced environment variables
- API rate limiting enabled by default in this template

A persistent application-data directory is not required for a standard deployment.

---

## First use

1. Open Web-Check in your browser.
2. Enter the domain or URL you want to examine.
3. Start the analysis.
4. Review the resulting sections, such as IP information, TLS certificates, DNS, headers, redirects and other available checks.

**Only scan websites and systems that you own or are explicitly authorised to assess.** Web-Check performs active checks from the server running the container, so the requests originate from your Unraid host or its network.

---

## Configuration

The template exposes the following optional variables. Most users can leave the defaults unchanged and use Web-Check without API keys.

### API keys and external services

API keys are optional. They enable or enrich specific checks and may be subject to third-party quotas, terms and data-handling policies.

| Variable | Purpose |
|---|---|
| `GOOGLE_CLOUD_API_KEY` | Additional Google-related checks |
| `SHODAN_API_KEY` | Additional host, hostname or security intelligence |
| `WHO_API_KEY` | Additional WHOIS/domain information through WhoAPI |
| `SECURITY_TRAILS_API_KEY` | Additional domain and IP intelligence |
| `URL_SCAN_API_KEY` | Additional URL intelligence through URLScan |
| `CLOUDMERSIVE_API_KEY` | Additional security and threat-intelligence checks |
| `TRANCO_USERNAME` | Username associated with a Tranco account |
| `TRANCO_API_KEY` | Tranco API access |
| `TORRENT_IP_API_KEY` | Additional IP intelligence |
| `BUILT_WITH_API_KEY` | Additional website technology information |

Leave a value empty if you do not use that service. Obtain credentials from the relevant provider and review that provider's terms before enabling it.

**Important:** the exact checks supported by an API key can vary between upstream versions. Refer to the [upstream Web-Check documentation](https://github.com/Lissy93/web-check) for the current details.

### Advanced options

| Variable | Default in this template | Purpose |
|---|---:|---|
| `API_ENABLE_RATE_LIMIT` | `true` | Enables rate limiting for API requests |
| `PUBLIC_API_TIMEOUT_LIMIT` | `25000` | Maximum API request timeout, in milliseconds |
| `API_CORS_ORIGIN` | Empty | Optional CORS origin configuration |
| `TRUST_PROXY` | Empty | Optional configuration when running behind a trusted reverse proxy |

For most installations, keep rate limiting enabled. Increase timeouts only when you understand why checks are taking longer and have considered the resource impact.

### Restrict or select checks

These settings let you control which checks the application runs.

| Variable | Example | Purpose |
|---|---|---|
| `API_DISABLED_CHECKS` | `ports,trace-route` | Comma-separated list of checks to disable |
| `API_ENABLED_CHECKS` | `get-ip,ssl,dns,headers` | If set, run only the listed checks |
| `API_BLOCKED_HOSTS` | `example.internal,192.168.0.0/16` | Comma-separated hosts or networks to block from scanning |

The values above are examples, not mandatory settings. Use check names recognised by the current upstream version.

`API_ENABLED_CHECKS` is an allow-list: when populated, checks not listed may no longer run. Avoid configuring `API_ENABLED_CHECKS` and `API_DISABLED_CHECKS` without first understanding how the upstream application applies both settings.

**Do not rely on `API_BLOCKED_HOSTS` as your only network security control.** Apply suitable firewall and network rules as well, especially if the instance is reachable by other users.

### Using a reverse proxy

You can publish Web-Check through a reverse proxy such as Traefik or Nginx Proxy Manager.

1. Keep the container's internal port set to `3000/tcp`.
2. Configure the proxy to forward to the Unraid host and published host port, or connect the container to the proxy's Docker network and forward directly to `web-check:3000`.
3. Terminate HTTPS at the reverse proxy.
4. If you configure `TRUST_PROXY`, trust only the proxy or proxies that actually control the forwarded client-address headers.
5. Protect the application with authentication or restrict access using a VPN or trusted network.

Do not assume that putting an application behind a reverse proxy automatically adds authentication. Configure an authentication layer explicitly if the service should not be publicly accessible.

---

## Security recommendations

Web-Check is designed to inspect arbitrary targets. This means a user who can access your instance may be able to make the server send requests to domains or hosts of their choosing.

- **Do not expose an unprotected instance directly to the public Internet.** Use a VPN, restrict network access, or add authentication at the reverse proxy.
- **Keep API rate limiting enabled**, as configured by this template.
- **Limit network reachability.** Use firewall rules and appropriate Docker networks to reduce access to internal services and sensitive networks.
- **Use `API_BLOCKED_HOSTS` and check-selection settings as additional safeguards**, not as a replacement for network-level controls.
- **Treat API keys as secrets.** Do not share screenshots or configuration exports that reveal them, and use restricted credentials where available.
- **Keep the image updated.** Consult the upstream repository and security advisories for current fixes.

Web-Check may send queries or target information to external services as part of particular checks. Review the upstream documentation and the privacy terms of services you enable before using the tool with sensitive targets.

## Troubleshooting

### The WebUI does not open

- Confirm that the container is running.
- Check that the host port you selected is not already in use.
- Make sure you are using the Unraid server's IP address and the **host port**, not necessarily the container port.
- Review the container logs for startup errors.

### Some checks show errors or no results

- Some checks depend on the target's configuration or network accessibility.
- Some checks need optional API credentials or may be limited by a provider's quota.
- Try again later if a third-party service is temporarily unavailable.
- Check the upstream project documentation for check-specific requirements.

### The interface is reachable through a reverse proxy, but results fail

- Confirm that the proxy forwards to the correct container port.
- Verify that the container can reach the Internet and perform DNS lookups.
- If you use custom proxy or CORS settings, review `TRUST_PROXY` and `API_CORS_ORIGIN`.
- Check the container logs and reverse-proxy logs for errors.

---

## Useful links

- **Official website:** https://web-check.xyz/
- **Upstream GitHub repository:** https://github.com/Lissy93/web-check
- **Docker Hub image:** https://hub.docker.com/r/lissy93/web-check
- **Live demo:** https://web-check.as93.net/
- **Unraid template repository:** https://github.com/MikaPST/unraid-community-apps
- **Report an upstream issue:** https://github.com/Lissy93/web-check/issues
- **Report an Unraid template issue:** https://github.com/MikaPST/unraid-community-apps/issues

## Credits and license

Web-Check is an open-source project created and maintained by [Alicia Sykes / Lissy93](https://github.com/Lissy93). The upstream project is licensed under the **MIT License**; see the [upstream license file](https://github.com/Lissy93/web-check/blob/master/LICENSE).

The Unraid template in this repository is maintained by **MikaPST** and is an unofficial community contribution. It is not an official Unraid or Web-Check project.
