# 🐳 Graylog Open Core Deployment

This project automates the deployment of a production-ready [Graylog Open Core](https://www.graylog.org/products/open-core) stack using Docker, systemd, and Ansible. It provisions persistent storage, secures permissions, configures unattended system updates, and simplifies service management.

---

## 📁 Contents

| File                      | Purpose                                                      |
|---------------------------|--------------------------------------------------------------|
| `docker-compose.yml`      | Defines the Graylog stack (Graylog, DataNode, MongoDB)       |
| `prepare-disk.yml`        | Prepares and mounts data disk, sets ACLs, configures sysctl  |
| `bootstrap.yml`           | Creates `ansibleadmin` user with SSH key and sudo access     |
| `graylog-docker.service`  | systemd unit to manage Graylog stack as a background service |
| `graylog_ufw_rules.yml`   | Configures firewall rules (UFW) for Graylog ports            |
| `requirements.yml`        | External Ansible role dependencies                           |
| `.gitignore`              | Ignores runtime files and artifacts                          |
| `README.md`               | You're here 👋                                               |

---

## ⚙️ Prerequisites

- Ubuntu 22.04 or Debian 12+
- Disk device for Docker data (e.g. `/dev/nvme0n1`)
- Ansible installed locally
- SSH access as root or another provisioning user

---

## 🚀 Deployment Overview

### 1. Clone and configure

```bash
git clone https://github.com/jimrippee/infra-docker.git
cd infra-docker/graylog
```

### 2. Optional: Create `ansibleadmin` user

```bash
ansible-playbook bootstrap.yml -u root -i inventory.ini \
  --extra-vars "ansibleadmin_pubkey='ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFSsEngu8whaTye0S5zRhHbuNbJQv4DY7I8qxU5cyes2'"
```

This creates a passwordless `ansibleadmin` user (UID 1100), installs your SSH key, and enables passwordless sudo access.

---

### 3. Prepare Docker Data Disk

```bash
ansible-playbook prepare-disk.yml -u root -i inventory.ini
```

This will:

- Format and mount `/dev/nvme0n1` to `/srv/docker-data`
- Configure `vm.max_map_count=262144`
- Create `/srv/docker-data/graylog` owned by `ansibleadmin` (UID 1100)
- Enable unattended upgrades with:
  - Security + kernel updates
  - Reboots every **Sunday at 3:00 AM**

---

### 4. Launch Stack

```bash
docker compose up -d
```

OR run as a background service:

```bash
cp graylog-docker.service /etc/systemd/system/
systemctl daemon-reexec
systemctl enable graylog-docker
systemctl start graylog-docker
```

---

### 5. Firewall Rules (Optional)

```bash
ansible-playbook graylog_ufw_rules.yml -u root -i inventory.ini
```

This allows ports such as:

- 9000 (Web UI)
- 5044 (Beats)
- 5140 (Syslog TCP/UDP)
- 12201 (GELF TCP/UDP)
- 13301–13302 (Forwarder)

---

## 🔐 Environment Variables

You must define these in a `.env` file:

```dotenv
GRAYLOG_PASSWORD_SECRET=your-random-secret
GRAYLOG_ROOT_PASSWORD_SHA2=your-sha256-hash
```

To generate a SHA256 password hash:

```bash
echo -n 'YourPasswordHere' | sha256sum
```

---

## 📍 Graylog Access

After launch, visit:

```
http://<your-server-ip>:9000
```

---

## 🛡 System Security

This deployment includes:

- ACL-controlled persistent data directory (`/srv/docker-data/graylog`)
- Hardened file permissions for Docker volumes
- `vm.max_map_count` for Elasticsearch compatibility
- `unattended-upgrades` with controlled reboots
- UFW rules scoped to essential Graylog ports

---

## 🔧 To Do

- Add TLS reverse proxy (Caddy or Traefik)
- Tailscale-based remote logging
- Graylog + Vector + Wazuh integration
- Automated snapshot backups

---

## 👤 Author

- [@jimrippee](https://github.com/jimrippee)

---

## 📝 License

MIT — Use at your own risk. Contributions welcome!
