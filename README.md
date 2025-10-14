# Jellyfin: Easy Setup (Windows, macOS, Linux)

This guide walks you through setting up a local Jellyfin media server using Docker. It’s written for beginners — no prior Git or Docker experience required. You’ll install the required tools, create a simple configuration file (`.env`), and start the server.

If you prefer not to use Docker, there’s a short “Without Docker” note near the end pointing you to native installers.

---

## What You’ll Get
- A Jellyfin server running on your computer
- Your media available at `http://localhost:8096`
- A simple, reusable setup controlled with `docker compose`

---

## Step 0 — Requirements
- A computer running Windows 10/11, macOS (Intel/Apple Silicon), or a modern Linux distro
- An administrator account on the machine (to install software)
- 2–4 GB free RAM and enough disk space for your media

---

## Step 1 — Install Required Tools

Choose your operating system and follow the steps.

### Windows 10/11
1. Install Git for Windows:
   - Download: https://git-scm.com/download/win
   - Accept defaults during installation.
2. Install Docker Desktop for Windows:
   - Download: https://www.docker.com/products/docker-desktop/
   - Requirements: enable virtualization (often on by default) and WSL 2 (the installer helps you enable it).
   - Launch Docker Desktop once after install to finish setup.

### macOS (Intel or Apple Silicon)
1. Install Git:
   - Easiest: open Terminal and run `xcode-select --install` (Command Line Tools include Git), or install via Homebrew `brew install git`.
2. Install Docker Desktop for Mac:
   - Download: https://www.docker.com/products/docker-desktop/
   - Open the app once to complete setup.

### Linux (Debian/Ubuntu-based)
1. Install Git:
   ```bash
   sudo apt update && sudo apt install -y git
   ```
2. Install Docker Engine and Compose plugin:
   ```bash
   # Official convenience script (simple and reliable)
   curl -fsSL https://get.docker.com | sudo sh

   # Enable and start Docker
   sudo systemctl enable --now docker

   # Optional: run Docker without sudo
   sudo usermod -aG docker "$USER"
   # Log out and back in (or reboot) to apply group change
   ```

### Linux (Fedora/RHEL/CentOS)
1. Install Git and Docker:
   ```bash
   sudo dnf install -y git docker
   sudo systemctl enable --now docker
   sudo usermod -aG docker "$USER"
   # Log out/in to apply group change
   ```

Note: On Linux, the Compose plugin is included with recent Docker packages. Use `docker compose ...` (with a space), not `docker-compose` (hyphen).

---

## Step 2 — Get the Project Files
Pick one method.

- With Git (recommended):
  ```bash
  git clone https://example.com/your/jellyfin.git
  cd jellyfin
  ```
- Without Git: Download the ZIP of this repository, extract it, and open the folder in your file manager or terminal.

This folder is your “project root”. The files you create below go here.

---

## Step 3 — Create Your `.env` File
A `.env` file stores simple settings (like folder paths and ports) used by Docker Compose. Create a new file named `.env` in the project root with the contents below, then adjust values for your computer.

Example `.env`:
```ini
# Timezone in Region/City format (see https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
TZ=America/New_York

# Which Jellyfin image tag to run
JELLYFIN_VERSION=latest

# Ports on your computer. Change if 8096/8920 are busy.
JELLYFIN_HTTP_PORT=8096
JELLYFIN_HTTPS_PORT=8920

# Where to store Jellyfin data on your computer
# These folders will be created if missing.
CONFIG_DIR=./.data/config
CACHE_DIR=./.data/cache

# Path to your media on your computer
# Examples:
#   Windows: C:\\Users\\YourName\\Videos
#   macOS:   /Users/yourname/Movies
#   Linux:   /home/yourname/media
MEDIA_DIR=./media

# Linux only (optional on Windows/macOS)
# Match these to your user so Jellyfin can read/write files it creates.
# Find values with: `id -u` and `id -g`
PUID=1000
PGID=1000
```

