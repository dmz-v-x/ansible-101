## Installing Ansible — Complete Guide from Absolute Beginner to Production Setup

Before we can start automating infrastructure using Ansible, we first need to **install and configure Ansible correctly**.

However, installing Ansible is not just about running a single command. To understand the installation process properly, we must first understand:

- Where Ansible should be installed
- What a **Control Node** is
- What **Target Nodes** are
- Different real-world deployment setups
- System requirements for running Ansible

---

### 1. Understanding the Ansible Architecture First

Before installing Ansible, we must understand **where Ansible actually runs**.

Ansible operates using a **control machine → target machines architecture**.

```
Control Node (runs Ansible)
        |
        | SSH
        |
Target Servers (managed by Ansible)
```

So there are two types of machines involved:

| Machine Type | Purpose |
|---|---|
| Control Node | Runs Ansible and sends instructions |
| Target Nodes | Servers where tasks are executed |

The **Control Node** runs playbooks and connects to target machines using **SSH**.

The **Target Nodes** execute the tasks sent by Ansible modules.

Important concept:

> Ansible itself only needs to be installed on the **Control Node**.

Target machines **do NOT require Ansible installation**.

---

### 2. What is the Control Node?

The **Control Node** is the machine where Ansible is installed and executed.

This machine:

- stores Ansible playbooks
- stores inventory files
- executes automation commands
- connects to remote machines via SSH

Example responsibilities of control node:

- Running `ansible-playbook`
- Managing inventories
- Running automation scripts
- Installing packages on remote servers
- Deploying applications

In simple terms:

> The Control Node is the **central automation machine**.

---

### 3. Control Node Operating System Requirement

An important limitation:

**Windows is not supported as an Ansible Control Node.**

Ansible control nodes must run **Unix-like systems**.

Supported systems include:

- Ubuntu
- Debian
- CentOS
- RedHat
- Fedora
- macOS
- WSL (Windows Subsystem for Linux)

Why?

Because Ansible relies heavily on:

- SSH
- Python
- Unix tools

These are native to Linux environments.

---

### 4. Target Nodes (Servers Being Managed)

Target nodes are the machines Ansible manages.

Examples include:

- Application servers
- Database servers
- Cloud instances
- Container hosts
- Virtual machines

These machines **do not need Ansible installed**.

They only need:

- SSH access
- Python installed

Example target nodes:

```
web-server-1
web-server-2
database-server
cache-server
```

Ansible connects to them and runs modules remotely.

---

### 5. Two Common Locations Where Ansible Can Be Installed

There are **two main ways** organizations install and run Ansible.

Both approaches use the same concept but differ in **where the control node is located**.

---

### 6. Option 1 — Installing Ansible on Your Local Machine

This is the **most common setup for learning and development**.

In this setup:

```
Developer Laptop (Control Node)
        |
        | SSH
        |
Remote Servers (Target Nodes)
```

Workflow:

1. Install Ansible on your laptop
2. Configure inventory of servers
3. Write playbooks locally
4. Run playbooks from your laptop
5. Ansible connects to servers via SSH

Example workflow:

```
ansible-playbook deploy.yml
```

Ansible then connects to all target servers and executes tasks.

Advantages of this setup:

- Easy to set up
- Good for development
- Fast iteration
- No extra infrastructure required

Example use case:

A DevOps engineer automating deployments for:

- staging servers
- development environments
- test clusters

---

### 7. Option 2 — Installing Ansible on a Remote Server (Central Automation Server)

In production environments, Ansible is often installed on a **dedicated automation server**.

This setup looks like:

```
Developer Machine
        |
        | SSH
        |
Automation Server (Control Node with Ansible)
        |
        | SSH
        |
Private Servers
```

This server is often called:

- Automation server
- Bastion host
- Jump server

---

### 8. Why Use a Remote Control Node?

This setup is useful when servers exist in a **private network**.

Example cloud architecture:

