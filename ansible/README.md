# ImeCo Server - Ansible Automation

This directory contains the Ansible playbooks and roles for automating the deployment of the ImeCo Server infrastructure.

## 📁 Directory Structure

*   `ansible.cfg`: Main Ansible configuration file.
*   `inventory/`: Server definitions and variables.
    *   `production.yml`: IP address and connection details for the production server.
    *   `group_vars/all/vars.yml`: Public configuration variables (deploy paths, etc.).
    *   `group_vars/all/vault.yml`: **Encrypted** file containing sensitive secrets (passwords, SSL certs, tokens).
*   `playbooks/`: Playbooks to execute different deployment scenarios.
    *   `site.yml`: Full infrastructure setup (Docker, Network, DB, Nginx, Apps).
    *   `deploy-apps.yml`: Quick redeployment for just the applications (no core infra changes).
*   `roles/`: Modular tasks for each component (common, security, docker, mysql, nginx, backup, and individual apps).

---

## 🔒 Setup: Ansible Vault (First Time Only)

We use Ansible Vault to securely store secrets like MySQL passwords, Rclone tokens, and SSL certificates.

**1. Create the Vault Password File**
Create a file named `.vault_pass` in the project root. This file is ignored by Git.
```bash
echo "your-secure-vault-password" > .vault_pass
```

**2. Configure your Secrets**
Edit the file at `inventory/group_vars/all/vault.yml`. Replace all the `REPLACE_ME` placeholders with your actual secrets.

**3. Encrypt the Vault File**
Once your secrets are added, encrypt the file so it is safe to commit to version control:
```bash
ansible-vault encrypt ansible/inventory/group_vars/all/vault.yml
```

*(Note: If you ever need to edit your secrets again, use: `ansible-vault edit ansible/inventory/group_vars/all/vault.yml`)*

---

## 🚀 How to Deploy

Ensure you have your SSH key set up to connect to the target server (`103.126.116.80`) correctly as documented in [DEPLOY_STEPS.md](file:///home/victor/projects/imeco/imeco-server/ansible/DEPLOY_STEPS.md).

### Scenario 0: Initial Server Setup (Create your users)
If you have a brand new server that only has a `root` user (or an existing user like `victor`), you must create/configure the `deploy` and `victor` users first. We have a playbook for this!
Run this command *once* to create your users, copy your SSH keys, and harden SSH security:
```bash
ansible-playbook playbooks/create-user.yml -u root
# OR
ansible-playbook playbooks/create-user.yml -u victor
```
Once this succeeds, your `deploy` user is ready for all future automation!

### Scenario A: Full Infrastructure Setup
This installs all dependencies (Docker, MySQL, Nginx, etc.) and boots up all apps and services as the `deploy` user.
```bash
ansible-playbook playbooks/setup.yml
```

### Scenario B: Quick Application Redeployment
If you only pushed changes to your application code (RMS, Travel API, or Globalindo Simetrika) and want to restart them:
```bash
ansible-playbook playbooks/deploy-apps.yml
```

---

## ⚠️ Notes on Private Git Repositories

The deployment uses `git clone` on the target server to pull your source code. 

If any of your repositories or submodules (like `globalindo-simetrika-website`) are **private**, the deployment will generate an SSH key on the target server at `/root/.ssh/id_ed25519.pub`. 

If the `git clone` task fails during deployment, it will print this public SSH key to your terminal. You must copy that key and add it to your GitHub repository's **Deploy Keys** to grant the server read access.