Tips:
- If you use `./.data/...` and `./media`, they’re relative to the project folder (easy to back up or move).
- On Windows, backslashes in `.env` values are fine (e.g., `C:\Users\You\Videos`).
- On Linux with SELinux (Fedora/RHEL), you might need a `:z` volume flag (see Troubleshooting).

---

## Step 4 — Create `docker-compose.yml`
If this repository doesn’t already include a `docker-compose.yml`, create one in the project root with the following content:

```yaml
version: "3.8"
services:
  jellyfin:
    image: jellyfin/jellyfin:${JELLYFIN_VERSION}
    container_name: jellyfin
    environment:
      - TZ=${TZ}
      - PUID=${PUID}
      - PGID=${PGID}
    ports:
      - "${JELLYFIN_HTTP_PORT}:8096"
      - "${JELLYFIN_HTTPS_PORT}:8920"
    volumes:
      - "${CONFIG_DIR}:/config"
      - "${CACHE_DIR}:/cache"
      - "${MEDIA_DIR}:/media"
    restart: unless-stopped
```

Notes:
- The `/media` path inside the container is where you’ll point Jellyfin when adding libraries.
- On Windows/macOS with Docker Desktop, Windows/mac paths in `MEDIA_DIR` work directly.

---

## Step 5 — Start Jellyfin
From the project folder in your terminal:

```bash
# Create local folders if you used the defaults
mkdir -p ./.data/config ./.data/cache ./media 2>/dev/null || true

# Start Jellyfin in the background
docker compose up -d

# Check status
docker compose ps
```

Access Jellyfin in your browser: `http://localhost:8096`

Follow the setup wizard:
- Create an admin account
- Add a Media Library and point it to `/media` (that maps to your `MEDIA_DIR` on the host)
- Finish setup and start scanning

---

## Day‑to‑Day Commands
```bash
# View logs (Ctrl+C to exit)
docker compose logs -f

# Stop the server
docker compose down

# Update to the latest image
docker compose pull && docker compose up -d
```

Backups:
- Your config lives in `CONFIG_DIR` on your computer. Back up that folder to preserve users, libraries, and settings.

---

## Troubleshooting
- Port already in use: Change `JELLYFIN_HTTP_PORT` (and/or `JELLYFIN_HTTPS_PORT`) in `.env`, then `docker compose up -d` again.
- Linux file permissions: Ensure `PUID`/`PGID` match your user (`id -u`, `id -g`). If needed: `sudo chown -R $UID:$GID ./.data` and your media folders.
- SELinux (Fedora/RHEL): Add `:z` to each volume in `docker-compose.yml`, e.g., `"${MEDIA_DIR}:/media:z"`.
- Firewall: Allow inbound connections to your chosen HTTP port (default 8096).
- Docker Desktop not running: Open Docker Desktop first, wait until it shows “Running”, then retry.

---

## Optional — Run Without Docker
Prefer native installs? See the official Jellyfin downloads and docs for your OS:
- https://jellyfin.org/downloads/
- https://jellyfin.org/docs/

Docker is recommended for simplicity, isolation, and easy updates, but native installs work well if you’re comfortable with manual setup.

---

## Uninstall / Clean Up
From the project folder:
```bash
# Stop and remove the container
docker compose down

# Optional: remove downloaded images
# WARNING: this removes the Jellyfin image from your machine
docker image rm jellyfin/jellyfin || true

# Optional: remove local data (deletes ALL Jellyfin settings and metadata)
# Be careful — this is irreversible.
rm -rf ./.data
```

---

## Quick Reference
- Start: `docker compose up -d`
- Open: `http://localhost:8096`
- Logs: `docker compose logs -f`
- Update: `docker compose pull && docker compose up -d`
- Config backup: back up `CONFIG_DIR`

You’re all set — enjoy your media server!
