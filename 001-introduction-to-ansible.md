## Introduction to Ansible

Infrastructure today is rarely a single server. Modern applications run on **multiple machines**, often across **cloud platforms, containers, and distributed systems**. Managing these machines manually quickly becomes inefficient, slow, and error-prone.

This is where **automation tools** like **Ansible** become extremely important.

Everything will be explained **from beginner to advanced level** in a logical order.

---

### 1. The Problem: Managing Servers Manually

Before understanding Ansible, we first need to understand the **problem it solves**.

In real production systems, companies often have:

- 5 servers  
- 10 servers  
- 100 servers  
- sometimes **thousands of servers**

These servers may run:

- Web applications
- Databases
- Containers
- Background jobs
- Monitoring tools

Now imagine you need to perform tasks such as:

- Install software
- Configure applications
- Deploy a new version of your app
- Update Docker
- Create system users
- Restart services
- Configure security policies

If you do this **manually**, the process usually looks like this:

```
ssh server1
run commands

ssh server2
run commands

ssh server3
run commands

...
```

For example:

```
ssh server1
sudo apt update
sudo apt install docker

ssh server2
sudo apt update
sudo apt install docker
```

Now imagine doing this on **50 servers**.

Problems with manual work:

**1. Extremely time consuming**

Every server requires manual login.

**2. Error-prone**

You might:

- Forget a step
- Run wrong command
- Miss one server

**3. Not repeatable**

If someone asks:

> "How did you configure these servers?"

You may not remember all steps.

**4. Difficult to scale**

Manual operations **do not scale** when infrastructure grows.

Because of these problems, modern DevOps practices rely heavily on **automation**.

---

### 2. What is Ansible?

**Ansible** is an **IT automation tool** used to automate tasks such as:

- Server configuration
- Application deployment
- Infrastructure provisioning
- Software installation
- Security configuration

In simple words:

**Ansible allows you to control many servers from one machine using automation scripts.**

Instead of logging into each server manually, you write instructions once and Ansible executes them on all servers.

Example tasks Ansible can automate:

| Task | Example |
|-----|------|
| Install software | Install Nginx on 50 servers |
| Deploy application | Deploy new version of web app |
| System updates | Update OS packages |
| User management | Create users and assign permissions |
| Infrastructure | Create cloud instances |
| Containers | Start Docker containers |

So instead of running commands manually, we define **automation instructions**.

---

### 3. Why Automation is Important

Automation brings several major benefits.

#### Efficiency

Instead of manually performing repetitive tasks, automation completes them in seconds.

Example:

Manual configuration of 50 servers might take **2 hours**.

Ansible can complete the same job in **less than a minute**.

---

#### Consistency

Humans make mistakes.

Automation ensures:

- Same configuration everywhere
- Same steps executed every time

---

#### Repeatability

Automation scripts act as **documentation of infrastructure**.

If servers crash, you can **rebuild them automatically**.

---

#### Scalability

Automation works for:

- 2 servers
- 20 servers
- 2000 servers

The process remains the same.

---

### 4. Key Advantages of Ansible

Ansible is extremely popular because of several design advantages.

#### 1. Execute tasks from a single machine

You run Ansible commands from your **control machine**.

From this one machine, Ansible connects to multiple servers and performs operations.

---

#### 2. Infrastructure as Code (YAML)

All configuration steps are written in **YAML files**.

Instead of running shell scripts manually, you describe infrastructure in **structured files**.

Example:

```yaml
- name: install nginx
  apt:
    name: nginx
    state: present
```

This makes configuration:

- readable
- version controlled
- shareable across teams

---

#### 3. Reusability

Automation files can be reused.

Example:

Same playbook can configure:

- staging environment
- production environment
- testing environment

---

#### 4. Reliability

Automation reduces human error.

The same configuration runs every time.

---

### 5. Agentless Architecture

One of the biggest advantages of Ansible:

**It is agentless.**

What does that mean?

Many automation tools require installing software on every server.

Example tools that require agents:

- Puppet
- Chef

These tools require:

```
agent software installed on every server
```

But Ansible does **not require any agent**.

Instead, it connects using **SSH**.

```
Control Machine
       |
       | SSH
       |
Target Servers
```

So you only need:

- SSH access
- Python installed on target machines

This makes Ansible:

- easier to deploy
- easier to maintain

---

### 6. How Ansible Works Internally

Now let's understand the **internal architecture**.

Ansible works with a few key components.

```
Control Machine
     |
     | executes playbook
     |
Inventory ---- Playbook ---- Modules
     |
     |
Target Servers
```

Main components:

1. Control Machine  
2. Inventory  
3. Playbooks  
4. Modules  
5. Target Machines

Let's understand each.

---

### 7. Control Machine

The **control machine** is the system where Ansible runs.

This machine:

- contains playbooks
- contains inventory files
- executes automation

It sends instructions to all target machines.

---

### 8. Target Machines

Target machines are servers being managed.

Examples:

- Linux servers
- Cloud instances
- Containers
- Bare metal machines

These machines execute tasks sent by Ansible.

---

### 9. Ansible Modules

Ansible modules are **small programs that perform specific tasks**.

Each module performs **one small operation**.

Examples of modules:

| Module | Purpose |
|------|------|
| file | manage files and directories |
| yum / apt | install packages |
| service | start or stop services |
| user | manage system users |
| docker_container | manage containers |
| ec2 | create AWS instances |

