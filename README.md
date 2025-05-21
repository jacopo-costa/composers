# Self-Hosted Services with Docker Compose

This repository contains a collection of Docker Compose configurations for self-hosted services, organized into folders by service category. Each service has its own directory containing a `docker-compose.yml` file and a structure for persistent storage. Environment-specific secrets and configuration are loaded from `.env` files (which are **not included** in this repository).

> ⚠️ **Important:** Ensure you create a `.env` file for each service before launching any stack. These files are ignored in version control and must be maintained separately.

---

## 📁 Repository Structure

Each directory represents a service or stack of related services. Here's a brief overview:

### 🧰 Core Service Stacks

- **`arr-stack/`**  
  Hosts media-related services including:

  - Radarr
  - Sonarr
  - qBittorrent
  - Jellyfin
  - Prowlarr
  - Jellyseerr

- **`authentik/`**  
  Identity provider for Single Sign-On and secure access.

- **`traefik/`**  
  Reverse proxy and load balancer with support for automatic HTTPS.

- **`crowdsec/`**  
  Collaborative security engine for detecting and blocking threats.

- **`immich/`**  
  High-performance photo and video backup and management service.

- **`grafana/`**  
  Visualization stack using:

  - Grafana
  - Prometheus

- **`nextcloud/`**  
  Nextcloud All-in-One for private cloud file storage and more.

- **`vaultwarden/`**  
  Lightweight Bitwarden-compatible password manager.

---

## 🗂 File Structure Per Service

Each service directory follows this pattern:

```

service-name/
├── docker-compose.yml
├── data/                  # Persistent volumes
└── .env                   # Environment variables (not committed)

```

### Example: `authentik/`

```

authentik/
├── docker-compose.yml
├── certs/
├── custom-templates/
├── media/
└── .env

```

---

## 🌐 Docker Networks

Each service stack uses a **custom Docker network** for internal communication. You **must create these networks manually** before launching the stack.

- Example: The `authentik` stack defines a custom network `authentik`, shared between its `server` and `worker` containers.
- External-facing services (e.g., Vaultwarden, Nextcloud, Authentik) are also connected to the shared `traefik` network so the reverse proxy can route requests.

### ✅ Creating Networks

```bash
docker network create traefik
docker network create authentik
docker network create arr
# and others as needed
```

---

## 🌍 Traefik Integration

External-facing services are exposed to the web through the **Traefik reverse proxy**. This is done using **Traefik labels** in the `docker-compose.yml` files. These labels define routes, security settings, and middleware.

### 📌 Example: Vaultwarden with Traefik

```yaml
labels:
  - traefik.enable=true
  - traefik.docker.network=traefik
  - traefik.http.routers.vaultwarden.rule=Host(`vault.dimoracosta.it`)
  - traefik.http.routers.vaultwarden.entrypoints=websecure
  - traefik.http.routers.vaultwarden.tls=true
  - traefik.http.routers.vaultwarden.tls.certResolver=cloudflare
  - traefik.http.routers.vaultwarden.middlewares=crowdsec-bouncer@file,authentik@file,ratelimiter@file
  - traefik.http.services.vaultwarden.loadbalancer.server.port=80
```

#### Key Elements:

- `traefik.enable=true`: Enables Traefik routing for the container
- `traefik.docker.network=traefik`: Specifies the network Traefik uses to communicate with the service
- `traefik.http.routers.<name>.rule`: Sets the domain rule (e.g., based on `Host(...)`)
- `entrypoints=websecure`: Uses HTTPS entrypoint
- `tls.certResolver`: Chooses which certificate resolver to use (e.g., Cloudflare DNS challenge)
- `middlewares`: Applies custom middlewares like rate limiting or authentication

Make sure the service container is connected to the `traefik` network:

```yaml
networks:
  - traefik
  - <internal-network-name>
```

---

## 🚀 Getting Started

1. **Clone this repository:**

   ```bash
   git clone https://github.com/jacopo-costa/composers.git
   cd composers
   ```

2. **Create required Docker networks:**

   ```bash
   docker network create traefik
   docker network create authentik
   docker network create arr
   # and others as needed
   ```

3. **Create `.env` files for each service:**

   - Create a `.env` file in each service folder if required.
   - Define all variables used in the service's `docker-compose.yml` in the form:

   ```bash
   ADMIN_TOKEN = 'XXX'
   ```

   - Example variables: `${POSTGRES_PASSWORD}`, `${ADMIN_EMAIL}`, etc.

4. **Start a service:**

   ```bash
   cd authentik
   docker compose up -d
   ```

5. **Repeat for any other service you wish to run.**

---

## 🛑 Warning: Keep Your Secrets Safe

- `.env` files contain **sensitive credentials** and **should not** be committed to version control.
- This repository includes a `.gitignore` rule to exclude `.env` and `.env.*` files.
- Always review your environment files before sharing or backing up your configuration.

---

## 📌 Tips

- Use `.env` files to manage secrets per service in a modular way.
- Create Docker networks ahead of time to avoid runtime errors.
- Use Traefik’s dashboard and labels to inspect routing and service status.
- Consider using `.env.example` templates to document expected environment variables.

---

## 📃 License

This project is open-source. Use and modify at your own discretion. You are responsible for securing your services.
