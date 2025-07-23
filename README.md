# Graylog Docker Stack (with Tailscale & Systemd)

This project bootstraps a secure, persistent Graylog stack using Docker Compose, tailored for headless or homelab environments. It supports automated deployment and systemd integration.

---

## 🔧 Components

- **Graylog**: Centralized log management
- **MongoDB**: Metadata storage
- **OpenSearch**: Log indexing and search
- **Tailscale**: Secure remote access over your tailnet

---

## 🚀 Deployment Steps

1. Provision your Debian 12 server
2. Run the Ansible playbook to install Docker and Tailscale
3. Mount and prepare persistent disk at `/srv/docker-data`
4. Place your `docker-compose.yml` and `.env` file in `/srv/docker-data`
5. Run:

```bash
docker compose up -d
```

---

## 🔐 .env File Security

Your `.env` file contains **secrets** needed to launch the Graylog container:

```env
GRAYLOG_PASSWORD_SECRET=...
GRAYLOG_ROOT_PASSWORD_SHA2=...
GRAYLOG_EXTERNAL_URI=...
```

### ✅ Keep it Secure:

- Do **not** check it into version control (`.gitignore` should include `.env`)
- Set restrictive permissions:
  ```bash
  chmod 600 /srv/docker-data/.env
  ```
- Store backups securely and rotate secrets periodically

---

## 🔁 Auto-Start with systemd

To ensure the stack survives reboots and reinitializes automatically, use a systemd unit:

### ✅ Example: `/etc/systemd/system/graylog-docker.service`

```ini
[Unit]
Description=Graylog Docker Compose Stack
Requires=network-online.target
After=network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/srv/docker-data
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

### Enable It:

```bash
sudo systemctl daemon-reexec
sudo systemctl enable --now graylog-docker.service
```

---

## 📎 Useful Commands

- View logs: `docker compose logs -f`
- Restart stack: `docker compose restart`
- Tear down: `docker compose down`

---

## 🧪 Verification

- Visit `http://<tailscale-ip>:9000`
- Login with root and your hashed password
- Confirm logs are arriving via GELF or syslog input

---

## 🧰 Maintenance Tips

- Monitor disk usage in `/srv/docker-data`
- Regularly update container images with:
  ```bash
  docker compose pull && docker compose up -d
  ```
- Consider Watchtower or systemd timers for automatic updates

---
