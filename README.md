# Homelab Rclone Mount

A lightweight, containerized solution to mount cloud storage remotes (such as Seafile WebDAV, Google Drive, Nextcloud, S3, OneDrive, etc.) to a local host directory using [Rclone](https://rclone.org/) and FUSE.

---

## Features

- **Generic & Configurable:** Mount any Rclone remote path to any host directory via `.env`.
- **FUSE Mount with Shared Propagation (`rshared`):** Makes mounted files accessible to the host system and other Docker containers.
- **Base64 Config Injection:** Easily inject your entire `rclone.conf` without worrying about multi-line `.env` parsing issues or mounting extra files into the container.
- **Optimized VFS Cache:** Uses `--vfs-cache-mode full` for high compatibility with read/write applications.

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
| `TZ` | Timezone for container logs | `UTC` |
| `RCLONE_REMOTE_PATH` | Rclone remote and subpath to mount | `seafile:Development` |
| `HOST_MOUNT_DIR` | Directory on the host where files will be mounted | `/srv/data/Development` |
| `CONTAINER_NAME` | Container name (optional) | `rclone-mount` |
| `RCLONE_MOUNT_OPTIONS` | Extra mount flags (optional) | `--vfs-cache-mode full --allow-other --allow-non-empty --buffer-size 32M` |
| `RCLONE_CONF_BASE64` | Base64-encoded `rclone.conf` | Single-line base64 string |

### 3. Generate Base64 Rclone Config
Generate the Base64 string from your existing `rclone.conf`:

- **Linux:**
  ```bash
  cat ~/.config/rclone/rclone.conf | base64 -w 0
  ```
- **macOS:**
  ```bash
  cat ~/.config/rclone/rclone.conf | base64
  ```

Paste the resulting string into `RCLONE_CONF_BASE64` in `.env`.

### 4. Start the Mount
```bash
docker compose up -d
```

### 5. Check Logs
```bash
docker compose logs -f
```

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
