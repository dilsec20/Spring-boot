# ⚙️ Ansible — Complete In-Depth Guide

> **"Ansible is an IT automation tool. It configures systems, deploys software, and orchestrates advanced IT tasks such as continuous deployments or zero downtime rolling updates. The best part? It's agentless."**

---

## 📑 Table of Contents

1. [What is Ansible?](#1-what-is-ansible)
2. [Ansible Architecture](#2-ansible-architecture)
3. [Ansible vs Terraform vs Chef/Puppet](#3-ansible-vs-terraform-vs-chefpuppet)
4. [Installation & Setup](#4-installation--setup)
5. [Inventory (Hosts)](#5-inventory-hosts)
6. [Ad-Hoc Commands](#6-ad-hoc-commands)
7. [Playbooks (YAML)](#7-playbooks-yaml)
8. [Modules (The building blocks)](#8-modules-the-building-blocks)
9. [Variables & Facts](#9-variables--facts)
10. [Conditionals & Loops](#10-conditionals--loops)
11. [Roles (Code Organization)](#11-roles-code-organization)
12. [Ansible Vault (Secrets)](#12-ansible-vault-secrets)
13. [Deploying Spring Boot with Ansible](#13-deploying-spring-boot-with-ansible)
14. [Ansible Best Practices](#14-ansible-best-practices)
15. [Interview Questions & Answers (50+)](#15-interview-questions--answers-50)

---

## 1. What is Ansible?

```
Problem: You need to install Java, Nginx, and deploy a JAR file to 50 servers.
Solution WITHOUT Ansible: SSH into 50 servers manually. Run commands 50 times. 😩
Solution WITH Ansible: Write one Playbook. Run one command. Ansible configures all 50 servers in parallel. ✅

Why Ansible is amazing:
  ✅ Agentless: No software to install on the target servers! Uses standard SSH.
  ✅ Idempotent: Run it 100 times, the result is the same. It only makes changes if needed.
  ✅ YAML: Playbooks are written in simple, readable YAML.
  ✅ Push-based: You push config from your laptop/control node to the servers.
```

---

## 2. Ansible Architecture

```
┌──────────────────────────────────────┐
│            Control Node              │
│       (Your Laptop / Jenkins)        │
│                                      │
│  ┌────────────┐     ┌─────────────┐  │
│  │ Inventory  │     │ Playbooks   │  │
│  │ (hosts.ini)│     │ (deploy.yml)│  │
│  └────────────┘     └─────────────┘  │
│            │               │         │
│            └───────┬───────┘         │
│                    │                 │
│             Ansible Engine           │
└────────────────────┬─────────────────┘
                     │ SSH (Agentless!)
          ┌──────────┼──────────┐
          ▼          ▼          ▼
   ┌─────────┐  ┌─────────┐  ┌─────────┐
   │ Server 1│  │ Server 2│  │ Server 3│
   │ (Web)   │  │ (Web)   │  │ (DB)    │
   └─────────┘  └─────────┘  └─────────┘
```

---

## 3. Ansible vs Terraform vs Chef/Puppet

| Tool | Primary Use | Agent Required? | Architecture | Language |
|------|-------------|-----------------|--------------|----------|
| **Ansible** | Configuration Mgmt | ❌ No (SSH) | Push | YAML |
| **Terraform**| Provisioning (Cloud) | ❌ No (API calls) | Push | HCL |
| **Chef** | Configuration Mgmt | ✅ Yes | Pull | Ruby |
| **Puppet** | Configuration Mgmt | ✅ Yes | Pull | Ruby (DSL) |

*Note: Use Terraform to create the EC2 instance, then use Ansible to configure what's inside the EC2 instance!*

---

## 4. Installation & Setup

```bash
# Install on Control Node (Linux/macOS)
# Target nodes just need Python and SSH access!

# Ubuntu/Debian
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

# macOS
brew install ansible

# Verify
ansible --version

# Setup SSH keys to target nodes (crucial for agentless!)
ssh-keygen -t ed25519
ssh-copy-id user@server-ip
```

---

## 5. Inventory (Hosts)

```ini
# inventory.ini — Tells Ansible WHICH servers to manage

# Uncategorized host
192.168.1.10

# Grouped hosts
[webservers]
web1.example.com
web2.example.com
192.168.1.11

[dbservers]
db1.example.com

# Groups of groups
[production:children]
webservers
dbservers

# Variables for a specific host
web1.example.com ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/mykey.pem
```

---

## 6. Ad-Hoc Commands

```bash
# Run a single command quickly without writing a playbook
# Syntax: ansible [group] -i [inventory] -m [module] -a [arguments]

# Ping all servers in inventory to test connectivity
ansible all -i inventory.ini -m ping

# Check uptime on webservers
ansible webservers -i inventory.ini -a "uptime"

# Install Nginx on webservers (uses apt module, run with sudo -b)
ansible webservers -i inventory.ini -b -m apt -a "name=nginx state=present"

# Copy a file to all servers
ansible all -i inventory.ini -m copy -a "src=app.jar dest=/opt/app.jar"
```

---

## 7. Playbooks (YAML)

```yaml
# deploy.yml — The recipe for configuration

---
- name: Deploy Spring Boot Application
  hosts: webservers            # Which hosts from inventory?
  become: yes                  # Run with sudo? (yes)
  
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        
    - name: Install Java 21
      apt:
        name: openjdk-21-jre
        state: present
        
    - name: Create application directory
      file:
        path: /opt/myapp
        state: directory
        mode: '0755'
        
    - name: Copy JAR file to server
      copy:
        src: ./target/myapp.jar
        dest: /opt/myapp/myapp.jar
        
    - name: Copy Systemd service file
      template:
        src: ./myapp.service.j2
        dest: /etc/systemd/system/myapp.service
      notify:                   # Triggers a handler if this task makes a change
        - Reload Systemd
        
    - name: Start and enable application service
      systemd:
        name: myapp
        state: started
        enabled: yes

  handlers:
    # Handlers only run at the end of the play, and ONLY if notified
    - name: Reload Systemd
      systemd:
        daemon_reload: yes
```

```bash
# Run the playbook
ansible-playbook -i inventory.ini deploy.yml

# Dry run (see what WOULD happen without actually making changes)
ansible-playbook -i inventory.ini deploy.yml --check
```

---

## 8. Modules (The building blocks)

```yaml
# Ansible has thousands of modules. They do the actual work.

# apt / yum: Package management
apt: name=nginx state=present
yum: name=httpd state=latest

# copy: Copy files from control node to target
copy: src=file.txt dest=/tmp/file.txt

# template: Copy file + replace variables (uses Jinja2)
template: src=config.j2 dest=/etc/config.conf

# file: Manage files and directories (permissions, ownership)
file: path=/data state=directory owner=appgroup mode=0755

# service / systemd: Manage services
systemd: name=nginx state=restarted enabled=yes

# user: Manage users
user: name=dilip groups=sudo shell=/bin/bash

# git: Clone repositories
git: repo=https://github.com/myorg/repo.git dest=/opt/code

# command / shell: Run raw commands (use only if no module exists)
shell: cat /etc/passwd | grep dilip > /tmp/out.txt
```

---

## 9. Variables & Facts

```yaml
# Defining variables
---
- hosts: all
  vars:
    app_port: 8080
    app_name: "SpringApp"
  
  tasks:
    - name: Print variable
      debug:
        msg: "Starting {{ app_name }} on port {{ app_port }}"

# Using Facts (Ansible automatically gathers info about the target server!)
    - name: Print server OS
      debug:
        msg: "This server is running {{ ansible_distribution }} {{ ansible_distribution_version }}"
        
    - name: Print server IP
      debug:
        msg: "The IP is {{ ansible_default_ipv4.address }}"
```

---

## 10. Conditionals & Loops

```yaml
# Conditionals (when)
- name: Install Apache on Debian systems
  apt:
    name: apache2
    state: present
  when: ansible_os_family == "Debian"

- name: Install Apache on RedHat systems
  yum:
    name: httpd
    state: present
  when: ansible_os_family == "RedHat"


# Loops (loop / with_items)
- name: Install multiple packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - htop
    - openjdk-21-jre

- name: Create multiple users
  user:
    name: "{{ item.name }}"
    groups: "{{ item.group }}"
  loop:
    - { name: 'alice', group: 'admin' }
    - { name: 'bob', group: 'dev' }
```

---

## 11. Roles (Code Organization)

```
When playbooks get too big, you split them into ROLES.
A role is an independent, reusable unit of tasks, variables, and templates.

Directory Structure:
my-ansible-project/
├── inventory.ini
├── site.yml                 # Master playbook
└── roles/
    ├── java/                # Role 1
    │   └── tasks/main.yml
    ├── database/            # Role 2
    │   └── tasks/main.yml
    └── springboot-app/      # Role 3
        ├── tasks/main.yml
        ├── templates/myapp.service.j2
        └── vars/main.yml
```

```yaml
# site.yml
---
- hosts: dbservers
  roles:
    - database

- hosts: webservers
  roles:
    - java
    - springboot-app
```

---

## 12. Ansible Vault (Secrets)

```bash
# Never store plain-text passwords in Git!

# Create an encrypted file
ansible-vault create secrets.yml
# Prompts for a password. Opens editor.
# Type: db_password: "super_secret_password"

# Edit existing vault
ansible-vault edit secrets.yml

# Run playbook that uses vaulted variables
ansible-playbook -i inventory.ini site.yml --ask-vault-pass
```

---

## 13. Deploying Spring Boot with Ansible

### The Playbook (`deploy-springboot.yml`)

```yaml
---
- name: Deploy Spring Boot App
  hosts: production
  become: yes
  vars:
    app_name: "payment-service"
    app_jar: "target/{{ app_name }}.jar"
    app_dir: "/opt/{{ app_name }}"
    app_user: "spring_user"
    app_port: 8080

  tasks:
    - name: Install Java
      apt:
        name: openjdk-21-jre-headless
        state: present
        update_cache: yes

    - name: Create application user
      user:
        name: "{{ app_user }}"
        system: yes
        shell: /usr/sbin/nologin

    - name: Create application directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"

    - name: Copy JAR file
      copy:
        src: "{{ app_jar }}"
        dest: "{{ app_dir }}/app.jar"
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0744'
      notify: Restart App

    - name: Create Systemd service
      template:
        src: springboot.service.j2
        dest: "/etc/systemd/system/{{ app_name }}.service"
      notify: 
        - Reload Systemd
        - Restart App

    - name: Ensure service is started and enabled
      systemd:
        name: "{{ app_name }}"
        state: started
        enabled: yes

  handlers:
    - name: Reload Systemd
      systemd:
        daemon_reload: yes

    - name: Restart App
      systemd:
        name: "{{ app_name }}"
        state: restarted
```

### The Template (`springboot.service.j2`)

```ini
[Unit]
Description={{ app_name }}
After=syslog.target network.target

[Service]
User={{ app_user }}
ExecStart=/usr/bin/java -jar {{ app_dir }}/app.jar --server.port={{ app_port }}
SuccessExitStatus=143
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

---

## 14. Ansible Best Practices

1. **Keep it Idempotent** — Ensure tasks can be run repeatedly without breaking things. (Use modules, avoid `shell` / `command` when possible).
2. **Use Roles** — Break large playbooks into reusable roles (Ansible Galaxy has thousands of community roles).
3. **Use Version Control** — Treat infrastructure as code. Commit playbooks to Git.
4. **Use Ansible Vault** — Encrypt all secrets (passwords, API keys, private keys).
5. **Name your tasks** — Always give `name:` to tasks so the output is readable.
6. **Use Native Modules** — Use the `apt` module instead of `shell: apt-get install`.
7. **Test with `--check`** — Always run a dry run in production before applying.
8. **Use Handlers** — Restart services only if the configuration actually changed.

---

## 15. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Ansible?** Open-source IT automation tool for configuration management, application deployment, and task automation.

**Q2: What makes Ansible agentless?** It doesn't require any software installed on target nodes. It uses standard SSH (Linux) or WinRM (Windows) to connect and execute commands.

**Q3: What is a Playbook?** A YAML file containing a series of tasks (the recipe) to be executed on specified hosts.

**Q4: What is an Inventory?** A file (usually INI or YAML) that lists the IP addresses or hostnames of the target servers, optionally organized into groups.

**Q5: What is Idempotence?** The property that running a task once has the same effect as running it multiple times. Ansible only makes changes if the system is not in the desired state.

**Q6: What language are Playbooks written in?** YAML.

**Q7: What language is Ansible written in?** Python. (Target nodes must have Python installed).

**Q8: What is an Ad-Hoc command?** A single Ansible command run directly from the CLI without a playbook. Useful for quick tasks like pinging servers.

---

### Intermediate

**Q9: Ansible vs Terraform?** Terraform is for Provisioning (creating infrastructure like AWS VPCs, EC2s). Ansible is for Configuration Management (installing software inside those EC2s).

**Q10: What are Ansible Modules?** Standalone scripts that Ansible executes on the target node. (e.g., `apt`, `copy`, `service`, `file`).

**Q11: What are Ansible Facts?** System variables automatically discovered by Ansible about the target node (OS version, IP address, CPU count). Accessed via `ansible_facts`.

**Q12: What is a Handler?** A special task that runs only when notified by another task, and only runs once at the end of the play. Commonly used to restart services after config changes.

**Q13: What is a Role?** A way to group related tasks, variables, files, and templates into a reusable, structured package.

**Q14: What is Ansible Vault?** A feature to encrypt sensitive data (passwords, keys) within Ansible files, so they can be safely stored in version control.

**Q15: How do you use variables in a string?** Using Jinja2 templating syntax: `{{ variable_name }}`.

---

### Rapid-Fire (Q16–Q50)

**Q16: Command to run a playbook?** `ansible-playbook playbook.yml`.

**Q17: Command to ping all hosts?** `ansible all -m ping`.

**Q18: What is Jinja2?** The templating engine Ansible uses for variables and templates (e.g., `.j2` files).

**Q19: How to run a task as root?** Use `become: yes`.

**Q20: What is the `copy` module?** Copies files from the control node to the target node.

**Q21: What is the `template` module?** Copies files from control node to target node AND evaluates Jinja2 variables within the file.

**Q22: How to do loops in Ansible?** Use the `loop` (or older `with_items`) keyword on a task.

**Q23: How to do conditionals in Ansible?** Use the `when` keyword on a task.

**Q24: What is `ansible-galaxy`?** Command-line tool to download community-created roles from the Ansible Galaxy repository.

**Q25: What is `--check` flag?** Dry run. Shows what would change without making actual changes.

**Q26: What is `--diff` flag?** Shows the exact differences made to files (useful with templates).

**Q27: What is `--tags` flag?** Runs only specific tasks tagged with a certain keyword.

**Q28: What is the `group_vars` folder?** A directory to define variables that apply to an entire group of hosts from the inventory.

**Q29: What is the `host_vars` folder?** A directory to define variables for a specific single host.

**Q30: What is `ansible.cfg`?** The primary configuration file for Ansible itself (e.g., setting default inventory path, timeout).

**Q31: What is the `command` module?** Executes a raw command on the target. Bypasses the shell (no pipes `|` or redirects `>`).

**Q32: What is the `shell` module?** Executes a raw command through the shell (`/bin/sh`). Supports pipes and redirects.

**Q33: Why avoid `command`/`shell` modules?** They are not natively idempotent. You have to handle idempotency manually (e.g., using `creates` or `removes`).

**Q34: What is `gather_facts: no`?** Disables fact gathering at the start of a play to speed up execution if facts aren't needed.

**Q35: What is `ansible_user`?** Inventory variable defining which SSH user to connect as.

**Q36: What is `ansible_ssh_private_key_file`?** Inventory variable defining the SSH key to use for connection.

**Q37: What does `notify` do?** Triggers a handler task.

**Q38: Can a handler be notified multiple times?** Yes, but it will still only execute ONCE at the end of the play.

**Q39: What is `register`?** Saves the output/result of a task into a variable for later use.

**Q40: How to ignore task errors?** `ignore_errors: yes`.

**Q41: What is `ansible-doc`?** Command-line tool to view documentation for Ansible modules.

**Q42: What is Ansible Tower / AWX?** Enterprise web UI / REST API for managing Ansible runs, schedules, and RBAC. (AWX is the open-source version).

**Q43: What is `delegate_to`?** Runs a task on a specific host other than the current target host (e.g., updating a load balancer).

**Q44: What is `local_action`?** Runs a task on the control node itself instead of the target node.

**Q45: What is a dynamic inventory?** An inventory script/plugin that queries a cloud provider (AWS, GCP) in real-time to get the list of servers, instead of a static text file.

**Q46: How do you encrypt an existing file with Vault?** `ansible-vault encrypt filename.yml`.

**Q47: How to view an encrypted vault file?** `ansible-vault view filename.yml`.

**Q48: What is `pre_tasks` and `post_tasks`?** Task lists that run explicitly before or after the roles defined in a play.

**Q49: What is the `file` module used for?** Creating directories, deleting files/directories, changing permissions/ownership.

**Q50: How does Ansible track state?** It doesn't store state centrally (like Terraform). It connects to the node, checks current state, and compares it to desired state in real-time.

---

## 📚 References

- [Ansible Official Documentation](https://docs.ansible.com/)
- [Ansible Modules List](https://docs.ansible.com/ansible/latest/collections/index_module.html)
- [Ansible Galaxy (Community Roles)](https://galaxy.ansible.com/)

---

> **Previous Topic:** [← 28 - Linux](../28-linux/README.md)  
> **Next Topic:** [30 - Jenkins →](../30-jenkins/README.md)
