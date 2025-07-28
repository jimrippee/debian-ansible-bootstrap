# Plex Media Server (Docker)

This directory contains the Docker configuration for a self-hosted Plex Media Server with:

- Hardware-accelerated transcoding via Intel Quick Sync
- RAM-disk (`tmpfs`) for high-speed transcoding
- Network TV tuner support (HDHomeRun)
- Persistent Plex data directory for library and metadata
- External media mounts for Movies and TV

---

## 📦 Container Overview

- **Image:** `linuxserver/plex:1.41.9`
- **Container Name:** `plex`
- **User Mapping:** UID=1100 / GID=1100 (maps to `ansibleadmin`)
- **Transcode Location:** `/transcode` mounted as `tmpfs` (4GB)
- **Data Directory:** `/srv/docker-data/plex/data`
- **Media Mounts:**
  - `/plex/movies` → `/dev/sdb1`
  - `/plex/tv`     → `/dev/sdb2`

---

## 📁 Folder Structure

```text
plex/
├── docker-compose.yml
├── README.md         ← You are here
└── data/              ← Plex Library (Preferences.xml, Metadata, etc.)
```

---

## 🔧 Runtime Features

### ✔️ Hardware Transcoding

Intel Quick Sync Video (iGPU) is enabled by passing `/dev/dri` into the container and verifying with:

```bash
docker exec plex vainfo --display drm
docker logs plex | grep -i hardware
```

### ✔️ RAM Disk for Transcoding

`/transcode` is mounted as a 4GB tmpfs inside the container with:

```yaml
tmpfs:
  - /transcode:size=4g,mode=1777
```

Confirm with:

```bash
docker exec plex mount | grep /transcode
```

---

## 📺 HDHomeRun Support

- Network-based tuner support is enabled out-of-box
- Ensure Plex can reach the HDHomeRun device over the LAN
- No additional ports are required to be opened on the host

---

## 🧠 Data Migration Notes

To migrate an existing Plex library:

```bash
rsync -aAXv -e ssh \
  "plex@oldhost:/var/lib/plexmediaserver/Library/Application Support/Plex Media Server/" \
  /srv/docker-data/plex/data/Library/Application\ Support/Plex\ Media\ Server/
```

Ensure proper ownership:

```bash
sudo chown -R ansibleadmin:ansibleadmin /srv/docker-data/plex/data
```

---

## ▶️ Running the Container

Start the container with:

```bash
docker-compose up -d
```

Check logs:

```bash
docker logs -f plex
```

---

## 🔒 Permissions

All container volumes are mapped to user/group ID 1100. If this is changed, update `PUID` and `PGID` in `docker-compose.yml`.

---

## 📌 Dependencies

- Docker Engine
- Intel VAAPI-compatible iGPU
- `intel-media-va-driver-non-free`, `vainfo`, `intel-gpu-tools` on host (optional for testing)
