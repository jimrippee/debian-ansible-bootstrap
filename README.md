# Debian Server Bootstrap with Ansible

This project automates the initial provisioning of a Debian-based server using Ansible. It installs core packages, configures the timezone, joins the server to your Tailscale tailnet, installs Docker using the `geerlingguy.docker` role, and enables basic firewall rules with UFW.

## 🛠️ Features

- Set timezone to UTC
- Full system update and dist-upgrade
- Docker installation via `geerlingguy.docker` role
- Tailscale installation and authentication
- UFW configuration to allow SSH and deny all other traffic
- Optional reboot after upgrade
- Ready for Graylog container deployment or similar apps

## 📁 Files

- `bootstrap.yml`: Main playbook
- `inventory.ini`: Define your target hosts
- `requirements.yml`: Role dependency for Docker (Geerlingguy)

## 🚀 Getting Started

### 1. Install Dependencies

```bash
brew install ansible  # or use apt/pip if on Linux
```

### 2. Install Required Roles

```bash
ansible-galaxy install -r requirements.yml
```

### 3. Run the Playbook

```bash
ansible-playbook -i inventory.ini bootstrap.yml --ask-become-pass --extra-vars "tailscale_authkey=tskey-xxxxxxxxxxxxxxxx"
```

You will be prompted for the `ansibleadmin` sudo password unless passwordless sudo is configured.

## 🔐 SSH Key Setup

To prepare the server for Ansible management:

```bash
ssh-copy-id ansibleadmin@your.server.ip
```

This enables Ansible to connect using your existing SSH key.

## 📦 Managing Secrets

Use `--extra-vars` to pass your Tailscale auth key inline or consider using Ansible Vault to encrypt secrets.

ansible-playbook -i inventory.ini bootstrap.yml --ask-become-pass --extra-vars "tailscale_authkey=tskey-abc123def456ghi789"


## ⏲️ Timezone

The system will be configured to use **UTC**. You can adjust this in the playbook if needed.

## 👥 Contributors

- You

## 📄 License

MIT
