# 🚀 ImeCo Server Deployment Guide

Concise guide for deploying ImeCo infrastructure from scratch to a new VPS.

---

## 1. Local Preparation
Perform these steps from your workspace root (`imeco/`):

*   **Personal SSH Key**: Ensure your personal key (`~/.ssh/general/id_ed25519_general`) is ready for the `victor` user.
*   **GitHub PAT**: Generate a **Fine-grained Personal Access Token** on GitHub:
    - Settings -> Developer Settings -> Fine-grained tokens.
    - Repository access: Select `imeco-infra` and all submodule repos.
    - Permissions: `Contents: Read-only`.
    - Add the token to `inventories/group_vars/all/vault.yml` as `github_token`.
*   **SSL Certs**: Move your original `.crt` and `.key` files to the workspace's SSL folder:
    `ssl/globalindosimetrika.co.id/`
    `ssl/api.anugrahtravel.com/`
    `ssl/radar-ms.com/`

---

## 2. Ansible Setup & Secrets
*All following commands must be run from the `imeco-server/ansible/` directory:*

```bash
cd imeco-server/ansible
```

1.  **Vault Password**: Create a `.vault_pass` file in this folder and enter your password.
2.  **Edit Vault**: Open `inventories/group_vars/all/vault.yml` and fill in all credentials.
3.  **Encrypt**: Run `ansible-vault encrypt inventories/group_vars/all/vault.yml`.

---

## 3. Execution (from `ansible/` folder)

### Initial Stage (Run Once)
Use the **Bootstrap Profile** to set up the `deploy` and `victor` users on the server:
```bash
ansible-playbook playbooks/create-user.yml -i inventories/bootstrap.yml
```
*Note: This profile uses your personal key (`~/.ssh/general/id_ed25519_general`) to gain initial access.*

### Full Setup (Infrastructure & Apps)
Once users are created, run the full setup using the default **Production Profile**:
```bash
ansible-playbook playbooks/setup.yml
```
*Note: This automatically uses `inventories/production.yml` and your dedicated `deploy` key.*

### Update Code (Redeploy)
To update app source code without touching infrastructure:
```bash
ansible-playbook playbooks/deploy-apps.yml
```

---

## 4. Verification
1.  **Login**: `ssh victor@103.126.116.80`
2.  **Status**: `sudo docker ps` (Ensure 6-8 containers are running).
3.  **Links**:
    *   https://globalindosimetrika.co.id
    *   https://radar-ms.com
    *   https://api.anugrahtravel.com
    *   http://103.126.116.80:8080 (phpMyAdmin)

---

> [!TIP]
> Always append `--check` at the end of commands for a simulation before actual execution.
