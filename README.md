# ImeCo Server Infrastructure

This repository contains the Ansible playbooks and Docker configuration required to provision and automate the deployment of the ImeCo server infrastructure, including the Nginx proxy, MySQL databases, and applications (RMS, Anugrah Travel API, and Globalindo).

---

## 📁 Directory Structure

*   `ansible/ansible.cfg`: Main Ansible configuration file.
*   `ansible/inventories/`: Server definitions and variables.
    *   `production.yml`: IP address and connection details for the production server.
    *   `group_vars/all/vars.yml`: Public configuration variables (deploy paths, etc.).
    *   `group_vars/all/vault.yml`: **Encrypted** file containing sensitive secrets (passwords, SSL certs, tokens).
*   `ansible/playbooks/`: Playbooks to execute different deployment scenarios.
    *   `setup.yml`: Full infrastructure setup (Docker, Network, DB, Nginx, Apps).
    *   `deploy-apps.yml`: Quick redeployment for just the applications (no core infra changes).
*   `ansible/roles/`: Modular tasks for each component (common, security, docker, mysql, nginx, backup, and individual apps).

---

## 🔧 Prerequisites (Before Running Ansible)

Before running Ansible for the very first time on a new server, prepare your local environment:

1.  **Personal SSH Key:** Ensure your personal SSH key (e.g., `~/.ssh/id_ed25519`) is available locally. This will be used to grant you admin access to the server.
2.  **GitHub PAT:** Generate a **Fine-grained Personal Access Token** on GitHub:
    *   Settings -> Developer Settings -> Fine-grained tokens.
    *   Repository access: Select `imeco-infra` and all application repos.
    *   Permissions: `Contents: Read-only`.
    *   Add this token to `ansible/inventories/group_vars/all/vault.yml` as `github_token`.
3.  **SSL Certs:** Place your original `.crt` and `.key` files into the workspace's SSL folder at `ansible/roles/nginx/files/ssl/<DOMAIN>/`.
4.  **Vault Password:** Create a file named `.vault_pass` in the `ansible/` folder and enter a secure password. **Do not commit this file.**

---

## 🔒 Phase 0: Setup Ansible Vault

We use Ansible Vault to securely store secrets like MySQL passwords and API tokens.

**1. Configure your Secrets**
Edit `ansible/inventories/group_vars/all/vault.yml`. Replace all placeholders with your actual secrets.

**2. Encrypt the Vault File**
```bash
cd ansible
ansible-vault encrypt inventories/group_vars/all/vault.yml
```
*(To edit later: `ansible-vault edit inventories/group_vars/all/vault.yml`)*

---

## 🚀 Complete Guide: New Server Initialization 

Follow these steps in order to set up a brand new VPS from scratch.

### Step 1: Initial Security & User Setup (Bootstrap)
This secures the `root` account and creates dedicated deployment users.
1. SSH into the server as `root` to verify access.
2. Run the bootstrap playbook:
   ```bash
   cd ansible
   ansible-playbook -i inventories/bootstrap.yml playbooks/create-user.yml --ask-pass
   ```
   *Note: Use `--ask-pass` if your server uses password authentication for root initially.*

### Step 2: Deploy Infrastructure
This installs Docker, Nginx, MySQL, and starts all application containers.
```bash
cd ansible
ansible-playbook -i inventories/production.yml playbooks/setup.yml
```

### Step 3: Migration (SQL & Assets)
Your containers are now running, but the database and upload folders are empty.

**A. Import SQL Databases**
1. Visit `http://<SERVER_IP>:8080` (phpMyAdmin).
2. Log in with user `root` and your `vault_mysql_root_password`.
3. Import your `.sql` backups into the respective databases (e.g., `db_rms`).

**B. Migrate User Assets (public/uploads)**
Manually copy your existing assets to the server using `rsync` or any SFTP tool (e.g., FileZilla):
```bash
# Example for Radar Mandiri Sukses
rsync -avz /path/to/local/public/uploads/ <USER>@<SERVER_IP>:/opt/imeco-infra/apps/radar-mandiri-sukses/public/uploads/
```

### Step 4: Final Hardening
Once the migration is complete, close the phpMyAdmin port for security.
1. In `ansible/roles/security/tasks/main.yml`, remove or comment out the rule for port `8080`.
2. Apply the security changes:
   ```bash
   cd ansible
   ansible-playbook -i inventories/production.yml playbooks/setup.yml --tags security
   ```

---

## 🛠️ Daily Operations Cheatsheet

Once the server is running, use these shortcuts from the `ansible/` folder for routine updates:

*   **Update Code Only (Fast):**
    ```bash
    ansible-playbook -i inventories/production.yml playbooks/deploy-apps.yml
    ```
*   **Update Nginx Configs:**
    ```bash
    ansible-playbook -i inventories/production.yml playbooks/setup.yml --tags nginx
    ```
*   **Restart Specific Services:**
    ```bash
    ansible-playbook -i inventories/production.yml playbooks/setup.yml --tags "mysql,pma"
    ```

---

## ⚠️ Notes on Private Repositories

The deployment uses `git clone` on the target server to pull your source code. 

If any of your repositories or submodules are **private**, the deployment will generate an SSH key at `/home/<USER>/.ssh/id_ed25519.pub`. Add this key to your GitHub **Deploy Keys** for the specific repository to grant the server read access.

