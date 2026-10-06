# Ansible Learning Lab

A practical Ansible learning lab for Linux System Administrators and DevOps Engineers.

This repository contains step-by-step exercises to learn Ansible from beginner to intermediate level.

---

## 🎯 Learning Goals

By completing this lab, you will learn how to:

- Understand Ansible architecture
- Install and configure Ansible
- Create an inventory
- Connect to Linux servers using SSH
- Run ad-hoc commands
- Create and run playbooks
- Manage packages
- Manage users and groups
- Manage files and directories
- Manage services
- Use variables
- Use handlers
- Use templates
- Use loops
- Use conditions
- Use facts
- Create roles
- Use Ansible Vault
- Work with tags
- Troubleshoot Ansible
- Build a small production-style project

---

# 1. What is Ansible?

Ansible is an automation tool used to configure and manage computers.

With Ansible, you can automate tasks such as:

```text
Install packages
Create users
Configure services
Copy files
Deploy applications
Manage configuration
Restart services
```

Instead of connecting to every server manually:

```text
Admin
  │
  ├── SSH → server01
  ├── SSH → server02
  ├── SSH → server03
  └── SSH → server04
```

Ansible allows you to manage them from one control node:

```text
                 Ansible Control Node
                         │
             ┌───────────┼───────────┐
             │           │           │
             ▼           ▼           ▼
          server01    server02    server03
```

---

# 2. Ansible Architecture

Ansible normally uses:

```text
Control Node
     │
     │ SSH
     ▼
Managed Nodes
```

### Control Node

The machine where Ansible is installed.

Example:

```text
Ubuntu 24.04
Ansible
Python
SSH
```

### Managed Node

A Linux server managed by Ansible.

Example:

```text
node01
node02
node03
```

Ansible is agentless.

You normally do not install an Ansible agent on the managed servers.

---

# 3. Lab Environment

Recommended lab:

```text
                 Ansible Control Node
                    Ubuntu Linux
                         │
              ┌──────────┼──────────┐
              │          │          │
              ▼          ▼          ▼
           node01      node02     node03
           Ubuntu      Ubuntu     Ubuntu
```

Recommended:

```text
Control Node: 1 CPU / 2 GB RAM
Managed Node: 1 CPU / 1 GB RAM each
```

You can use:

- Proxmox
- VMware
- VirtualBox
- KVM
- UTM
- Docker containers

---

# 4. Install Ansible

On Ubuntu:

```bash
sudo apt update
sudo apt install ansible -y
```

Check the version:

```bash
ansible --version
```

Example:

```text
ansible [core 2.x.x]
```

---

# 5. SSH Configuration

Ansible normally connects to Linux servers using SSH.

Create an SSH key:

```bash
ssh-keygen
```

Copy the public key:

```bash
ssh-copy-id user@node01
```

Test the connection:

```bash
ssh user@node01
```

You should be able to log in without entering the password.

---

# 6. Create the Project

Create a project directory:

```bash
mkdir ansible-lab
cd ansible-lab
```

Recommended structure:

```text
ansible-lab/
├── inventory
├── ansible.cfg
├── playbook.yml
└── README.md
```

---

# 7. Create an Inventory

Create:

```text
inventory
```

Example:

```ini
[web]
node01
node02

[db]
node03
```

You can also use IP addresses:

```ini
[web]
192.168.1.101
192.168.1.102

[db]
192.168.1.103
```

---

# 8. Test Ansible Connectivity

Run:

```bash
ansible all -i inventory -m ping
```

Expected result:

```text
node01 | SUCCESS => {
    "ping": "pong"
}

node02 | SUCCESS => {
    "ping": "pong"
}

node03 | SUCCESS => {
    "ping": "pong"
}
```

Congratulations!

You have successfully connected Ansible to your Linux servers.

---

# 9. Ad-Hoc Commands

Ad-hoc commands allow you to execute one task without creating a playbook.

Example:

```bash
ansible all -i inventory -m command -a "uptime"
```

Check memory:

```bash
ansible all -i inventory -m command -a "free -h"
```

Check disk space:

