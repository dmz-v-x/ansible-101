## Introduction to Ansible Playbooks — From Absolute Beginner to Real-World Infrastructure Automation

In the previous sections we learned:

- What **Ansible** is
- How **inventory** works
- How to connect to servers using **SSH**
- How to run **ad-hoc commands**
- How to manage servers on **DigitalOcean and AWS**

Now we move to the **most important concept in Ansible**:

**Ansible Playbooks**

Playbooks are what make Ansible powerful enough to automate **real production infrastructure**.

---

### 1. Real-World Scenario (Why Playbooks Exist)

Imagine a company deploying a **web application**.

Infrastructure:

```
Internet
   |
Load Balancer
   |
Web Servers
   |        |
Server1   Server2
```

Each server must have:

- Nginx installed
- Nginx service started
- Same configuration

If we do this manually:

```
ssh server1
install nginx
start nginx

ssh server2
install nginx
start nginx
```

Problems:

- repetitive work
- error prone
- not scalable
- no documentation

Using **Ansible playbooks**, we define the infrastructure as code.

Example result:

```
Run playbook once → configure all servers automatically
```

---

### 2. Ansible as Infrastructure as Code (IaC)

Ansible is an **Infrastructure as Code (IaC) tool**.

Infrastructure as Code means:

Infrastructure configuration is written as **code files**.

Example:

```
install nginx
start nginx
configure firewall
```

These steps are written in a file and stored like software code.

Benefits:

| Benefit | Explanation |
|------|------|
| Version control | Infrastructure tracked in Git |
| Automation | No manual configuration |
| Reproducibility | Same setup everywhere |
| Documentation | Code explains infrastructure |

This is why **Ansible configuration files are treated like code**.

---

### 3. Creating an Ansible Project

In real environments we organize Ansible code inside a project.

Create a project directory:

```
mkdir ansible
cd ansible
```

Typical project structure:

```
ansible/
│
├── hosts
├── ansible.cfg
└── my-playbook.yaml
```

Explanation:

| File | Purpose |
|----|----|
| hosts | inventory file |
| ansible.cfg | configuration |
| playbook | automation tasks |

Projects are usually stored in **Git repositories**.

Example:

```
git init
git add .
git commit -m "initial ansible automation"
```

---

### 4. Minimum Requirements for an Ansible Playbook

Every playbook requires **two things**.

### 1 Managed Nodes

Servers that Ansible will configure.

Example:

```
webserver1
webserver2
```

These are defined in the **inventory file**.

---

### 2 At Least One Task

A playbook must perform at least one action.

Examples:

- install software
- copy files
- start services
- deploy application

---

### 5. Creating the Inventory File

First we create the inventory file called **hosts**.

```
vim hosts
```

Example:

```hosts
[webserver]
157.230.29.95
157.230.29.113

[webserver:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=root
```

Explanation:

### Group

```
[webserver]
```

This defines a group of servers.

### Servers

```
157.230.29.95
157.230.29.113
```

These are the machines Ansible will manage.

### Group Variables

```
[webserver:vars]
```

These variables apply to all hosts in the group.

Variables used:

```
ansible_ssh_private_key_file
ansible_user
```

This tells Ansible:

```
ssh -i ~/.ssh/id_rsa root@server
```

---

### 6. Why Every Ansible Project Has an Inventory

Ansible always interacts with **multiple machines**.

Examples:

- web servers
- database servers
- cache servers

So inventory is always present in Ansible projects.

Example infrastructure:

```
[webserver]
web1
web2

[database]
db1
```

---

### 7. What is an Ansible Playbook?

A **Playbook** is a configuration file that defines:

- which servers to configure
- what tasks to run
- in what order

A playbook is written in **YAML**.

Key characteristics:

| Feature | Explanation |
|------|------|
| YAML format | Human readable |
| Ordered execution | Tasks run sequentially |
| Multiple plays | One playbook can configure multiple server types |

---

### 8. Playbook Structure

Structure hierarchy:

```
Playbook
   |
   |-- Play
         |
         |-- Tasks
               |
               |-- Modules
```

Explanation:

| Component | Description |
|------|------|
| Playbook | entire automation file |
| Play | group of tasks executed on hosts |
| Task | single step |
| Module | actual action |

---

### 9. Creating the First Playbook

Create the file:

```
vim my-playbook.yaml
```

Example playbook:

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: Install Nginx Server
      apt:
        name: nginx
        state: latest

    - name: Start Nginx Server
      service:
        name: nginx
        state: started
