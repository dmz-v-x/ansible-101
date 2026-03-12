## Setting Up Managed Servers for Ansible — Complete Step-by-Step Guide

Before Ansible can automate servers, those servers must be **prepared and accessible**.  
These servers are called **Managed Nodes** (or **Target Servers**).

Everything will be explained **from absolute beginner level to real-world production setup** in a logical order.

---

### 1. Understanding Managed Servers (Target Nodes)

In an Ansible architecture there are two types of machines:

| Machine | Purpose |
|---|---|
| Control Node | Runs Ansible |
| Managed Node | Servers managed by Ansible |

Managed nodes are the **servers where automation tasks are executed**.

Examples:

- Web servers
- Database servers
- Docker hosts
- Application servers
- Cloud instances

In real production systems these may exist as:

- Cloud VMs (AWS, Azure, GCP)
- DigitalOcean droplets
- Virtual machines
- Bare metal servers
- Containers

For learning purposes we will create **two servers** that Ansible will manage.

---

### 2. Creating Servers (Example: DigitalOcean Droplets)

To practice Ansible automation we typically create multiple servers.

For example:

```
server-1
server-2
```

These servers could be created using:

- DigitalOcean droplets
- AWS EC2 instances
- Local virtual machines
- Vagrant environments

Example DigitalOcean setup:

| Server | Example IP |
|---|---|
| server-1 | 142.93.xx.xx |
| server-2 | 134.209.xx.xx |

These servers will become the **target machines for Ansible**.

---

### 3. Goal: Managing Servers Using Ansible

Once the servers are created, the goal is:

- Connect to them
- Run commands on them
- Install software
- Configure services
- Deploy applications

Without Ansible, the process would be manual:

```
ssh server1
run commands

ssh server2
run commands
```

With Ansible:

```
Run one playbook → configure all servers
```

Ansible automatically connects to all machines and executes tasks.

---

### 4. How Ansible Connects to Servers

Ansible connects to servers using **SSH (Secure Shell)**.

Architecture:

```
Control Node
     |
     | SSH
     |
Managed Servers
```

When you run a playbook:

1. Ansible connects via SSH
2. Sends module code to server
3. Executes module
4. Returns result
5. Removes temporary module

This happens automatically.

Important point:

**No agent needs to be installed on target servers.**

This is why Ansible is called **agentless**.

---

### 5. Requirements for Managed Servers

For Ansible to work properly, target servers must meet some basic requirements.

| Requirement | Why |
|---|---|
| SSH access | For communication |
| Python installed | To run Ansible modules |
| Valid user permissions | To execute commands |

Without these requirements Ansible cannot execute tasks.

---

### 6. Why Python is Required on Linux Servers

Ansible modules are written in **Python**.

When Ansible runs a task:

1. Module code is copied to the server
2. Python executes that module
3. Module performs task
4. Results are returned

Example tasks executed by modules:

- install software
- create directories
- manage services
- modify configuration files

Because modules run using Python, the server must have **Python installed**.

---

### 7. What About Windows Servers?

If the managed server is **Windows**, Ansible works differently.

Instead of SSH + Python, it uses:

- **WinRM** (Windows Remote Management)
- **PowerShell**

So Windows servers require:

- PowerShell
- WinRM enabled

But in most DevOps environments, managed servers are usually **Linux machines**.

---

### 8. Python Availability on Ubuntu Servers

Most Linux distributions already include Python.

Examples:

| OS | Python Included |
|---|---|
| Ubuntu | Yes |
| Debian | Yes |
| CentOS | Yes |
| RedHat | Yes |

Ubuntu servers typically come with **Python3 pre-installed**.

However, it is always good practice to **verify Python installation**.

---

### 9. Checking Python Installation on Server

To verify Python is installed, connect to the server first.

Example:

```
ssh root@server-ip
```

Then check Python version:

```
python --version
```

or

```
python3 --version
```

Example output:

```
Python 3.10.12
```

This confirms Python is installed.

---

### 10. Checking Python Binary Location

Another way to verify Python installation is by checking its binary location.

On most Linux systems, Python binaries are stored in:

```
/usr/bin/
```

Check using:

```
ls /usr/bin/python3
```

Example output:

```
/usr/bin/python3
```

If the binary exists, Python is installed.

---

### 11. Installing Python if Missing

In rare cases Python may not exist on the server.

This can happen with:

- minimal cloud images
- lightweight container OS images

If Python is missing, install it.

Example for Ubuntu:

```
sudo apt update
sudo apt install python3 -y
```

After installation verify again:

```
python3 --version
```

---

### 12. Testing SSH Connectivity

Before using Ansible, verify SSH access.

Example:

```
ssh root@server-ip
```

If login succeeds, the control node can communicate with the server.

Ansible will later use the **same SSH connection method**.

---

### 13. Example Managed Server Setup

After creating servers, the setup may look like this:

```
Control Node
   |
   | SSH
   |
Server-1 (Python installed)
Server-2 (Python installed)
```

Example servers:

```
server-1 → 142.93.xx.xx
server-2 → 134.209.xx.xx
```

These servers will be added to **Ansible inventory later**.

---

### 14. Why Proper Server Setup Matters

Preparing managed servers correctly ensures:

- Playbooks execute successfully
- Modules run without errors
- Automation remains reliable

Common issues when servers are not prepared:

- SSH authentication failures
- Missing Python interpreter
- Permission errors
- Network connectivity issues

Proper preparation avoids these problems.

---

### 15. Summary

Before Ansible can manage servers, those servers must be properly prepared.

Key steps include:

- Create target servers (for example DigitalOcean droplets)
- Ensure SSH access is working
- Ensure Python is installed on Linux machines
- Verify Python using:

```
python --version
```

or

```
ls /usr/bin/python3
```

Important concepts:

- Ansible connects using **SSH**
- No agent installation is required
- Python executes Ansible modules on Linux
- Windows uses **PowerShell and WinRM**

Once managed servers are prepared, the next step is to **add them to Ansible inventory and start running playbooks**.
