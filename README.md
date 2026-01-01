# NGINX Load Balancing Demo with Node.js

This repository demonstrates how to set up a simple Node.js application, containerize it with Docker, run multiple instances using Docker Compose, and load balance traffic across these instances using NGINX. This is a practical guide for developers who want to learn about reverse proxying, SSL termination, and load balancing with NGINX in a local development environment.

---

## Table of Contents
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Node.js Application](#nodejs-application)
- [Docker & Docker Compose](#docker--docker-compose)
- [NGINX Configuration](#nginx-configuration)
- [Running the Demo](#running-the-demo)
- [Practicing and Customizing](#practicing-and-customizing)
- [License](#license)

---

## Project Structure

```
nginx---config/
├── Dockerfile
├── docker-compose.yaml
├── index.html
├── package.json
├── package-lock.json
├── server.js
└── README.md
```

- `server.js`: Simple Express server serving `index.html`.
- `index.html`: Static landing page.
- `Dockerfile`: Containerizes the Node.js app.
- `docker-compose.yaml`: Defines three app replicas for load balancing.
- `package.json`: Project metadata and dependencies.
- `README.md`: This documentation.

---

## How It Works

1. **Node.js App**: A minimal Express server serves a static HTML page.
2. **Docker Compose**: Spins up three containers (`app1`, `app2`, `app3`), each running the Node.js app on different ports (3001, 3002, 3003).
3. **NGINX**: Acts as a reverse proxy and load balancer, distributing incoming requests to the three Node.js containers. SSL termination is handled by NGINX.

---

## Node.js Application

- [`server.js`](server.js:1):
  - Uses Express to serve [`index.html`](index.html:1) for all routes.
  - Logs which replica handled the request (via `APP_NAME` environment variable).

---

## Docker & Docker Compose

- [`Dockerfile`](Dockerfile:1):
  - Builds a Node.js 14 image, installs dependencies, exposes port 3000.
- [`docker-compose.yaml`](docker-compose.yaml:1):
  - Defines three services (`app1`, `app2`, `app3`), each mapped to a unique host port (3001, 3002, 3003).

---

## NGINX Configuration

The NGINX configuration is typically found at `/usr/local/etc/nginx/nginx.conf` on macOS (Homebrew installs). Here’s the relevant configuration used for this demo:

```
worker_processes 1;

events{
    worker_connections 1024;
}

http{
    include mime.types;

    upstream nodejs_cluster {
        least_conn;
        server 127.0.0.1:3001;
        server 127.0.0.1:3002;
        server 127.0.0.1:3003;
    }
    server {
        listen 443 ssl;
        server_name localhost;

        ssl_certificate /Users/arjo/nginx-certs/nginx-selfsigned.crt;
        ssl_certificate_key /Users/arjo/nginx-certs/nginx-selfsigned.key;

        location / {
            proxy_pass http://nodejs_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real_IP $remote_addr;
        }
    }
    server{
        listen 80;
        server_name localhost;

        location / {
            return 301 https://$host$request_uri;
        }
    }
}
```

### Key Points
- **Upstream Block**: Defines a load-balanced group (`nodejs_cluster`) with three backend servers.
- **SSL Termination**: NGINX listens on 443 with a self-signed certificate. HTTP (port 80) is redirected to HTTPS.
- **Proxying**: Requests to `/` are proxied to the Node.js cluster.

---

## Running the Demo

### Prerequisites
- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/)
- [NGINX](https://nginx.org/en/download.html) (installed locally)
- Self-signed SSL certificates (see below)

### Steps

1. **Clone the Repo**
   ```sh
   git clone <this-repo-url>
   cd nginx---config
   ```
2. **Build and Start the Node.js Cluster**
   ```sh
   docker-compose up --build
   ```
3. **Generate Self-Signed SSL Certificates** (if not already present)
   ```sh
   mkdir -p ~/nginx-certs
   openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout ~/nginx-certs/nginx-selfsigned.key \
     -out ~/nginx-certs/nginx-selfsigned.crt \
     -subj "/CN=localhost"
   ```
   Update the certificate paths in your NGINX config if needed.
4. **Update NGINX Config**
   - Replace `/usr/local/etc/nginx/nginx.conf` with the provided config above.
   - Reload NGINX:
     ```sh
     sudo nginx -s reload
     ```
5. **Access the App**
   - Visit [https://localhost](https://localhost) in your browser.
   - You may need to accept the self-signed certificate warning.
   - Refresh the page to see requests handled by different app replicas (check Docker logs).

---

## Practicing and Customizing

- **Change Replica Count**: Edit [`docker-compose.yaml`](docker-compose.yaml:1) to add/remove app services.
- **Modify NGINX Load Balancing**: Try different load balancing methods (e.g., `round_robin`, `ip_hash`).
- **Add More Routes**: Expand [`server.js`](server.js:1) and [`index.html`](index.html:1) for more complex demos.
- **Use Real Certificates**: Replace the self-signed certs with trusted ones for production.

---

## License

MIT
