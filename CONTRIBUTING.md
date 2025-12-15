# 📚 Contributing & Technical Documentation

Welcome to the **Infra Automation Challenge**! This document provides comprehensive technical details about the project architecture, how it works, and how to contribute.

---

## 📖 Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [How to Get Started](#how-to-get-started)
4. [Project Structure](#project-structure)
5. [Ansible Workflow](#ansible-workflow)
6. [Contributing Guidelines](#contributing-guidelines)
7. [Troubleshooting](#troubleshooting)

---

## 📖 Project Overview

This project automates the complete bootstrap and deployment of a **containerized web application** on a fresh Ubuntu server.

### What It Does

1. **System Setup** → Updates packages, installs dependencies
2. **User Management** → Creates a `deploy` user with sudo access
3. **Docker Installation** → Sets up Docker daemon with JSON logging
4. **Log Rotation** → Configures fail2ban and Docker log rotation
5. **Application Deployment** → Builds and runs a containerized app
6. **Idempotency** → Safe to re-run without breaking anything

### Use Cases

- Multi-environment deployments (dev, staging, production)
- Infrastructure-as-Code (IaC) for reproducible builds
- Scaling: deploy the same app to dozens of servers
- Disaster recovery: quickly rebuild failed servers

---

## 🏗️ Architecture

### High-Level Flow

```
┌─────────────────────────────────────────────────────────────┐
│                  Your Local Machine                         │
│  (Ansible + SSH key to remote server)                       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ SSH (port 22)
                     │
┌────────────────────▼────────────────────────────────────────┐
│         Fresh Ubuntu 24.04 Server (Cloud)                   │
│                                                              │
│  1. Update packages (APT)                                    │
│  2. Create deploy user                                       │
│  3. Install Docker                                           │
│  4. Copy app files                                           │
│  5. Build Docker image                                       │
│  6. Run container (Nginx on port 80)                        │
│                                                              │
│  ┌──────────────────────────────┐                           │
│  │   Docker Container (Nginx)   │                           │
│  │   - Listening on port 80     │                           │
│  │   - Serving index.html       │                           │
│  └──────────────────────────────┘                           │
└────────────────────────────────────────────────────────────┘
```

---

## 🔧 How to Get Started

### Step 1: Clone the Repository

```bash
git clone https://github.com/prncnano2000/infra-automation-challenge.git
cd infra-automation-challenge
```

### Step 2: Add Your SSH Key

Place your **private SSH key** at the project root:

```bash
cp ~/.ssh/your-key.pem ./id_rsa.pem
chmod 600 id_rsa.pem
```

**DO NOT commit this file to git!** Add to `.gitignore`:

```bash
echo "id_rsa.pem" >> .gitignore
```

### Step 3: Configure Server IPs

Edit `ansible/inventory.ini` and add your server(s):

```ini
[all]
your.server.ip.here ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem

[app_servers]
your.server.ip.here
```

**Example:**

```ini
[all]
18.156.129.16 ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem
192.168.1.100 ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem

[app_servers]
18.156.129.16
192.168.1.100
```

### Step 4: Run the Playbook

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/site.yml
```

That's it! ✅

---

## 📁 Project Structure

```
infra-automation-challenge/
│
├── README.md                        # Project overview & assignment details
├── QUICK_START.md                   # Quick setup and deployment guide
├── CONTRIBUTING.md                  # This file (technical documentation)
├── CODE_OF_CONDUCT.md               # Community guidelines
├── LICENSE                          # Project license
├── SECURITY.md                      # Security best practices
├── SUPPORT.md                       # Getting help
│
├── id_rsa.pem                       # ⚠️ YOUR SSH PRIVATE KEY (add to .gitignore)
├── ansible.cfg                      # Ansible configuration (role path, etc.)
│
├── ansible/
│   │
│   ├── inventory.ini                # Server inventory with IPs and SSH settings
│   │
│   ├── playbooks/
│   │   └── site.yml                 # Main playbook that orchestrates everything
│   │
│   ├── roles/
│   │   │
│   │   ├── common/
│   │   │   └── tasks/
│   │   │       └── main.yml         # System setup, user creation, sudo config
│   │   │
│   │   ├── docker/
│   │   │   └── tasks/
│   │   │       └── main.yml         # Docker installation & daemon setup
│   │   │
│   │   ├── logging/
│   │   │   ├── tasks/
│   │   │   │   └── main.yml         # Log rotation configuration
│   │   │   ├── handlers/
│   │   │   │   └── main.yml         # Docker service restart handler
│   │   │   └── templates/
│   │   │       └── fail2ban.logrotate.j2  # Log rotation template
│   │   │
│   │   └── app/
│   │       └── tasks/
│   │           └── main.yml         # Copy app, build image, run container
│   │
│   └── vars/
│       └── main.yml                 # Global variables (used by all roles)
│
└── docker/
    └── app/
        ├── Dockerfile               # Nginx-based Docker image
        └── index.html               # Sample HTML page
```

---

## 🔄 Ansible Workflow

### 1. Inventory (`ansible/inventory.ini`)

Defines which servers to deploy to and how to connect:

```ini
[all]
18.156.129.16 ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem
```

- `18.156.129.16` → Server IP address
- `ansible_user=ubuntu` → SSH username
- `ansible_ssh_private_key_file=./id_rsa.pem` → Path to your private key

### 2. Playbook (`ansible/playbooks/site.yml`)

The main orchestration file that runs roles in order:

```yaml
---
- name: Provisionnement et déploiement serveur
  hosts: app_servers
  become: yes  # Run with sudo privileges

  roles:
    - role: common   # Step 1: System setup
    - role: docker   # Step 2: Docker installation
    - role: logging  # Step 3: Log rotation configuration
    - role: app      # Step 4: Application deployment
```

### 3. Roles (Reusable Task Blocks)

Each role handles one responsibility:

#### **Role: `common`**

Prepares the server for deployment:

```yaml
---
- name: Mise à jour du cache APT
  apt:
    update_cache: yes

- name: Installation des paquets requis
  apt:
    name:
      - curl
      - wget
      - git
    state: present

- name: Création de l'utilisateur deploy
  user:
    name: deploy
    state: present

- name: Autoriser sudo sans mot de passe
  lineinfile:
    path: /etc/sudoers.d/deploy
    line: "deploy ALL=(ALL) NOPASSWD:ALL"
    create: yes
```

**What it does:**
- Updates package lists
- Installs required tools
- Creates a `deploy` user
- Grants sudo access without password

#### **Role: `docker`**

Installs and starts Docker:

```yaml
---
- name: Installer Docker
  apt:
    name: docker.io
    state: present

- name: Activer Docker
  systemd:
    name: docker
    state: started
    enabled: yes
```

**What it does:**
- Installs `docker.io` package
- Starts the Docker daemon
- Enables Docker to auto-start on reboot

#### **Role: `logging`**

Configures log rotation for fail2ban and Docker:

```yaml
---
- name: Configurer la rotation des logs fail2ban
  template:
    src: fail2ban.logrotate.j2
    dest: /etc/logrotate.d/fail2ban
    mode: '0644'

- name: Configurer les logs Docker
  copy:
    dest: /etc/docker/daemon.json
    content: |
      {
        "log-driver": "json-file",
        "log-opts": {
          "max-size": "10m",
          "max-file": "3"
        }
      }
    mode: '0644'
  notify: restart docker
```

**Handlers** (`handlers/main.yml`):

```yaml
---
- name: restart docker
  service:
    name: docker
    state: restarted
```

**Log Rotation Template** (`templates/fail2ban.logrotate.j2`):

```jinja
/var/log/fail2ban.log {
    weekly
    rotate 4
    missingok
    notifempty
    compress
    delaycompress
    postrotate
        systemctl reload fail2ban > /dev/null 2>&1 || true
    endscript
}
```

**What it does:**
- Configures fail2ban logs to rotate weekly, keeping 4 rotations
- Sets Docker to use JSON file logging driver with size limits
- Limits Docker logs to max 10MB per file and max 3 files
- Automatically restarts Docker when log configuration changes
- Ensures logs don't consume disk space over time

#### **Role: `app`**

Deploys the containerized application:

```yaml
---
- name: Copier l'application Docker
  copy:
    src: ../../docker/app/
    dest: /opt/app

- name: Construire l'image Docker
  docker_image:
    name: demo_app
    tag: latest
    source: build
    build:
      path: /opt/app
    state: present

- name: Lancer le conteneur applicatif
  docker_container:
    name: demo_app
    image: demo_app:latest
    state: started
    restart_policy: always
    ports:
      - "80:80"
```

**What it does:**
- Copies `docker/app/` to the server
- Builds a Docker image from the Dockerfile
- Runs the container on port 80

### 4. Ansible Configuration (`ansible.cfg`)

```ini
[defaults]
roles_path = ./ansible/roles
inventory = ./ansible/inventory.ini
host_key_checking = False
```

**Why this matters:**
- `roles_path` → Tells Ansible where to find roles
- `host_key_checking = False` → Trusts SSH host keys automatically

### 5. Global Variables (`ansible/vars/main.yml`)

Centralized configuration used across all roles:

```yaml
---
packages:
  - curl
  - wget
  - git
  - apt-transport-https
  - ca-certificates
  - gnupg
  - lsb-release

docker_image: demo_app:latest
app_port: 80
```

---

## 🔀 Contributing Guidelines

### 1. **Fork & Clone**

```bash
git clone https://github.com/YOUR_USERNAME/infra-automation-challenge.git
cd infra-automation-challenge
```

### 2. **Create a Feature Branch**

```bash
git checkout -b feature/my-improvement
```

### 3. **Make Changes**

- Add new roles in `ansible/roles/`
- Enhance existing tasks
- Improve documentation
- Fix bugs or security issues

### 4. **Test Locally**

```bash
# Lint your YAML
ansible-playbook --syntax-check ansible/playbooks/site.yml

# Dry run (preview changes)
ansible-playbook -i ansible/inventory.ini ansible/playbooks/site.yml --check

# Full deployment
ansible-playbook -i ansible/inventory.ini ansible/playbooks/site.yml
```

### 5. **Commit & Push**

```bash
git add .
git commit -m "Add feature: describe your change"
git push origin feature/my-improvement
```

### 6. **Open a Pull Request**

- Fill out the PR template
- Link any related issues
- Request reviews from 2+ community members

### Review Criteria

PRs will be reviewed for:

- ✅ **Correctness** — Does it work as intended?
- ✅ **Idempotency** — Safe to run multiple times?
- ✅ **Security** — No hardcoded secrets, proper permissions?
- ✅ **Readability** — Clear, well-documented code
- ✅ **Best Practices** — Follows Ansible conventions

---

## 🆘 Troubleshooting

### Problem: `ansible: command not found`

**Solution:** Install Ansible

```bash
pip install ansible
# or
brew install ansible  # macOS
sudo apt install ansible  # Ubuntu
```

### Problem: `SSH permission denied`

**Solution:** Check SSH key permissions

```bash
chmod 600 id_rsa.pem
ssh -i id_rsa.pem ubuntu@your.server.ip  # Test connection
```

### Problem: `role 'common' was not found`

**Solution:** Ensure `ansible.cfg` exists at project root with:

```ini
[defaults]
roles_path = ./ansible/roles
```

And verify roles are in `ansible/roles/`:

```bash
ls -la ansible/roles/  # Should show: common, docker, app
```

### Problem: `Cannot locate specified Dockerfile`

**Solution:** Ensure Docker files are copied correctly:

```bash
ls -la docker/app/  # Should show: Dockerfile, index.html
```

### Problem: Docker container won't start

**Solution:** Check Docker logs

```bash
ssh -i id_rsa.pem ubuntu@your.server.ip
docker logs demo_app
docker ps -a  # See all containers
```

---

## 📞 Getting Help

- **Ask in Issues** — Report bugs or ask questions
- **Discussions** — Share ideas and solutions
- **Community** — Join the Scalyz Community Slack

---

## 🎯 Key Principles

1. **Idempotency** — The playbook must be safe to run multiple times
2. **Reproducibility** — Same input = same output, every time
3. **Maintainability** — Clear, documented, easy to extend
4. **Security** — No hardcoded credentials, proper file permissions
5. **Scalability** — Works for 1 server or 100 servers

---

**Thank you for contributing! Let's build amazing infrastructure together. 💪**

---

## 👥 Project Contributors

| Name | Contribution | Role |
|------|--------------|------|
| **Noumabeu Moutacdie Jordan** | Full project implementation, Ansible roles, Docker setup, documentation | Lead Developer & Maintainer |

### How to Get Recognized

1. Submit your solution via Pull Request
2. Get approved by 2+ community reviewers
3. Your name will be added to this contributors list
4. Your work becomes part of the public portfolio

---

## 📝 License

This project is open source. See `LICENSE` for details.

Maintained with ❤️ by the Scalyz Community

