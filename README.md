# Global Proxy Gateway

This repository contains the configuration for a centralized NGINX Reverse Proxy (Gateway) running on Docker. It routes external traffic from different domains and subdomains (like `firmador.top` and `equinoxbytes.com`) to their respective services running on the same VPS.

By keeping the proxy in an independent stack, you can add, modify, or restart routing rules and subdomains without interrupting or restarting your main application stacks.

---

## Prerequisites

Before starting this stack, you must ensure that the shared external Docker network exists on the host VPS. This network allows NGINX to resolve and communicate with other containers running in different Docker Compose stacks securely.

Create the network by running:
```bash
docker network create proxy-net
```

---

## Directory Structure

```text
proxy-gateway/
├── docker-compose.yml
├── README.md
└── nginx/
    ├── Dockerfile
    └── gateway.conf
```

* **`docker-compose.yml`**: Defines the NGINX service, exposing ports `80` (HTTP) and `443` (HTTPS) to the host, and binds it to the external `proxy-net` network.
* **`nginx/Dockerfile`**: Simple Dockerfile using `nginx:alpine` that replaces the default configuration with our custom rules.
* **`nginx/gateway.conf`**: The main NGINX server block configuration where all domain routing rules are defined.

---

## How to Add or Modify Routes

To edit routing rules, modify the [`nginx/gateway.conf`](nginx/gateway.conf) file. There are two primary routing methods:

### 1. Routing to Containers in the Same Network
If the target service is running in another Docker Compose stack on the same VPS, connect that service to the `proxy-net` network in its `docker-compose.yml`. You can then proxy directly using the container name and internal port:

```nginx
server {
    listen 80;
    server_name myapp.mydomain.com;

    location / {
        proxy_pass http://my-container-name:port;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 2. Routing to Host Ports (Other Docker Ecosystems or Standalone Apps)
If the service is not connected to `proxy-net` but exposes a port publicly or locally on the host VPS (e.g., port `4000`), proxy traffic using the host's IP address:

```nginx
server {
    listen 80;
    server_name myapp.mydomain.com;

    location / {
        proxy_pass http://37.60.249.189:port; # VPS Host IP
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## Deploying Changes

Every time you modify the configuration files, you need to rebuild and restart the NGINX container.

Run the following commands in the project directory:

```bash
# Rebuild the gateway image and recreate the container in the background
docker compose up -d --build gateway
```
