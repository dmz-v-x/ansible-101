## Ansible Modules and Collections 

In the previous sections we learned:

- What **Ansible** is
- How to create **inventory files**
- How to connect to servers using **SSH**
- How to run **ad-hoc commands**
- How to create and run **playbooks**
- How Ansible automates infrastructure

Now we move to one of the **core building blocks of Ansible automation**:

**Modules and Collections**

Understanding these concepts is critical because:

> Every task you run in Ansible ultimately executes a **module**.

---

### 1. Real-World Scenario: Automating Web Infrastructure

Imagine a company deploying a web application.

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

Each server must perform tasks such as:

- Install Nginx
- Start Nginx service
- Create application directories
- Copy configuration files
- Deploy application code

Without automation you would run commands manually:

```
ssh server1
apt install nginx
systemctl start nginx

ssh server2
apt install nginx
systemctl start nginx
```

Instead, Ansible automates these tasks.

But how does Ansible perform them?

The answer is:

**Modules**

---

### 2. What is an Ansible Module?

An **Ansible Module** is a small program that performs a specific task on a remote server.

Examples of tasks modules can perform:

- Install software
- Create directories
- Start services
- Copy files
- Manage users
- Configure firewall
- Create cloud instances

In simple terms:

> Modules are the **actual workers** that perform automation tasks.

---

### 3. How Modules Work Internally

When you run a playbook:

```
ansible-playbook deploy.yml
```

Ansible performs the following process:

```
Control Node
      |
      | SSH
      |
Remote Server
```

Steps internally:

1. Ansible connects to the server via SSH
2. Ansible transfers the module code
3. Python executes the module
4. Module performs the task
5. Result is returned
6. Temporary files are removed

Important:

Modules are **temporary**.

They are not permanently installed on the server.

---

### 4. Modules are Very Granular

Ansible modules are designed to do **one small task only**.

Example tasks:

| Module | Purpose |
|------|------|
file | create/delete directories |
apt | install packages |
service | manage services |
copy | copy files |
user | manage users |

Example tasks in a playbook:

```
install nginx
start nginx
create directory
copy config file
```

Each step uses a **different module**.

---

### 5. Example Modules in Action

Example playbook:

```yaml
---
- name: Configure Web Server
  hosts: webserver

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Create web directory
      file:
        path: /var/www/app
        state: directory

    - name: Start nginx
      service:
        name: nginx
        state: started
```

Modules used here:

| Task | Module |
|----|----|
Install nginx | apt |
Create directory | file |
Start service | service |

---

### 6. Commonly Used Ansible Modules

Below are modules commonly used in real production environments.

---

## Package Management Modules

Install software packages.

Examples:

| OS | Module |
|---|---|
Ubuntu/Debian | apt |
RedHat/CentOS | yum |
Generic | package |

Example:

```yaml
apt:
  name: nginx
  state: present
```

---

## Service Management Module

Control services.

Example:

```yaml
service:
  name: nginx
  state: started
```

Possible states:

| State | Meaning |
|---|---|
started | start service |
stopped | stop service |
restarted | restart service |

---

## File Module

Used to manage files and directories.

Example:

```yaml
file:
  path: /var/www/app
  state: directory
```

Possible states:

| State | Meaning |
|---|---|
directory | create directory |
file | create file |
absent | delete |

---

## Copy Module

Copy files from control node to remote server.

Example:

```yaml
copy:
  src: index.html
  dest: /var/www/html/index.html
```

---

## User Module

Manage system users.

Example:

```yaml
user:
  name: devuser
  state: present
```

---

### 7. Using Modules in Ad-Hoc Commands

Modules can also be used outside playbooks.

Example:

```
ansible webserver -i hosts -m apt -a "name=nginx state=present"
```

Explanation:

| Parameter | Meaning |
|---|---|
-m | module |
-a | arguments |

This installs nginx on webservers.

---

### 8. Where Modules Come From

Ansible includes **hundreds of modules by default**.

Examples:

- Linux system modules
- Cloud modules
- Docker modules
- Database modules

However, as infrastructure grew more complex, managing modules became difficult.

This led to the introduction of:

**Collections**

---

### 9. What is an Ansible Collection?

A **Collection** is a package that contains:

- modules
- plugins
- roles
- documentation

Collections allow Ansible to organize modules logically.

Example collection:

```
amazon.aws
```

Contains modules for:

- EC2
- VPC
- S3
- IAM

---

### 10. Why Collections Were Introduced

Earlier versions of Ansible had all modules in one huge repository.

Problems:

- difficult to maintain
- slow updates
- cloud modules mixed with system modules

Collections solved this by grouping modules by domain.

Example:

| Collection | Purpose |
|---|---|
amazon.aws | AWS automation |
community.docker | Docker automation |
kubernetes.core | Kubernetes automation |

---

### 11. Collection Structure

A collection contains multiple components.

Example structure:

```
collection/
   |
   |-- modules
   |-- roles
   |-- plugins
   |-- docs
```

Modules inside collections are accessed using **fully qualified names**.

Example:

```
amazon.aws.ec2_instance
```

---

### 12. Installing Ansible Collections

Collections are installed using:

```
ansible-galaxy
```

Example:

```
ansible-galaxy collection install amazon.aws
```

This installs AWS modules.

---

### 13. Listing Installed Collections

To see installed collections:

```
ansible-galaxy collection list
```

Example output:

```
amazon.aws
community.docker
ansible.posix
```

---

### 14. Using Modules from Collections

Example: creating an EC2 instance.

```yaml
---
- name: Create AWS instance
  hosts: localhost

  tasks:
    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: my-server
        instance_type: t2.micro
```

Here:

```
amazon.aws.ec2_instance
```

is the module.

---

### 15. Real-World Automation Scenario

Let's combine everything we learned.

Scenario:

Deploy a web server infrastructure automatically.

Requirements:

- Install nginx
- Create application directory
- Copy web page
- Start nginx

---

### Complete Playbook

```yaml
---
- name: Deploy Web Application
  hosts: webserver

  tasks:

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Create application directory
      file:
        path: /var/www/app
        state: directory

    - name: Copy website file
      copy:
        src: index.html
        dest: /var/www/app/index.html

    - name: Start nginx
      service:
        name: nginx
        state: started
```

Modules used:

| Task | Module |
|---|---|
Install nginx | apt |
Create directory | file |
Copy webpage | copy |
Start nginx | service |

---

### 16. Running the Playbook

Command:

```
ansible-playbook -i hosts deploy.yml
```

Result:

All servers are automatically configured.

---

### 17. Why Modules Make Ansible Powerful

Modules provide several advantages:

### Simplicity

Modules abstract complex commands.

Example:

Instead of:

```
apt-get install nginx
```

We write:

```
apt:
  name: nginx
```

---

### Idempotency

Modules check the current state before making changes.

Example:

If nginx already exists, the module does nothing.

---

### Cross-Platform Support

Modules work across multiple operating systems.

Example:

```
package module
```

works on:

- Ubuntu
- CentOS
- RedHat

---

### 18. Summary

Modules and collections are the **core execution engine of Ansible**.

Key points:

### Modules

- Small programs that perform tasks
- Execute on remote machines
- Run through playbooks or ad-hoc commands
- Each module performs one action

### Collections

- Packages containing modules and plugins
- Organize modules by domain
- Installed using `ansible-galaxy`

Example collections:

```
amazon.aws
community.docker
kubernetes.core
```

Understanding modules and collections allows you to automate:

- servers
- containers
- cloud infrastructure
- databases
- networks

They form the **foundation of all Ansible automation**.

