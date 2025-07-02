# Ansible DevOps Demo

Automates provisioning and deployment of a two-tier demo stack — **PostgreSQL 16** and a **Spring Boot** application — across Linux hosts with Ansible.  
Choose between running the app as a native systemd service, via **Docker Compose**, or updating an existing **Kubernetes** deployment image.

---

## Repository Layout

```
ansible.cfg               # Controller defaults
hosts.yaml                # Example static inventory
host_vars/                # Per-host variables
files/
  spring.service.j2       # Systemd service template
playbooks/
  docker.yaml                     # Install & configure Docker Engine
  postgres.yaml                   # Install / configure PostgreSQL 16
  spring.yaml                     # Build & run Spring Boot under systemd
  spring-docker.yaml              # Build & run app with Docker Compose
  k8s-update-spring-deployment.yaml # Roll out new image to Kubernetes
  hostvars_and_facts.yml          # Debug helper
```

---

## Prerequisites

* **Control node**: Python 3, Ansible ≥ 2.16
* **Managed hosts**: Ubuntu 24.04 (tested) with SSH access
* **Optional**
  * Internet access to Docker & Maven repositories (for Docker / build)
  * Kubernetes cluster and `kubectl` configured (for K8s rollout)

---

## Quick Start

1. **Inventory** — copy `hosts.yaml` and adjust hostnames / IPs.

2. **Check connectivity**
   ```bash
   ansible -m ping all
   ```

3. **Provision database (runs on postgres-16)**
   ```bash
   ansible-playbook -l dbservers playbooks/postgres.yaml
   ```

4. **Deploy application**

   * **Systemd**
     ```bash
     ansible-playbook -l appservers playbooks/spring.yaml
     ```

   * **Docker Compose** (installs Docker if missing)
     ```bash
     ansible-playbook -l appservers playbooks/spring-docker.yaml
     ```

   * **Kubernetes** image update (runs only on the controller)
     ```bash
     ansible-playbook playbooks/k8s-update-spring-deployment.yaml
     ```

---

## Variable Overview

Key host-specific settings live in `host_vars/*`.

```yaml
# host_vars/appserver-vm.yaml
appdir: "~/spring"
branch: "main"
app_port: 8080
app:
  env:
    spring.datasource.url: jdbc:postgresql://dbserver:5432/homesharing
```

---

## Using Ansible Vault

```bash
# create & encrypt secret file
ansible-vault encrypt playbooks/vars/api_key.yml

# run playbook that needs it
ansible-playbook --ask-vault-pass playbooks/…
```
Store a password in `~/.ansible/vault_pass.txt` and add `--vault-password-file` to automate decryption.

---

## Useful Commands

| Task | Command |
|------|---------|
| Gather host facts | `ansible-playbook -l <host> playbooks/hostvars_and_facts.yml` |
| Ignore host-key checking | set `host_key_checking = false` in **ansible.cfg** |
| Install PostgreSQL role from Galaxy | `ansible-galaxy install geerlingguy.postgresql` |