# SysOps POC

---

## 📚 Project Overview

This project provisions a complete environment using **Vagrant**, **Ansible**, **Docker**, **Docker Compose**, **Echoserver**, **Nginx** and **HAProxy**.

---

## 🛠️ Stack Overview

| Component  | Purpose                                             |
|------------|---------------------------------------------------- |
| Vagrant    | Creates Ubuntu 22.04 VM                             |
| Ansible    | Automates Docker installation, app deployment       |
| Docker     | Manages containers inside VM                        |
| HAProxy    | HTTP load balancing based on request paths          |
| Nginx      | Serve `/statics` with roundrobin loadbalancing      |
| EchoServers| Serve `/api` responses with roundrobin loadbalancing|

---

## 📂 Project Structure

```
.
├── ansible
│   ├── group_vars
│   │   └── all.yaml
│   ├── playbook.yaml
│   └── roles
│       ├── common
│       │   └── tasks
│       │       └── main.yaml
│       ├── echo_server
│       │   ├── handlers
│       │   │   └── main.yaml
│       │   └── tasks
│       │       ├── main.yaml
│       │       └── templates
│       │           └── docker-compose.yaml.j2
│       ├── haproxy
│       │   ├── handlers
│       │   │   └── main.yaml
│       │   └── tasks
│       │       ├── main.yaml
│       │       └── templates
│       │           ├── docker-compose.yaml.j2
│       │           └── haproxy.cfg.j2
│       └── nginx_server
│           ├── handlers
│           │   └── main.yaml
│           ├── tasks
│           │   └── main.yaml
│           └── templates
│               ├── default.conf.j2
│               └── docker-compose.yaml.j2
├── Readme.md
└── Vagrantfile

```
---

## 📜 HAProxy Routing Rules

| URL Path          | Routed To                  | Load Balancing Method            |
|-------------------|----------------------------|----------------------------------|
| `/api`            | EchoServers (`:8080`)      | Round Robin                      |
| `/statics`        | Nginx Servers (`:80`)      | Round Robin                      |
| `/statics/<path>` | Nginx Servers (`:80`)      | URI Hashing (Sticky Session)     |

## 🌐 How Traffic Flows

```
Local workstation (curl http://localhost:8888) 
    ➔ Vagrant Host 
        ➔ Docker HAProxy container
            ➔ Internal Docker Network (common_network) 
                ➔ EchoServer / Nginx containers
```
> **Note:**  
> All containers (**HAProxy**, **Nginx**, **EchoServer**) are connected through a common Docker bridge network.

---

## 🚀 Getting Started

### 1. Install Prerequisites
- [Vagrant](https://developer.hashicorp.com/vagrant/install)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

### 2. Clone Repository
```bash
git clone <repo_url>
cd sysops
```

### 3. Launch the Environment
```bash
vagrant up
```

This will:

- Create the VM
- Install Docker and Docker Compose
- Deploy HAProxy, EchoServer, and Nginx containers
- Configure HAProxy routing

> **Note:**  
> If you change the Vagrantfile, use: `vagrant reload`

---

## 🧪 Testing

Once the VM is up, you can test:

### Test EchoServer (`/api`)
```bash
curl http://localhost:8888/api
```
- Load balanced across EchoServers using Round Robin.

---

### Test Nginx Static Content (`/statics`)
```bash
curl http://localhost:8888/statics
```
- Served by Nginx backend via Round Robin.

---

### Test Sticky Sessions (`/statics/<path>`)
```bash
curl http://localhost:8888/statics/foo
curl http://localhost:8888/statics/bar
```
- HAProxy uses URI Hashing for path based routing.

---

## 📋 Configuration Variables

Modify these inside your Ansible variables (or `group_vars`):

| Variable              | Default | Description                       |
|------------------------|---------|----------------------------------|
| `echo_server_count`    | 3       | Number of EchoServer containers  |
| `nginx_server_count`   | 3       | Number of Nginx containers       |

### Apply Changes

After modifying variables, apply changes:

```bash
vagrant provision
```

This will automatically:

- Regenerate Docker Compose files
- Restart containers if configuration changes (using Ansible Handlers)

---

## 🛠️ Useful Docker Commands (Inside VM)

SSH into VM:
```bash
vagrant ssh
```

Then:

| Command | Purpose |
|---------|---------|
| `docker ps` | Check running containers |
| `docker logs haproxy` | View HAProxy logs |
| `docker compose down && docker compose up -d` | Restart services manually |

---

---

## 🔥 Cleaning Up

To destroy the Vagrant setup:

`vagrant destroy` or `vagrant destroy -f`


✅ Removes VM and all containers.

---

## 🛠️ Useful Links

- https://documentation.ubuntu.com/public-images/public-images-how-to/run-a-vagrant-box/
- https://developer.hashicorp.com/vagrant/tutorials/get-started/network-folder-sync
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html
- https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers
- https://singhabhishek.hashnode.dev/load-balancing-with-haproxy-a-beginners-guide
- https://docs.haproxy.org/3.0/configuration.html

