## Ansible Inventory and Default Inventory 

Ansible works by **connecting to remote machines and executing tasks on them**.  
But before Ansible can do anything, it must know:

- **Which servers exist**
- **How to connect to them**
- **How those servers are organized**

This information is stored in something called the **Ansible Inventory**.

Inventory is one of the **core building blocks of Ansible automation**.

---

### 1. What is Ansible Inventory?

An **Ansible Inventory** is a file or collection of files that contains the **list of servers managed by Ansible**.

These servers are called:

- hosts
- managed nodes
- target machines

Inventory tells Ansible:

- where to run tasks
- how to connect to servers
- which credentials to use

In simple terms:

> Inventory is the **database of infrastructure** for Ansible.

---

### 2. Why Inventory is Required

Consider a simple Ansible command:

```
ansible all -m ping
```

This command means:

```
Run the ping module on all hosts
```

But where are these hosts defined?

They come from the **inventory**.

Without inventory, Ansible would not know which machines to target.

---

### 3. Basic Inventory Example

A simple inventory file may look like this:

```
[webservers]
192.168.1.10
192.168.1.11

[databases]
192.168.1.20
```

Explanation:

| Section | Meaning |
|---|---|
webservers | group of web servers |
databases | group of database servers |

Servers can be targeted using these group names.

Example:

```
ansible webservers -m ping
```

---

### 4. Types of Inventory in Ansible

Ansible supports two major types of inventory.

| Type | Description |
|---|---|
Static Inventory | manually defined host lists |
Dynamic Inventory | generated automatically |

Most beginners start with **static inventory**.

---

# STATIC INVENTORY

### 5. Static Inventory

Static inventory is a simple text file containing hosts.

Example:

```
[webservers]
web1
web2
```

It is the most common type used in:

- small infrastructure
- development environments
- simple automation tasks

---

### 6. Inventory File Formats

Ansible supports multiple inventory formats.

Most common formats are:

| Format | Description |
|---|---|
INI format | traditional inventory |
YAML format | modern structured inventory |

---

### 7. INI Style Inventory (Most Common)

Example:

```
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

This is the **classic Ansible inventory format**.

---

### 8. YAML Inventory Format

Example:

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
        web2:
    dbservers:
      hosts:
        db1:
```

YAML inventory is easier to structure for large infrastructures.

---

### 9. Using Hostnames Instead of IPs

Inventory does not require IP addresses.

You can use:

- DNS names
- domain names
- internal hostnames

Example:

```
[webservers]
web1.company.com
web2.company.com
```

---

### 10. Grouping Hosts

Grouping hosts is extremely useful.

Example groups:

```
[webservers]
web1
web2

[databases]
db1
db2
```

Now playbooks can target groups.

Example:

```
hosts: webservers
```

This runs tasks only on web servers.

---

### 11. Nested Groups

Groups can contain other groups.

Example:

```
[webservers]
web1
web2

[dbservers]
db1

[production:children]
webservers
dbservers
```

Now targeting `production` runs tasks on both groups.

---

### 12. Host Variables

Variables can be defined for individual hosts.

Example:

```
web1 ansible_host=192.168.1.10 http_port=80
web2 ansible_host=192.168.1.11 http_port=8080
```

Now:

| Host | Port |
|---|---|
web1 | 80 |
web2 | 8080 |

---

### 13. Group Variables

Variables can also apply to an entire group.

Example:

```
[webservers]
web1
web2

[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/key.pem
```

Now both servers share these settings.

---

# ANSIBLE DEFAULT INVENTORY

### 14. What is Default Inventory?

Ansible has a **default inventory location**.

Default path:

```
/etc/ansible/hosts
```

If you run:

```
ansible all -m ping
```

Ansible automatically reads:

```
/etc/ansible/hosts
```

unless another inventory is specified.

---

### 15. Overriding Inventory Using CLI

We can specify inventory manually:

```
ansible-playbook -i hosts playbook.yml
```

Here:

```
-i
```

means inventory.

This overrides the default inventory.

---

# CONFIGURING DEFAULT INVENTORY