Example:

Create directory:

```yaml
file:
  path: /app/data
  state: directory
```

Install Nginx:

```yaml
yum:
  name: nginx
  state: latest
```

Start service:

```yaml
service:
  name: nginx
  state: started
```

Modules are **granular**, meaning:

> Each module does only one specific task.

Internally Ansible:

1. Pushes module to server
2. Executes module
3. Removes module after execution

---

### 10. What is a Playbook?

A **Playbook** is the core of Ansible automation.

A playbook is a **YAML file that describes automation tasks**.

It defines:

- what tasks should run
- on which servers
- in what order

Structure:

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
|---|---|
| Playbook | full automation file |
| Play | group of tasks executed on hosts |
| Task | individual step |
| Module | actual action |

---

### 11. Example Playbook

Example configuration for installing and starting Nginx.

```yaml
tasks:
  - name: create directory for nginx
    file:
      path: /path/to/nginx/dir
      state: directory

  - name: install nginx latest version
    yum:
      name: nginx
      state: latest

  - name: start nginx
    service:
      name: nginx
      state: started
```

This configuration performs three steps:

1. Create directory
2. Install nginx
3. Start nginx service

Each step is called a **task**.

Each task uses a **module**.

---

### 12. Defining Target Servers (hosts)

Playbooks must specify **which machines should run tasks**.

This is done using the `hosts` attribute.

Example:

```yaml
- hosts: databases
```

This means:

> Run these tasks on all servers belonging to the **databases group**.

---

### 13. Executing Tasks as Specific User

Sometimes tasks must run with specific privileges.

Example:

```yaml
remote_user: root
```

This means Ansible will connect as **root user**.

But you can also run tasks as normal users.

---

### 14. Example Database Play

```yaml
- hosts: databases
  remote_user: root

  tasks:
    - name: Rename table foo to bar
      postgresql_table:
        table: foo
        rename: bar

    - name: Set owner to someuser
      postgresql_table:
        name: foo
        owner: someuser

    - name: Truncate table foo
      postgresql_table:
        name: foo
        truncate: yes
```

This play performs database operations on all servers in the **databases group**.

---

### 15. Variables in Ansible

Often we repeat the same values multiple times.

Example:

```
tablename = foo
```

Instead of repeating this everywhere, we use **variables**.

Example:

```yaml
vars:
  tablename: foo
  tableowner: someuser
```

Then reference them like this:

```
{{ tablename }}
```

Example:

```yaml
name: Rename table {{ tablename }} to bar
```

Variables make playbooks:

- cleaner
- reusable
- easier to maintain

---

### 16. Ansible Inventory

Now a critical question arises:

**Where does Ansible get the list of servers from?**

The answer is **Inventory**.

Inventory is a file that contains **all servers managed by Ansible**.

Example:

```
10.24.0.100

[webservers]
10.24.0.1
10.24.0.2

[databases]
10.24.0.7
10.24.0.8
```

Explanation:

```
[webservers]
10.24.0.1
10.24.0.2
```

This defines a **group of servers** called `webservers`.

Similarly:

```
[databases]
10.24.0.7
10.24.0.8
```

These groups are referenced in playbooks.

---

### 17. Infrastructure Automation with Ansible

Ansible is not limited to simple tasks.

It can automate:

- server provisioning
- cloud infrastructure
- container orchestration
- application deployments

Example tasks:

- Create AWS EC2 instance
- Configure Kubernetes cluster
- Deploy Docker containers
- Setup monitoring tools

---

### 18. Ansible with Docker

Normally when creating Docker containers we write a **Dockerfile**.

Dockerfile prepares environment:

- copy application artifact
- configure environment variables
- start application

Example:

```
COPY app.jar
ENV DB_HOST=db
CMD run.sh
```

But Ansible can perform the same configuration.

Instead of only creating containers, Ansible can configure:

- Docker containers
- Host machines
- Network
- Storage
- Cloud infrastructure

So Ansible can manage **both containers and hosts together**.

---

### 19. Ansible Tower

Ansible Tower is a **web UI for managing Ansible automation**.

It is developed by **Red Hat**.

Tower provides:

- Web dashboard
- Centralized playbook management
- Role-based access control
- Job scheduling
- Team collaboration

Example use case:

In large organizations:

- DevOps team creates automation
- Multiple teams reuse playbooks

Tower helps manage automation centrally.

---

### 20. Ansible vs Puppet vs Chef

Several automation tools exist.

The most popular ones are:

- Ansible
- Puppet
- Chef

Comparison:

| Feature | Ansible | Puppet / Chef |
|---|---|---|
| Language | YAML | Ruby |
| Learning curve | Easy | Harder |
| Agent | Agentless | Requires agent |
| Setup | Simple | Complex |
| Maintenance | Minimal | Requires agent updates |

Because of simplicity, Ansible has become **extremely popular in DevOps workflows**.

---

### 21. Summary

We covered the **complete foundation of Ansible**.

Key takeaways:

- Ansible is an **IT automation tool**
- It automates server configuration and deployment
- Uses **YAML playbooks**
- Works through **SSH (agentless)**
- Uses **modules to perform tasks**
- Groups servers using **inventory**
- Executes tasks sequentially using **playbooks**

Ansible enables teams to manage infrastructure as **code**, making systems:

- repeatable
- scalable
- reliable
- automated

It is now one of the **core tools in modern DevOps environments**.
