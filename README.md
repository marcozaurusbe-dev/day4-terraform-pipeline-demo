# VyOS Network Automation Lab

End-to-end Network Automation using:

- Git
- GitHub
- GitHub Actions
- Containerlab
- Docker
- VyOS
- Ansible

---

# Architecture

                  GitHub
                     |
                     v
              GitHub Actions
                     |
                     v
                Ansible
                     |
                     v
              Containerlab
                     |
          +----------+----------+
          |                     |
          v                     v
        VyOS1               VyOS2

---

# Prerequisites

Ubuntu 22.04 or later

Minimum:

- 8 GB RAM
- 4 CPU Cores
- 20 GB Free Disk

Recommended:

- 16 GB RAM
- 8 CPU Cores

---

# Install Docker

Update packages:

sudo apt update

sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

Add Docker repository:

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" \
| sudo tee /etc/apt/sources.list.d/docker.list

Install Docker:

sudo apt update

sudo apt install -y \
docker-ce \
docker-ce-cli \
containerd.io

Verify:

docker version

---

# Install Containerlab

curl -sL https://containerlab.dev/setup | sudo bash

Verify:

containerlab version

---

# Install Python

sudo apt install -y \
python3 \
python3-pip \
python3-venv

Verify:

python3 --version

---

# Create Python Virtual Environment

python3 -m venv .venv

source .venv/bin/activate

---

# Install Ansible

pip install ansible

Verify:

ansible --version

---

# Install Ansible Lint

pip install ansible-lint

Verify:

ansible-lint --version

---

# Install VyOS Collection

ansible-galaxy collection install vyos.vyos

Verify:

ansible-galaxy collection list | grep vyos

---

# Create Project Directory

mkdir vyos-network-automation

cd vyos-network-automation

---

# Clone Repository

git clone <your-repository>

cd vyos-network-automation

---

# Folder Structure

.
├── .github
│   └── workflows
│       └── network-ci.yml
│
├── containerlab
│   └── topology.yml
│
├── ansible
│   ├── inventory.yml
│   ├── site.yml
│   ├── group_vars
│   └── roles
│
└── tests

---

# Deploy Containerlab

Deploy topology:

sudo containerlab deploy \
-t containerlab/topology.yml

Expected output:

INFO Containerlab started
INFO Deploy completed

---

# Inspect Topology

sudo containerlab inspect

Example:

Name                 Kind
clab-vyos-lab-r1     linux
clab-vyos-lab-r2     linux

---

# View Running Containers

docker ps

Expected:

clab-vyos-lab-r1
clab-vyos-lab-r2

---

# Obtain Management IPs

docker inspect clab-vyos-lab-r1

docker inspect clab-vyos-lab-r2

Extract:

IPAddress

Example:

r1 = 172.20.20.2
r2 = 172.20.20.3

---

# Access Router Console

docker exec -it clab-vyos-lab-r1 bash

or

docker exec -it clab-vyos-lab-r2 bash

---

# Configure SSH

Inside VyOS:

configure

set service ssh

set system login user ansible \
authentication plaintext-password 'Ansible123!'

set system login user ansible \
level admin

commit

save

exit

Repeat on r2

---

# Test SSH

ssh ansible@172.20.20.2

ssh ansible@172.20.20.3

Expected:

Welcome to VyOS

---

# Validate Inventory

ansible-inventory \
-i ansible/inventory.yml \
--graph

Expected:

@all
  @vyos
    r1
    r2

---

# Ping Devices

ansible \
-i ansible/inventory.yml \
vyos \
-m ping

Expected:

r1 SUCCESS
r2 SUCCESS

---

# Execute Configuration Playbook

ansible-playbook \
-i ansible/inventory.yml \
ansible/site.yml

Expected:

PLAY RECAP

r1 ok=
r2 ok=

---

# Verify OSPF

ansible-playbook \
-i ansible/inventory.yml \
tests/verify.yml

Expected:

show ip ospf neighbor

---

# Destroy Lab

sudo containerlab destroy \
-t containerlab/topology.yml

---

# Git Operations

Status:

git status

Add:

git add .

Commit:

git commit \
-m "Configured OSPF"

Push:

git push origin main

---

# GitHub Actions

Workflow:

.github/workflows/network-ci.yml

Pipeline stages:

1. Checkout
2. Install Ansible
3. Install VyOS Collection
4. Lint
5. Deploy Lab
6. Configure Routers
7. Verify State

---

# Create GitHub Self-Hosted Runner

Create Directory:

mkdir actions-runner

cd actions-runner

Download:

curl -o actions-runner.tar.gz -L \
https://github.com/actions/runner/releases/latest/download/actions-runner-linux-x64.tar.gz

Extract:

tar xzf actions-runner.tar.gz

Configure:

./config.sh

Start:

./run.sh

Verify:

GitHub Repository
Settings
Actions
Runners

Status should be:

Idle

---

# Troubleshooting

## Error

docker: command not found

Fix:

sudo apt install docker-ce

Verify:

docker version

---

## Error

permission denied while trying to connect to docker

Fix:

sudo usermod -aG docker $USER

newgrp docker

Verify:

docker ps

---

## Error

network local-net not found

Fix:

docker network create local-net

Verify:

docker network ls

---

## Error

Containerlab deployment failed

Check logs:

sudo containerlab inspect

Check containers:

docker ps -a

---

## Error

No route to host

Verify router IP:

docker inspect clab-vyos-lab-r1

Check SSH:

ssh ansible@<router-ip>

---

## Error

Authentication failed

Verify:

show configuration commands | grep ansible

Reset password:

configure

delete system login user ansible

set system login user ansible \
authentication plaintext-password 'Ansible123!'

commit

save

---

## Error

ansible_connection failure

Verify:

ansible_network_os: vyos.vyos.vyos

ansible_connection: network_cli

Verify collection:

ansible-galaxy collection list

---

## Error

Collection not found

Install:

ansible-galaxy collection install vyos.vyos

---

## Error

Could not match supplied host pattern

Verify:

ansible-inventory \
-i ansible/inventory.yml \
--graph

---

## Error

GitHub runner offline

Check:

./run.sh

Check service:

sudo systemctl status actions.runner.*

Restart:

sudo systemctl restart actions.runner.*

---

# Complete Lab Teardown

Destroy topology:

sudo containerlab destroy \
-t containerlab/topology.yml

Remove containers:

docker rm -f $(docker ps -aq)

Remove networks:

docker network prune -f

Remove volumes:

docker volume prune -f

Verify:

docker ps

docker network ls