### 16. Setting Default Inventory in ansible.cfg

Instead of passing `-i` every time, we can configure the inventory path in the Ansible configuration file.

Example configuration:

```ini
[defaults]
inventory = /path/to/your/inventory
```

Now Ansible will automatically use that inventory.

---

### 17. Example Project with Default Inventory

Project structure:

```
ansible-project/
│
├── ansible.cfg
├── hosts
└── deploy.yml
```

ansible.cfg:

```
[defaults]
inventory = hosts
```

Now you can run:

```
ansible-playbook deploy.yml
```

without specifying `-i`.

---

### 18. Multiple Inventory Sources

Ansible supports multiple inventory files.

Example:

```
inventory = hosts1.ini,hosts2.ini
```

Example use case:

| Inventory | Purpose |
|---|---|
hosts1 | web servers |
hosts2 | database servers |

Both inventories will be loaded.

---

### 19. Using an Inventory Directory

Instead of a single file, inventory can be a directory.

Example structure:

```
inventory/
├── webservers.ini
├── databases.ini
├── staging.ini
```

Configuration:

```
inventory = inventory/
```

Ansible automatically loads all files inside.

This is common in **large infrastructures**.

---

# DYNAMIC INVENTORY

### 20. What is Dynamic Inventory?

Dynamic inventory generates hosts automatically.

Example cloud infrastructure:

```
AWS EC2
```

Instances change frequently.

Example:

```
Server created
Server destroyed
Server scaled
```

Static inventory becomes outdated quickly.

Dynamic inventory solves this.

---

### 21. Example Dynamic Inventory (AWS)

Example plugin:

```
amazon.aws.aws_ec2
```

It automatically discovers:

- EC2 instances
- instance tags
- instance IP addresses

Playbooks automatically target new servers.

---

# REAL WORLD INVENTORY STRUCTURE

### 22. Production Inventory Example

Example project:

```
inventory/
├── dev
│   └── hosts.ini
├── staging
│   └── hosts.ini
├── prod
│   └── hosts.ini
```

Each environment has its own inventory.

Example command:

```
ansible-playbook -i inventory/prod deploy.yml
```

---

### 23. Using Inventory with Playbooks

Example playbook:

```yaml
---
- name: Deploy web server
  hosts: webservers

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

Ansible looks in inventory for:

```
webservers
```

and runs tasks there.

---

# INVENTORY BEST PRACTICES

### 24. Use Group Names Instead of IPs

Avoid targeting raw IP addresses.

Prefer:

```
hosts: webservers
```

instead of:

```
hosts: 192.168.1.10
```

---

### 25. Separate Environments

Maintain separate inventories for:

```
dev
staging
production
```

---

### 26. Use Inventory Directory for Large Systems

Large infrastructures use directory structure:

```
inventory/
├── webservers.ini
├── databases.ini
├── cache.ini
```

---

# COMMON INVENTORY GOTCHAS

### 27. Inventory Path Errors

If Ansible cannot find inventory:

```
[WARNING]: No inventory was parsed
```

Ensure correct path in:

```
ansible.cfg
```

---

### 28. Hostnames Must Be Reachable

Inventory entries must resolve via:

- DNS
- /etc/hosts
- IP addresses

Otherwise SSH will fail.

---

### 29. Avoid Hardcoding Credentials

Use:

```
group_vars/
host_vars/
```

instead of embedding secrets inside inventory.

---

# SUMMARY

Inventory is the **foundation of Ansible automation**.

Key concepts covered:

- inventory defines managed servers
- hosts can be grouped logically
- variables can be defined per host or group
- inventory can be static or dynamic
- default inventory location is `/etc/ansible/hosts`
- inventory path can be configured in `ansible.cfg`

Example configuration:

```
[defaults]
inventory = /path/to/your/inventory
```

Multiple inventories can also be used:

```
inventory = hosts1.ini,hosts2.ini
```

Or inventory directories:

```
inventory = inventory/
```

Understanding inventory allows Ansible to scale from:

- **2 servers**
- to **thousands of servers across multiple cloud environments**

It is one of the most important components of Ansible infrastructure automation.

```
