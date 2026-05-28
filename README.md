# Resilient Home-Lab Docker Compose Stack 🚀

This repository contains a highly optimized, production-grade, and resilient multi-container Docker Compose layout designed for home-lab services. It includes search, media streaming, downloads, change detection, and headless browsing engines bound together by a robust, secure environment overriding system.

---

## 🏗️ Services Overview

1.  **[SearXNG](https://docs.searxng.org/)**: A privacy-respecting, self-hosted metasearch engine.
2.  **[Valkey](https://valkey.io/)**: A high-performance, open-source key-value database (fully API-compatible Redis fork) acting as SearXNG's caching and rate-limiting (bot-protection) backend.
3.  **[qBittorrent](https://www.qbittorrent.org/)**: A lightweight, secure torrent downloader (includes automated, custom Python search engine setups).
4.  **[Jellyfin](https://jellyfin.org/)**: The ultimate voluntary media system for organizing and streaming your private libraries (configured with host-networking and AMD hardware acceleration).
5.  **[ChangeDetection.io](https://changedetection.io/)**: (Optional) Powerful self-hosted website change monitoring engine.
6.  **[Flaresolverr](https://github.com/FlareSolverr/FlareSolverr)**: (Optional) Proxy server to bypass DDoS protection mechanisms for scraping and indices.

---

## 💪 Core Architecture & Resiliency Upgrades

This compose stack is built with the highest standards of production container orchestration:

### 1. 🌲 Hierarchical Environment Overrides
All service definitions support a modern, 4-layered environment file hierarchy. Variables are evaluated sequentially (later files override/supersede earlier ones), allowing host-specific overrides to be kept strictly separate from the base configurations:
```yaml
    env_file:
      - path: ../.env          # Global default settings
        required: false
      - path: ../.env.local    # Global machine-specific overrides
        required: false
      - path: .env             # Service-specific default settings
        required: false
      - path: .env.local       # Service-specific machine-specific overrides
        required: false
```

### 2. 🛡️ 100% Secure & Public Ready
*   **Zero Hardcoded Secrets**: Cryptographic keys like `SEARXNG_SECRET` are passed dynamically from `.env` using environment variables. No secrets are stored in `settings.yml`.
*   **Privacy-Friendly Directory Mounts**: Your physical host storage directories (e.g. `/mnt/...`) are kept in your local `.env` and are strictly excluded from version control via `.gitignore`.
*   **Clean `.env.example`**: A fully commented template is provided for a seamless open-source setup experience.

### 3. 📉 Resource Boundaries
Every container is capped with memory and CPU boundaries using Docker's `deploy.resources.limits` configuration to prevent memory leaks or background loop bugs from freezing your host system.

### 4. 🪵 Log Rotations
To protect your host disk from filling up, all containers are constrained to standard JSON file logging rotations (`max-size: "10m"`, `max-file: "3"`).

### 5. 🧟 Zombie Process Reaping
Containers running headless Chromium instances (`browser-sockpuppet-chrome` and `flaresolverr`) are configured with `init: true`. This invokes the lightweight Docker init-system to automatically reap zombie child processes.

---

## ⚡ Quick-Start & Deployment

### Prerequisite: Set Up Environment Files
1.  Clone this repository to your host machine.
2.  Copy the template file to `.env`:
    ```bash
    cp .env.example .env
    ```
3.  Open the newly created `.env` file and customize the volume paths (`MEDIA_BOOKS`, `DOWNLOADS_DIR`, etc.) to point to your storage directories.
4.  Generate a unique, cryptographically secure key for SearxNG and add it to your `.env` under `SEARXNG_SECRET`:
    ```bash
    openssl rand -hex 32
    ```

### Running the Services
Bring up the entire stack in the background:
```bash
docker compose up -d
```

Check the health status of all running containers:
```bash
docker compose ps
```

---

## 📋 Default Port Configurations

| Service | Host Port | Internal Port | Environment Override Key |
| :--- | :--- | :--- | :--- |
| **SearXNG** | `8888` | `8888` | `SEARXNG_PORT` |
| **qBittorrent** | `8080` | `8080` | `QBITTORRENT_PORT` |
| **ChangeDetection** | `5000` | `5000` | `CHANGEDETECTION_PORT` |
| **Flaresolverr** | `8191` | `8191` | `FLARESOLVERR_PORT` |
| **Jellyfin** | *Host Network* | `8096` | *Managed via Host Net* |