```bash
ansible all -i inventory -m command -a "df -h"
```

Check hostname:

```bash
ansible all -i inventory -m command -a "hostname"
```

---

# 10. Ansible Modules

Modules are the building blocks of Ansible.

Common modules:

```text
ansible.builtin.ping
ansible.builtin.command
ansible.builtin.shell
ansible.builtin.copy
ansible.builtin.file
ansible.builtin.package
ansible.builtin.service
ansible.builtin.user
ansible.builtin.template
```

Example:

```bash
ansible all -i inventory -m ansible.builtin.ping
```

---

# 11. Your First Playbook

Create:

```text
playbook.yml
```

Example:

```yaml
---
- name: Test Ansible
  hosts: all
  become: true

  tasks:

    - name: Show hostname
      ansible.builtin.command:
        cmd: hostname
```

Run:

```bash
ansible-playbook -i inventory playbook.yml
```

---

# 12. Install a Package

Example: install Nginx.

```yaml
---
- name: Install Nginx
  hosts: web
  become: true

  tasks:

    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
```

Run:

```bash
ansible-playbook -i inventory playbook.yml
```

---

# 13. Start a Service

```yaml
- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true
```

This means:

```text
state: started
    ↓
Start the service

enabled: true
    ↓
Start automatically after reboot
```

---

# 14. Manage Files

Create a directory:

```yaml
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    mode: "0755"
```

Create a file:

```yaml
- name: Create configuration file
  ansible.builtin.file:
    path: /opt/myapp/config.txt
    state: touch
```

Copy a file:

```yaml
- name: Copy configuration
  ansible.builtin.copy:
    src: config.txt
    dest: /opt/myapp/config.txt
```

---

# 15. Manage Users

Create a user:

```yaml
- name: Create developer user
  ansible.builtin.user:
    name: developer
    state: present
    shell: /bin/bash
```

Add the user to a group:

```yaml
- name: Add developer to wheel
  ansible.builtin.user:
    name: developer
    groups: wheel
    append: true
```

---

# 16. Variables

Variables allow you to avoid hard-coding values.

Example:

```yaml
---
- name: Variable example
  hosts: all

  vars:
    package_name: nginx

  tasks:

    - name: Install package
      ansible.builtin.package:
        name: "{{ package_name }}"
        state: present
```

Now you can change:

```yaml
package_name: nginx
```

to:

```yaml
package_name: httpd
```

without changing the task.

---

# 17. Facts

Ansible can collect information about the managed server.

Example:

```yaml
---
- name: Gather system information
  hosts: all

  tasks:

    - name: Show operating system
      ansible.builtin.debug:
        var: ansible_facts.distribution
```

You can inspect all facts:

```bash
ansible all -i inventory -m setup
```

Useful facts include:

```text
ansible_facts.hostname
ansible_facts.distribution
ansible_facts.kernel
ansible_facts.memtotal_mb
ansible_facts.processor_vcpus
```

---

# 18. Conditionals

You can execute tasks only when a condition is true.

Example:

```yaml
- name: Install nginx on Ubuntu
  ansible.builtin.apt:
    name: nginx
    state: present
  when: ansible_facts.distribution == "Ubuntu"
```

For Red Hat systems:

```yaml
- name: Install nginx on Red Hat
  ansible.builtin.dnf:
    name: nginx
    state: present
  when: ansible_facts.os_family == "RedHat"
```

---

# 19. Loops

Loops allow you to repeat a task.

Example:

```yaml
- name: Install packages
  ansible.builtin.package:
    name: "{{ item }}"
    state: present
  loop:
    - vim
    - curl
    - wget
    - git
```

Ansible processes:

```text
vim
curl
wget
git
```

---

# 20. Handlers

Handlers are usually used when a configuration change requires a service restart.

Example:

```yaml
---
- name: Configure nginx
  hosts: web
  become: true

  tasks:

    - name: Copy nginx configuration
      ansible.builtin.copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

  handlers:

    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

The handler runs only when the configuration task reports a change.

---

# 21. Templates

Templates use Jinja2.

Example:

```text
nginx.conf.j2
```

```jinja2
server {
    listen {{ nginx_port }};

    server_name {{ server_name }};
}
```

Variables:

```yaml
vars:
  nginx_port: 80
  server_name: example.com