```
Internet
   |
Public Subnet
   |
Bastion / Automation Server (Ansible installed)
   |
Private Subnet
   |
Application Servers
Database Servers
Internal Services
```

In this architecture:

- Private servers **cannot be accessed directly from internet**
- Only the **bastion host** can access them

So the workflow becomes:

1. Connect to bastion host
2. Run Ansible from bastion
3. Bastion connects to private servers

Advantages:

- Improved security
- Centralized automation
- Team collaboration
- Controlled access to infrastructure

This setup is **very common in cloud environments**.

---

### 9. Ansible Installation Process

Now that we understand the architecture, we can install Ansible.

The installation process is straightforward.

Example for **Ubuntu / Debian systems**.

---

### 10. Step 1 — Update System Packages

Before installing any software, update package lists.

```
sudo apt update
```

This ensures the package manager knows the latest versions.

---

### 11. Step 2 — Install Ansible

Install Ansible using the package manager.

```
sudo apt -y install ansible
```

Explanation:

- `apt` → package manager
- `install` → install software
- `ansible` → package name
- `-y` → automatically confirm installation

After installation, Ansible binaries will be available on the system.

---

### 12. Step 3 — Verify Installation

After installing Ansible, verify it using:

```
ansible --version
```

Example output:

```
ansible [core 2.x]
python version = 3.x
```

This confirms:

- Ansible installed successfully
- Python environment detected
- Version information

---

### 13. Ansible System Requirements

For Ansible to work properly, certain requirements must be satisfied.

---

#### Requirement 1 — Python on Control Node

Ansible is written in **Python**.

Therefore Python must be installed on the control machine.

Most Linux distributions already include Python.

Check using:

```
python3 --version
```

If Python is missing:

```
sudo apt install python3
```

---

#### Requirement 2 — Python on Target Nodes

Target nodes must also have Python installed.

Why?

Because Ansible modules are executed using Python.

When Ansible connects to a server:

1. It copies the module to the server
2. Python runs the module
3. The module performs the task
4. Results are returned

Without Python, modules cannot execute.

---

#### Requirement 3 — SSH Access

Ansible connects to servers using **SSH**.

Therefore:

- SSH must be enabled on target machines
- Control node must have access credentials

Example connection test:

```
ssh user@server-ip
```

If SSH works, Ansible will also work.

---

### 14. Testing Ansible Installation

After installing Ansible, we should test connectivity.

Example:

```
ansible all -m ping
```

Explanation:

| Part | Meaning |
|---|---|
| ansible | command |
| all | target group |
| -m ping | run ping module |

Output example:

```
server1 | SUCCESS
server2 | SUCCESS
```

This confirms Ansible can communicate with target machines.

---

### 15. Typical Ansible Environment Layout

A typical Ansible project directory looks like:

```
ansible-project/
│
├── inventory
├── playbook.yml
├── roles/
├── group_vars/
└── host_vars/
```

Components:

| File | Purpose |
|---|---|
| inventory | list of servers |
| playbook.yml | automation instructions |
| roles | reusable automation modules |
| group_vars | variables for server groups |
| host_vars | variables for specific hosts |

---

### 16. Best Practices for Installing Ansible

When setting up Ansible in production:

1. Use a dedicated automation server  
2. Use SSH key authentication  
3. Store playbooks in Git  
4. Separate development and production inventories  
5. Use environment-specific configuration  

Example environments:

```
inventory_dev
inventory_stage
inventory_prod
```

---

### 17. Summary

Installing Ansible requires understanding the **control node and target node architecture**.

Key points:

- Ansible runs on a **Control Node**
- Control node manages **Target Nodes**
- Target nodes **do not require Ansible installation**
- Python must exist on both machines
- Communication happens using **SSH**

Two common deployment setups exist:

1. Local machine control node (development)
2. Remote automation server (production)

Installation is simple:

```
sudo apt update
sudo apt -y install ansible
ansible --version
```

Once installed, Ansible can automate tasks across **multiple servers simultaneously**, making infrastructure management efficient, consistent, and scalable.
