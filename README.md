<div align="center">
  <h1>🧭 nginx-proxy</h1>
  <p><strong>Automatic virtual host reverse proxy for Docker with Let's Encrypt</strong></p>

  <p>
    <a href="#quick-start"><img alt="Get Started" src="https://img.shields.io/badge/Get%20Started-1ea463?style=for-the-badge&logo=rocket" /></a>
    <a href="#using-with-your-app-containers"><img alt="How To Use" src="https://img.shields.io/badge/How%20to%20Use-5865F2?style=for-the-badge&logo=readme" /></a>
    <a href="docker-compose.yml"><img alt="Open docker-compose.yml" src="https://img.shields.io/badge/docker%20compose-yml-0db7ed?style=for-the-badge&logo=docker" /></a>
  </p>

  <p>
    <img alt="Docker" src="https://img.shields.io/badge/Docker-✔-2496ED?style=flat&logo=docker&logoColor=white" />
    <img alt="NGINX" src="https://img.shields.io/badge/NGINX-Enabled-009639?style=flat&logo=nginx&logoColor=white" />
    <img alt="ACME" src="https://img.shields.io/badge/ACME%20Companion-Let's%20Encrypt-F7B93E?style=flat" />
    <img alt="Status" src="https://img.shields.io/badge/Usage-Production%20ready-success?style=flat" />
  </p>
</div>

<p align="center">
  <a href="#overview">Overview</a> •
  <a href="#prerequisites">Prerequisites</a> •
  <a href="#quick-start">Quick start</a> •
  <a href="#using-with-your-app-containers">Use with your apps</a> •
  <a href="#docker-run-example-your-app-container">Docker run example</a> •
  <a href="#included-compose-services">Included services</a>
  </p>

## Overview
<a id="overview"></a>
Simple setup for `jwilder/nginx-proxy` with Let's Encrypt companion. It automatically routes requests based on hostnames and can provision certificates via ACME.

## Prerequisites
<a id="prerequisites"></a>
- Docker and Docker Compose
- An external Docker network named `web` that your apps join:

```bash
docker network create web
```

## Quick start
<a id="quick-start"></a>
Bring up the proxy and ACME companion:

```bash
docker compose up -d
```

This stack exposes ports 80 and 443 and listens for containers on the `web` network.

## Using with your app containers
<a id="using-with-your-app-containers"></a>
Attach your containers to the same `web` network and set the environment variables below.

### Required/optional environment variables
- **VIRTUAL_HOST**: Comma-separated hostnames to route to this container (e.g., `app.example.com`).
- **LETSENCRYPT_HOST**: Hostname(s) for which to request TLS certificates. Typically the same as `VIRTUAL_HOST`.
- **LETSENCRYPT_EMAIL** (optional): Email used for ACME registration and renewal notices.
- **HOST_PORT** (optional): Port your app listens on inside the container. If omitted, the proxy detects the port automatically from `EXPOSE` or `ports`.

### Example service
```yaml
services:
  whoami:
    image: traefik/whoami
    container_name: whoami
    restart: always
    expose:
      - "80"          # alternatively, publish and set HOST_PORT
    environment:
      - VIRTUAL_HOST=example.com
      - LETSENCRYPT_HOST=example.com
      - LETSENCRYPT_EMAIL=you@example.com   # optional
      # - HOST_PORT=80                      # optional; auto-detected if omitted
    networks:
      - web

networks:
  web:
    external: true
```

Notes:
- Use `expose` to make the internal port visible to the proxy without publishing it on the host. If you instead publish with `ports`, `HOST_PORT` is typically unnecessary because the proxy will detect the exposed/published port.
- Ensure your DNS records point to the server running this proxy.

### Docker run example (your app container)
<a id="docker-run-example-your-app-container"></a>
If you prefer `docker run`, here's an example using all supported env variables for an app container (not the proxy):

```bash
# Create the external network once
docker network create web

# Run your application container
docker run -d \
  --name whoami \
  --restart always \
  --expose 80 \
  -e VIRTUAL_HOST=example.com \
  -e LETSENCRYPT_HOST=example.com \
  -e LETSENCRYPT_EMAIL=you@example.com \  # optional
  -e HOST_PORT=80 \                         # optional; auto-detected if omitted
  --network web \
  traefik/whoami
```

Notes:
- Replace `example.com` and the email with your domain and contact.

## Included compose services
<a id="included-compose-services"></a>
This repository's `docker-compose.yml` defines:
- `nginx-proxy` (listens on 80/443)
- `nginx-proxy-acme` companion (obtains and renews certificates)

Both services join the external `web` network.