```

---

# 22. Ansible Configuration

Create:

```text
ansible.cfg
```

Example:

```ini
[defaults]
inventory = ./inventory
remote_user = ansible
host_key_checking = False
```

Now you can run:

```bash
ansible all -m ping
```

instead of:

```bash
ansible all -i inventory -m ping
```

---

# 23. Privilege Escalation

Use:

```yaml
become: true
```

Example:

```yaml
- name: Install package
  hosts: all
  become: true

  tasks:

    - name: Install curl
      ansible.builtin.package:
        name: curl
        state: present
```

This allows Ansible to use elevated privileges, usually through `sudo`.

---

# 24. Tags

Tags allow you to execute specific parts of a playbook.

Example:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
  tags:
    - packages
```

Run only this tag:

```bash
ansible-playbook site.yml --tags packages
```

Skip it:

```bash
ansible-playbook site.yml --skip-tags packages
```

---

# 25. Ansible Vault

Ansible Vault is used to protect sensitive information.

Example:

```bash
ansible-vault create secrets.yml
```

Example content:

```yaml
database_password: "MySecretPassword"
```

Run a playbook:

```bash
ansible-playbook site.yml --ask-vault-pass
```

Never store plain-text passwords in Git.

---

# 26. Roles

Roles help organize larger Ansible projects.

Example:

```text
ansible-lab/
│
├── inventory
├── ansible.cfg
├── site.yml
│
└── roles/
    └── nginx/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── templates/
        ├── files/
        ├── vars/
        ├── defaults/
        │   └── main.yml
        └── meta/
            └── main.yml
```

Run a role:

```yaml
---
- name: Web server
  hosts: web
  become: true

  roles:
    - nginx
```

---

# 27. Ansible Galaxy

Ansible Galaxy provides reusable roles and collections.

Search:

```bash
ansible-galaxy search nginx
```

Install a role:

```bash
ansible-galaxy role install geerlingguy.nginx
```

List installed roles:

```bash
ansible-galaxy role list
```

---

# 28. Troubleshooting

Check connectivity:

```bash
ansible all -m ping
```

Increase verbosity:

```bash
ansible-playbook site.yml -v
```

More verbose:

```bash
ansible-playbook site.yml -vvv
```

Check syntax:

```bash
ansible-playbook site.yml --syntax-check
```

Check what would change:

```bash
ansible-playbook site.yml --check
```

Show differences:

```bash
ansible-playbook site.yml --check --diff
```

---

# 29. Production Best Practices

Before running Ansible against production:

```text
1. Test in a lab
2. Use version control
3. Review the playbook
4. Use --check when possible
5. Use --diff for configuration changes
6. Use Ansible Vault for secrets
7. Use roles for larger projects
8. Use tags
9. Keep backups
10. Use change management
```

Avoid:

```yaml
shell: rm -rf /some/path/*
```

when a safer Ansible module can be used.

Prefer:

```yaml
ansible.builtin.file:
```

or:

```yaml
ansible.builtin.package:
```

instead of unnecessary shell commands.

---

# 30. Idempotency

One of the most important Ansible concepts is **idempotency**.

If you run this:

```yaml
- name: Install nginx
  ansible.builtin.package:
    name: nginx
    state: present
```

the first run may produce:

```text
changed=1
```

Run it again:

```text
changed=0
```

because Nginx is already installed.

This is a major difference between automation and simply running shell commands.

---

# 31. Complete Mini Project

Build the following environment:

```text
                 Ansible Control Node
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
       web01           web02          db01
       Ubuntu          Ubuntu         Ubuntu
          │              │              │
        Nginx          Nginx         PostgreSQL
```

Requirements:

### Web servers

Install:

```text
nginx
git
curl
```

Configure:

```text
nginx
```

Start and enable the service.

### Database server

Install:

```text
postgresql
```

Start and enable PostgreSQL.

