# Deploy Steps (Concise)

## 1. New Server (one-time initialization)

1. Run user bootstrap and SSH hardening:
```bash
cd ansible
ansible-playbook -i inventories/bootstrap.yml playbooks/create-user.yml --ask-pass
```

2. Run full infrastructure provisioning (Docker, network, MySQL, Nginx, initial app stack):
```bash
ansible-playbook -i inventories/production.yml playbooks/setup.yml --vault-password-file .vault_pass
```

3. Run manual data migration:
```bash
# Import SQL via phpMyAdmin or mysql client
# Copy upload assets (RMS example)
rsync -avz /path/local/uploads/ <USER>@<SERVER_IP>:/opt/imeco-infra/apps/radar-mandiri-sukses/public/uploads/
```

## 2. Daily Deploy (routine app updates)

1. Deploy Globalindo website only:
```bash
cd ansible
ansible-playbook -i inventories/production.yml playbooks/deploy-apps.yml --tags globalindo --vault-password-file .vault_pass
```

2. Deploy Travel API only:
```bash
ansible-playbook -i inventories/production.yml playbooks/deploy-apps.yml --tags travel --vault-password-file .vault_pass
```

3. Deploy RMS only:
```bash
ansible-playbook -i inventories/production.yml playbooks/deploy-apps.yml --tags rms --vault-password-file .vault_pass
```

4. Deploy all apps listed in `deploy-apps.yml`:
```bash
ansible-playbook -i inventories/production.yml playbooks/deploy-apps.yml --vault-password-file .vault_pass
```

## 3. Infra Only (when changing infra services, not app code)

1. Apply/update Nginx only:
```bash
cd ansible
ansible-playbook -i inventories/production.yml playbooks/setup.yml --tags nginx --vault-password-file .vault_pass
```

2. Apply/update MySQL + phpMyAdmin only:
```bash
ansible-playbook -i inventories/production.yml playbooks/setup.yml --tags "mysql,pma" --vault-password-file .vault_pass
```

## 4. Quick Rule

1. Use `setup.yml` for infrastructure provisioning/updates.
2. Use `deploy-apps.yml` for routine app code deploys.
