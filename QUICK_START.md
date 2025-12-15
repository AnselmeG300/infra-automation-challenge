# 🚀 Quick Start Guide

Automated server bootstrap and containerized app deployment using **Ansible**.

---

## 📋 Prerequisites

- **Ansible** installed locally: `ansible --version`
- **SSH access** to your Ubuntu server
- **Private SSH key** in the project root (see setup below)

---

## 🔧 Setup

### 1. **Place Your Private SSH Key**

Copy your private SSH key to the project root:

```bash
cp /path/to/your/key.pem ./id_rsa.pem
chmod 600 ./id_rsa.pem
```

The key must be named `id_rsa.pem` and placed at the **root level** of the project:

```
infra-automation-challenge/
├── id_rsa.pem                    ← Your private key (600 permissions)
├── ansible.cfg
├── ansible/
│   ├── inventory.ini
│   ├── playbooks/
│   └── roles/
└── docker/
```

### 2. **Configure Your Server in Inventory**

Edit `ansible/inventory.ini` and add your server(s):

```ini
[all]
18.156.129.16  ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem ansible_python_interpreter=/usr/bin/python3

[app_servers]
18.156.129.16

```

**Replace `18.156.129.16`** with your actual server IP address.

- `ansible_user=ubuntu` — SSH username (for Ubuntu servers)
- `ansible_ssh_private_key_file=./id_rsa.pem` — Path to your private key

---

## ▶️ Run the Deployment

Execute the playbook to bootstrap and deploy your server:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/site.yml
```

### Expected Output

The playbook will:

1. ✅ Gather facts about the server
2. ✅ Update APT cache and install required packages
3. ✅ Create a `deploy` user with sudo access
4. ✅ Install Docker with JSON logging configuration
5. ✅ Configure log rotation for fail2ban and Docker
6. ✅ Copy the application files
7. ✅ Build the Docker image
8. ✅ Launch the containerized app on port 80

```
PLAY [Provisionnement et déploiement serveur] ***
TASK [Gathering Facts] ... ok
TASK [common : Mise à jour du cache APT] ... changed
TASK [common : Installation des paquets requis] ... changed
TASK [common : Création de l'utilisateur deploy] ... ok
TASK [docker : Installer Docker] ... changed
TASK [app : Construire l'image Docker] ... changed
TASK [app : Lancer le conteneur applicatif] ... changed

PLAY RECAP ***
your.server.ip : ok=8 changed=5 unreachable=0 failed=0
```

---

## ✔️ Verify Deployment

### Check if the app is running

```bash
# From your local machine
curl http://18.156.129.16
```

You should see the sample HTML page served by Nginx inside Docker.

### SSH into the server to verify

```bash
ssh -i id_rsa.pem ubuntu@18.156.129.16

# Check Docker container
docker ps

# View Docker logs
docker logs demo_app
```

---

## 🔄 Re-run Deployment (Idempotent)

You can safely re-run the playbook multiple times:

```bash
ansible-playbook -i ansible/inventory.ini ansible/playbooks/site.yml
```

The playbook is **idempotent** — it won't break anything on subsequent runs, only apply necessary changes.

---

## 📁 Project Structure

```
infra-automation-challenge/
├── README.md                    # Assignment overview
├── QUICK_START.md              # This file
├── CONTRIBUTING.md             # Detailed documentation
├── id_rsa.pem                  # Your SSH private key (⚠️ add to .gitignore)
├── ansible.cfg                 # Ansible configuration
├── ansible/
│   ├── inventory.ini           # Server inventory with IPs
│   ├── playbooks/
│   │   └── site.yml            # Main playbook
│   ├── roles/
│   │   ├── common/             # OS setup, user creation
│   │   ├── docker/             # Docker installation
│   │   └── app/                # App deployment
│   │   └── logging/            # App deployment
│   └── vars/
│       └── main.yml            # Global variables
└── docker/
    └── app/
        ├── Dockerfile          # Nginx container
        └── index.html          # Sample webpage
```

---

## ⚠️ Important Notes

### SSH Key Security

- Never commit `id_rsa.pem` to git — add it to `.gitignore`
- The key should have **600 permissions** (`chmod 600 id_rsa.pem`)
- Use a **unique key per environment** (dev, staging, prod)

### Updating Server IPs

To deploy to different servers, simply edit `ansible/inventory.ini`:

```ini

[all]
18.156.129.16  ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem ansible_python_interpreter=/usr/bin/python3
192.168.1.100 ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem ansible_python_interpreter=/usr/bin/python3
192.168.1.101 ansible_user=ubuntu ansible_ssh_private_key_file=./id_rsa.pem ansible_python_interpreter=/usr/bin/python3

[app_servers]
18.156.129.16
192.168.1.100
192.168.1.101

```

---

## 🆘 Troubleshooting

### SSH Connection Issues

```bash
# Test SSH connectivity
ssh -i id_rsa.pem ubuntu@your.server.ip

# Verify key permissions
ls -la id_rsa.pem  # Should show: -rw------- (600)
```

### Ansible Can't Find Inventory

Make sure you're in the **project root** directory:

```bash
pwd  # Should be: /path/to/infra-automation-challenge
ansible-playbook -i ansible/inventory.ini ansible/playbooks/site.yml
```

### Docker Build Fails

Check that Docker files are in the correct location:

```bash
ls -la docker/app/
# Should show: Dockerfile, index.html
```

---

## 📚 Learn More

See `CONTRIBUTING.md` for detailed technical documentation and project architecture.

---

**Happy deploying! 🎉**