### Users

Create:

```text
ansibleadmin
developer
```

### Monitoring

Create:

```text
/opt/monitoring/status.txt
```

containing:

```text
Server: <hostname>
OS: <distribution>
Kernel: <kernel version>
```

---

# 32. Recommended Project Structure

```text
ansible-production-lab/
│
├── README.md
├── ansible.cfg
├── inventory/
│   ├── hosts
│   └── group_vars/
│       ├── web.yml
│       └── db.yml
│
├── playbooks/
│   ├── site.yml
│   ├── web.yml
│   └── database.yml
│
├── roles/
│   ├── nginx/
│   ├── postgresql/
│   └── common/
│
├── templates/
├── files/
└── vault/
```

---

# 33. Useful Commands Cheat Sheet

```bash
# Version
ansible --version

# Test connectivity
ansible all -m ping

# Run command
ansible all -m command -a "uptime"

# List hosts
ansible all --list-hosts

# Run playbook
ansible-playbook site.yml

# Syntax check
ansible-playbook site.yml --syntax-check

# Dry run
ansible-playbook site.yml --check

# Show differences
ansible-playbook site.yml --check --diff

# Verbose mode
ansible-playbook site.yml -vvv

# Vault
ansible-vault create secrets.yml

# Galaxy
ansible-galaxy role list
```

---

# 34. Learning Path

Follow this order:

```text
01. Linux basics
       ↓
02. SSH
       ↓
03. Ansible installation
       ↓
04. Inventory
       ↓
05. Ad-hoc commands
       ↓
06. Modules
       ↓
07. Playbooks
       ↓
08. Variables
       ↓
09. Facts
       ↓
10. Conditionals
       ↓
11. Loops
       ↓
12. Handlers
       ↓
13. Templates
       ↓
14. Tags
       ↓
15. Ansible Vault
       ↓
16. Roles
       ↓
17. Ansible Galaxy
       ↓
18. Production project
```

---

# 35. Exercises

## Exercise 1 — Connectivity

Create an inventory with three Linux servers.

Run:

```bash
ansible all -m ping
```

Goal:

```text
3 hosts → SUCCESS
```

---

## Exercise 2 — System Information

Use Ansible to collect:

```text
Hostname
IP address
Operating system
Kernel
Memory
CPU
```

---

## Exercise 3 — Package Management

Install:

```text
vim
curl
wget
git
```

on all servers.

---

## Exercise 4 — Users

Create:

```text
developer
operator
```

on every server.

---

## Exercise 5 — Nginx

Install and configure Nginx on the `web` group.

---

## Exercise 6 — PostgreSQL

Install PostgreSQL only on the `db` group.

---

## Exercise 7 — Templates

Create a dynamic web page containing:

```text
Hostname:
Operating System:
Kernel:
```

---

## Exercise 8 — Handler

Change the Nginx configuration and restart Nginx only when the configuration changes.

---

## Exercise 9 — Vault

Store a database password using Ansible Vault.

---

## Exercise 10 — Production Project

Create a complete Ansible project containing:

```text
Inventory
Variables
Playbooks
Roles
Templates
Handlers
Vault
Tags
Documentation
```

---

# 36. Final Goal

After completing this repository, you should be comfortable with:

```text
Linux
   │
   ├── SSH
   │
   └── Ansible
          │
          ├── Inventory
          ├── Modules
          ├── Playbooks
          ├── Variables
          ├── Facts
          ├── Loops
          ├── Conditions
          ├── Handlers
          ├── Templates
          ├── Roles
          └── Vault
```

The final goal is not only to know Ansible commands.

The goal is to be able to **automate a real Linux environment safely and repeatably**.

---

## 🚀 Next Step

Build the lab with:

```text
1 × Ansible Control Node
3 × Ubuntu Linux Nodes
```

Then complete the exercises from top to bottom.

A good target project is:

```text
Ansible
+
Linux
+
Nginx
+
PostgreSQL
+
Docker
+
Git
+
CI/CD
```

This combination provides a strong foundation for a Linux System Administrator / DevOps Engineer role.
