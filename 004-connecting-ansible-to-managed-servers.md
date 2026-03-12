## Connecting Ansible to Managed Servers — Inventory, Ad-Hoc Commands, and Host Grouping

After installing Ansible and preparing managed servers, the next step is to **connect Ansible to those servers** so it can manage them.

To do that we must understand several important concepts:

- What **Ansible Inventory** is
- What **Ad-Hoc Commands** are
- How Ansible authenticates to servers
- How to organize servers using **groups**
- Commonly used **Ansible ad-hoc commands**
- How to write a clean and maintainable inventory file

This guide explains everything **step-by-step from beginner to advanced level**, following the same logical approach as previous sections.

---

### 1. The Problem: How Does Ansible Know Which Servers to Manage?

When we run an Ansible command, Ansible needs to know:

- Which servers should receive the commands?
- How should it connect to them?
- Which credentials should be used?

Example:

```
ansible ??? -m ping
```

But **which machines should Ansible ping?**

To answer this, Ansible uses something called an **Inventory**.

---

## Understanding Ansible Inventory

### 2. What is Ansible Inventory?

An **Inventory** is a file that contains information about the servers that Ansible will manage.

These servers are called:

- hosts
- managed nodes
- target machines

Inventory contains:

- IP addresses or hostnames of servers
- authentication details
- grouping information
- variables related to servers

In simple terms:

> Inventory tells Ansible **where to run automation tasks**.

---

### 3. Default Inventory Location

Ansible already has a default inventory file location.

```
/etc/ansible/hosts
```

However, in many projects developers prefer to create their **own inventory file** inside the project directory.

Example:

```
vim hosts
```

This creates a custom inventory file called `hosts`.

---

### 4. Adding Servers to the Inventory File

Suppose we created two cloud servers (DigitalOcean droplets):

```
157.230.29.95
157.230.29.113
```

To allow Ansible to manage them, we add these IP addresses into the inventory file.

Example inventory:

```
157.230.29.95
157.230.29.113
```

Now Ansible knows:

> These are the servers that must be managed.

But Ansible still cannot connect yet because it does not know:

- which user to log in as
- which SSH key to use

---

### 5. Authentication: How Ansible Connects to Servers

Ansible uses **SSH authentication** to connect to servers.

Typical SSH connection looks like this:

```
ssh root@157.230.29.95
```

Sometimes we specify a private key:

```
ssh -i ~/.ssh/id_rsa root@157.230.29.95
```

So Ansible must also know:

- SSH private key
- username

These are defined inside the **inventory file**.

---

### 6. Specifying SSH Private Key in Inventory

To tell Ansible which private key to use, we use the variable:

```
ansible_ssh_private_key_file
```

Example:

