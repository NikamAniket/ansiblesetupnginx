# Ansible Lab Setup (AWS EC2 → Docker Containers → Nginx Installation)

## Part 1: Create AWS EC2 Instance

### Step 1: Launch EC2

* OS: Ubuntu 24.04 LTS
* Instance Type: t2.medium (recommended)
* Storage: 20 GB
* Security Group: Allow SSH (Port 22)

### Step 2: Connect to EC2

Connect to the EC2 instance.

---


## Part 2: Install Docker

### Step 3: Install Docker


sudo -i
apt update
apt install docker.io -y


Installs Docker on EC2.

### Step 4: Verify Docker

docker -v
docker ps


Checks Docker installation.

---

## Part 3: Create Containers

### Step 5: Create Master and Target Containers

docker run -itd --name ansible_master ubuntu /bin/bash
docker run -itd --name target1 ubuntu /bin/bash
docker run -itd --name target2 ubuntu /bin/bash


Creates 3 Ubuntu containers.

### Step 6: Verify Containers

docker ps


Lists running containers.

# Part 4: Install Ansible on Master

### Step 7: Login to Master

docker exec -it ansible_master bash

Access ansible master container.

### Step 8: Install Ansible
apt update
apt install python-is-python3 vim iputils-ping openssh-client -y
apt install software-properties-common -y
add-apt-repository --yes --update ppa:ansible/ansible
apt install ansible -y


Installs Ansible and dependencies.

### Step 9: Verify Installation

ansible --version

Displays installed Ansible version.

---

# Part 5: Configure Target1

### Step 10: Login
docker exec -it target1 bash

Access target container.
### Step 11: Install SSH

apt update
apt install vim python-is-python3 iputils-ping openssh-client -y
apt install openssh-server -y

Installs SSH server.

### Step 12: Enable Root Login

cd /etc/ssh
vi sshd_config

Modify:

```text
PermitRootLogin yes
PasswordAuthentication yes
```

Allows SSH login using root.

### Step 13: Start SSH
service ssh status
service ssh start
service ssh status

Starts SSH service.

### Step 14: Set Root Password

passwd root

Example:
admin
admin
ts root password.

---

# Part 6: Configure Target2

### Step 15: Repeat Steps 10–14

docker exec -it target2 bash


Configure second target machine.

---

# Part 7: Get Target IPs

### Step 16: Find Container IPs

docker inspect target1 | grep IPAddress
docker inspect target2 | grep IPAddress

or 
sudo docker inspect target1
sudo docker inspect target2 

Example:
172.17.0.3
172.17.0.4
Used by Ansible inventory.

---

# Part 8: Configure Ansible Inventory

### Step 17: Login to Master
docker exec -it ansible_master bash

### Step 18: Edit Hosts File
cd /etc/ansible
vi hosts


Add:

eb]
172.17.0.3
172.17.0.4
```

Inventory of managed hosts.

---

# Part 9: Setup Passwordless SSH Master

### Step 19: Generate SSH Key
ssh-keygen

Creates public/private key pair.

### Step 20: Copy Key to Target1

ssh-copy-id root@172.17.0.3


Allows passwordless login.

### Step 21: Copy Key to Target2

```bash
ssh-copy-id root@172.17.0.4
```

Allows passwordless login.

ssh root@172.17.0.3
ssh root@172.17.0.4
\\ests connectivity.

---

# Part 10: Verify Ansible Connectivity

### Step 23: Ping All Hosts

```bash
ansible all -m ping
```

Expected:

```text
172.17.0.3 | SUCCESS
172.17.0.4 | SUCCESS
```

Confirms Ansible connectivity.

---

# Part 11: Create Nginx Playbook

### Step 24: Create Playbook

```bash
vi /etc/ansible/installnginx.yaml
```

Paste:

```yaml
---
- hosts: all
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: latest
```

Installs latest Nginx on all targets.

---

# Part 12: Run Playbook

### Step 25: Execute Playbook

```bash
ansible-playbook installnginx.yaml
```

Runs installation on all servers.

---

# Part 13: Verify Nginx Installation

### Step 26: Check Nginx Version

ansible all -a "nginx -v"

Displays installed Nginx version.

### Step 27: Check Nginx Binary

ansible all -a "which nginx"

Expected:
/usr/sbin/nginx

Confirms Nginx installation.

### Step 28: Verify Playbook Success


PLAY RECAP

172.17.0.3 : ok=2 failed=0
172.17.0.4 : ok=2 failed=0
This confirms Nginx was successfully installed on both target servers.