---
```

---

### 10. Understanding the Playbook Line by Line

### YAML Start

```
---
```

This indicates the beginning of a YAML document.

---

### Play Name

```
- name: Configure nginx web server
```

This is optional but recommended.

It describes what the play does.

---

### Hosts

```
hosts: webserver
```

This tells Ansible:

```
Run tasks on servers in the webserver group
```

---

### Tasks Section

```
tasks:
```

Defines a list of tasks.

Tasks execute **top to bottom**.

---

### Task 1 — Install Nginx

```
- name: Install Nginx Server
```

Task description.

Module used:

```
apt
```

Arguments:

```
name: nginx
state: latest
```

Meaning:

Install latest version of nginx.

---

### Task 2 — Start Nginx

Module:

```
service
```

Arguments:

```
name: nginx
state: started
```

Meaning:

Start nginx service.

---

### 11. Creating Project Specific Ansible Configuration

Each project can have its own configuration.

Create file:

```
vim ansible.cfg
```

Example:

```ini
[defaults]
host_key_checking = False
```

This disables SSH host key prompt for this project.

Project structure now:

```
ansible/
│
├── hosts
├── ansible.cfg
└── my-playbook.yaml
```

---

### 12. Running the Playbook

Command:

```
ansible-playbook -i <hostfile> <playbookfile>
```

Example:

```
ansible-playbook -i hosts my-playbook.yaml
```

Explanation:

| Parameter | Meaning |
|------|------|
| ansible-playbook | playbook runner |
| -i hosts | inventory file |
| my-playbook.yaml | playbook file |

Ansible will:

1. connect to servers
2. gather system info
3. execute tasks
4. show results

---

### 13. The Gather Facts Step

When a playbook starts, Ansible automatically runs:

```
Gathering Facts
```

This uses the **setup module**.

Example output:

```
TASK [Gathering Facts]
ok: [157.230.29.95]
ok: [157.230.29.113]
```

---

### 14. What Gather Facts Does

The **Gather Facts module** collects system information.

Examples of collected data:

| Information | Example |
|------|------|
| OS type | Ubuntu |
| Kernel version | 5.x |
| CPU info | cores |
| Memory | RAM |
| Network | interfaces |
| hostname | server name |

These facts become variables usable inside playbooks.

Example variable:

```
ansible_distribution
```

Example use:

```
if OS == Ubuntu install apt packages
```

---

### 15. Verifying Nginx Installation

After playbook runs we can verify.

Connect to server:

```
ssh root@157.230.29.95
```

Check process:

```
ps aux | grep nginx
```

You should see nginx running.

---

### 16. Installing Specific Version of Nginx

Sometimes we need a specific version.

Updated playbook:

```yaml
---
- name: Configure nginx web server
  hosts: webserver
  tasks:
    - name: install nginx server
      apt:
        name: nginx=1.24.0-1ubuntu1
        state: present

    - name: start nginx server
      service:
        name: nginx
        state: started
---
```

This installs an exact version.

---

### 17. Understanding Idempotency

One of the most important properties of Ansible:

**Idempotency**

Definition:

Running the same playbook multiple times produces the **same final state**.

Example:

Desired state:

```
nginx installed
nginx running
```

First run:

```
nginx installed
nginx started
```

Second run:

```
nothing changes
```

Because the desired state already exists.

Formula:

```
Actual State == Desired State
```

Then:

```
No change required
```

This prevents unnecessary operations.

---

### 18. Removing Nginx with Ansible

We can also uninstall services.

Example playbook:

```yaml
---
- name: Remove nginx web server
  hosts: webserver
  tasks:
    - name: uninstall nginx server
      apt:
        name: nginx
        state: absent

    - name: stop nginx server
      service:
        name: nginx
        state: stopped
---
```

Meaning:

```
remove nginx package
stop service
```

---

### 19. Real-World Deployment Workflow

In real DevOps workflows:

-> Infrastructure created (AWS / DigitalOcean)

-> Servers added to inventory

-> Playbooks written

-> Code stored in Git

-> Automation executed

Example CI/CD pipeline:

```
Git push
   |
CI pipeline
   |
Run ansible-playbook
   |
Servers configured automatically
```

---

### 20. Complete Workflow Summary

We built a complete Ansible automation flow.

Step-by-step process:

-> Create servers

-> Install Ansible

-> Configure SSH access

-> Create inventory file

-> Create Ansible project

-> Write playbook

-> Run playbook

-> Verify server configuration

Final project:

```
ansible/
│
├── hosts
├── ansible.cfg
└── my-playbook.yaml
```

Using this approach we can automate:

- web servers
- databases
- containers
- cloud infrastructure

Playbooks allow infrastructure to be managed as **code**, making systems scalable, repeatable, and reliable.