```
157.230.29.95 ansible_ssh_private_key_file=~/.ssh/id_rsa
157.230.29.113 ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Now Ansible knows:

- which key to use when connecting

---

### 7. Specifying the SSH User

Just like SSH commands require a user:

```
ssh root@server-ip
```

Ansible also needs the username.

This is defined using:

```
ansible_user
```

Example:

```
157.230.29.95 ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=root
157.230.29.113 ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=root
```

Now Ansible knows:

- which server
- which user
- which SSH key

---

### 8. Complete Inventory Example

```
157.230.29.95 ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=root
157.230.29.113 ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=root
```

Once this file is saved, Ansible can connect to both servers.

---

# Understanding Ansible Ad-Hoc Commands

### 9. What Are Ansible Ad-Hoc Commands?

Ansible **Ad-Hoc Commands** are quick one-line commands used to perform simple tasks on servers.

They are usually used for:

- quick operations
- testing connectivity
- running one-time commands
- troubleshooting

Important property:

> Ad-hoc commands are **not stored for reuse**.

They are temporary commands executed directly from the terminal.

Example:

```
ansible all -m ping
```

This command checks connectivity with all servers.

---

### 10. General Syntax of Ad-Hoc Commands

The general structure of an Ansible ad-hoc command is:

```
ansible [pattern] -i [inventory] -m [module] -a "[arguments]"
```

Explanation:

| Part | Meaning |
|-----|------|
| ansible | command |
| pattern | which hosts to target |
| -i | inventory file |
| -m | module |
| -a | module arguments |

Example:

```
ansible all -i hosts -m ping
```

---

### 11. First Ad-Hoc Command — Ping

Example:

```
ansible all -i hosts -m ping
```

Explanation:

| Part | Meaning |
|---|---|
| all | target all hosts |
| -i hosts | use hosts file |
| -m ping | use ping module |

Output:

```
157.230.29.95 | SUCCESS
157.230.29.113 | SUCCESS
```

Important note:

This is **not an ICMP ping**.

Instead, Ansible ping verifies:

- SSH connection works
- Python is available
- Ansible modules can run

---

# Commonly Used Ansible Ad-Hoc Commands

Below are some **frequently used ad-hoc commands** in real environments.

---

### 12. Run Shell Commands

Execute a command on remote servers.

```
ansible all -i hosts -m shell -a "uptime"
```

Example output:

```
157.230.29.95 | SUCCESS | uptime
157.230.29.113 | SUCCESS | uptime
```

---

### 13. Run Command Module

Safer alternative to shell.

```
ansible all -i hosts -m command -a "df -h"
```

Shows disk usage.

---

### 14. Install Packages

Using apt module:

```
ansible all -i hosts -m apt -a "name=nginx state=present"
```

Installs nginx.

---

### 15. Copy Files to Servers

```
ansible all -i hosts -m copy -a "src=file.txt dest=/tmp/file.txt"
```

Copies file from control node to servers.

---

### 16. Create Directories

```
ansible all -i hosts -m file -a "path=/app state=directory"
```

Creates directory.

---

### 17. Manage Services

```
ansible all -i hosts -m service -a "name=nginx state=started"
```

Starts nginx service.

---

### 18. Reboot Servers

```
ansible all -i hosts -m reboot
```

Reboots machines.

---

### 19. Gather System Information

```
ansible all -i hosts -m setup
```

Displays server facts such as:

- OS
- CPU
- Memory
- Network

---

# Targeting Specific Hosts

Ansible allows targeting different sets of servers.

---

### 20. Target All Servers

```
ansible all -i hosts -m ping
```

Targets every host in inventory.

---

### 21. Target Single Server

Example:

```
ansible 157.230.29.95 -i hosts -m ping
```

Only tests connection to that server.

---

### 22. Target Groups of Servers

Instead of targeting individual servers, we can group them.

This becomes very useful in real environments.

Example environments:

| Group Type | Example |
|---|---|
| Cloud platform | aws, azure |
| Role | database, web |
| Region | us-east, asia |
| Environment | dev, staging, prod |

---

# Creating Host Groups

### 23. Grouping Servers in Inventory

Example inventory:

```
[droplet]
157.230.29.95
157.230.29.113
```

Now both servers belong to the group:

```
droplet
```

---

### 24. Running Commands on Groups

Example:

```
ansible droplet -i hosts -m ping
```

This command targets only servers in **droplet group**.

---

### 25. Example of Multi-Group Inventory

```
[droplet]
157.230.29.95
157.230.29.113

[aws]
3.20.xx.xx
3.40.xx.xx

[webservers]
10.0.1.10
10.0.1.11

[databases]
10.0.2.10
```

Now Ansible can run tasks on:

- droplet servers
- AWS servers
- web servers
- database servers

---

# Using Group Variables

### 26. The Problem with Repeating Configuration

Earlier we wrote:

```
157.230.29.95 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
157.230.29.113 ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_rsa
```

This repeats the same values.

This is not ideal.

---

### 27. Using Group Variables

Instead we define variables for the group.

Example:

```
[droplet]
157.230.29.95
157.230.29.113

[droplet:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=root
```

Now both servers automatically inherit these values.

Advantages:

- cleaner inventory
- easier to maintain
- less repetition

---

### 28. Final Clean Inventory Example

```
[droplet]
157.230.29.95
157.230.29.113

[droplet:vars]
ansible_user=root
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Now running:

```
ansible droplet -i hosts -m ping
```

Will connect to both servers using the same credentials.

---

### Summary

We covered the entire process of connecting Ansible to managed servers.

Key concepts:

**Inventory**

- Defines servers managed by Ansible
- Default location `/etc/ansible/hosts`
- Contains IPs, groups, and variables

**Ad-Hoc Commands**

- Quick one-time commands
- Not stored for reuse
- Useful for testing and troubleshooting

Example syntax:

```
ansible [pattern] -i [inventory] -m [module] -a "[arguments]"
```

**Targeting Options**

Ansible can target:

- individual servers
- groups of servers
- all servers

**Grouping Servers**

Servers can be grouped by:

- platform
- region
- role
- environment

Using groups makes infrastructure **organized and scalable**.

Once inventory and connectivity are working, the next step is to start writing **Ansible Playbooks for real automation**.
