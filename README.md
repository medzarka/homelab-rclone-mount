# Homelab Rclone Mount

A robust, production-ready, containerized solution to mount cloud storage remotes (such as Seafile WebDAV, Google Drive, Nextcloud, S3, OneDrive, etc.) to a local host directory using [Rclone](https://rclone.org/) and FUSE.

---

## Features & Enhancements

- **Generic & Configurable:** Mount any Rclone remote path to any host directory via `.env`.
- **FUSE Mount with Shared Propagation (`rshared`):** Makes mounted files accessible to the host system and other Docker containers.
- **Docker Healthcheck:** Actively monitors the FUSE mount status (`ls /data`) to detect stale or hung mounts and report unhealthy status for automatic healing.
- **Persistent VFS Disk Cache:** Docker-managed `rclone-cache` volume preserves cached chunks across container restarts with safety bounds (`--vfs-cache-max-size 10G`, `--vfs-cache-max-age 24h`).
- **Base64 Config Injection & AES-256 Encryption:** Supports both clear and password-protected encrypted `rclone.conf` files.
- **Startup Diagnostics & Error Handling:** Validates environment variables, checks base64 decode status, and unmounts stale FUSE sessions before starting.
- **Log Rotation:** Built-in JSON log rotation prevents disk bloat on the host.

---

## Prerequisites

1. **Docker & Docker Compose** installed on the host.
2. **FUSE Support:** Ensure the host supports FUSE (`/dev/fuse` is present).
3. **Target Directory:** Create the host mount directory before starting the container:
   ```bash
   mkdir -p /srv/data/Development
   ```

---

## Quick Start

### 1. Copy Environment Configuration
```bash
cp .env.example .env
```

### 2. Configure Environment Variables
Edit `.env` with your desired configuration:

| Variable | Description | Example / Default |
| :--- | :--- | :--- |
| `RCLONE_REMOTE_PATH` | Rclone remote and subpath to mount | `seafile:Development` |
| `HOST_MOUNT_DIR` | Directory on the host where files will be mounted | `/srv/data/Development` |
| `CONTAINER_NAME` | Container name (optional) | `rclone-mount` |
| `TZ` | Timezone for container logs | `UTC` |
| `RCLONE_LOG_LEVEL` | Log level (`DEBUG`, `INFO`, `NOTICE`, `ERROR`) | `INFO` |
| `RCLONE_MOUNT_OPTIONS` | Extra mount flags (optional) | `--vfs-cache-mode full --vfs-cache-max-size 10G --vfs-cache-max-age 24h --allow-other --allow-non-empty --buffer-size 32M` |
| `RCLONE_CONF_BASE64` | Base64-encoded `rclone.conf` | Single-line base64 string |
| `RCLONE_CONFIG_PASS` | Password to decrypt config (if encrypted) | Optional / leave empty for clear mode |

### 3. Generate Base64 Rclone Config

#### Mode A: Clear Configuration (Default)
Generate the Base64 string directly:
- **Linux:** `cat ~/.config/rclone/rclone.conf | base64 -w 0`
- **macOS:** `cat ~/.config/rclone/rclone.conf | base64`

Paste the output into `RCLONE_CONF_BASE64` and leave `RCLONE_CONFIG_PASS=""`.

#### Mode B: Encrypted Configuration (Enhanced Security)
1. Encrypt your `rclone.conf` with a master password:
   ```bash
   rclone config
   # Select 's' (Set configuration password) -> choose your password
   ```
2. Convert the encrypted file to Base64:
   - **Linux:** `cat ~/.config/rclone/rclone.conf | base64 -w 0`
   - **macOS:** `cat ~/.config/rclone/rclone.conf | base64`
3. Set `RCLONE_CONFIG_PASS="your_password"` in `.env`.
   - Rclone will automatically decrypt the config in memory upon starting.

### 4. Start the Mount
```bash
docker compose up -d
```

### 5. Check Status & Logs
- Check container health status:
  ```bash
  docker compose ps
  ```
- View real-time logs:
  ```bash
  docker compose logs -f
  ```

---

## Monitoring & Remote Control (Optional)

If you want Prometheus metrics or the Rclone Remote Control API:
1. Uncomment the `ports` section in `docker-compose.yaml`:
   ```yaml
   ports:
     - "5572:5572"
   ```
2. Add `--rc --rc-addr :5572 --rc-no-auth` to `RCLONE_MOUNT_OPTIONS` in your `.env`.
3. Scrape Prometheus metrics at `http://<host>:5572/metrics`.

---

## Unmounting / Stopping

To cleanly stop and unmount:
```bash
docker compose down
```

If the mount becomes unresponsive or busy on the host:
```bash
fusermount -u /srv/data/Development
# or
umount -l /srv/data/Development
```
