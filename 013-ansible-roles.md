## Ansible Roles

As Ansible automation grows, playbooks can become **very large and difficult to maintain**.  
Roles solve this problem by allowing us to **organize automation code into reusable packages**.

Roles are one of the **most important best practices in Ansible**, especially in real production environments.

---

### 1. Why Roles Exist (The Problem)

Let’s start with a simple playbook.

Example:

```yaml
---
- name: Setup web server
  hosts: webservers

  tasks:

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Copy nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf

    - name: Start nginx
      service:
        name: nginx
        state: started
```

Now imagine your infrastructure grows.

You may need to configure:

- Web servers
- Database servers
- Load balancers
- Monitoring agents
- Logging agents
- Security tools

Your playbook might become **hundreds or thousands of lines long**.

Problems:

- Hard to read
- Hard to reuse
- Hard to maintain
- Difficult for teams to collaborate

This is where **roles come into picture**.

---

### 2. What is an Ansible Role?

An **Ansible Role** is a structured way of organizing automation code.

A role groups together everything needed for a specific task:

- tasks
- variables
- files
- templates
- handlers
- metadata

In simple terms:

> A role is a **package of automation logic**.

Example role:

```
nginx-role
```

This role may include:

- tasks to install nginx
- configuration templates
- variables
- service restart handlers

---

### 3. Real World Example

Imagine deploying a web application infrastructure.

Components:

```
Infrastructure
│
├── Web Server
├── Database
├── Cache
└── Monitoring
```

Each component can be a **role**.

Example:

```
roles/
├── nginx
├── mysql
├── redis
└── prometheus
```

Each role handles **one responsibility**.

This follows the **separation of concerns principle**.

---

### 4. When Do Ansible Roles Come into Picture?

Roles become useful when:

| Situation | Why Roles Help |
|---|---|
Playbooks become large | Improves organization |
Tasks repeat across projects | Enables reuse |
Teams collaborate | Clear modular structure |
Infrastructure becomes complex | Separates components |

---

### 5. What Roles Are Used For

Roles help with:

- Modularizing playbooks
- Reusing automation code
- Organizing infrastructure components
- Sharing automation through Ansible Galaxy

Example:

Instead of writing everything inside one playbook, we break it into roles.

```
playbook
   |
   |-- role: nginx
   |-- role: mysql
   |-- role: redis
```

---

### 6. Role Directory Structure

Roles follow a **standard directory structure**.

Example:

```
my-role/
├── defaults/
│   └── main.yml
├── files/
│   └── something.txt
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
│   └── config.j2
├── vars/
│   └── main.yml
```

Each directory has a specific purpose.

---

### 7. Understanding Each Role Directory

Let’s examine each folder carefully.

---

#### tasks/

This is the **most important directory**.

File:

```
tasks/main.yml
```

Contains the list of tasks executed by the role.

Example:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

---

#### handlers/

Handlers contain tasks triggered by notifications.

Example:

```
handlers/main.yml
```

Example handler:

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

Handlers run **only when notified**.

---

#### templates/

Contains **Jinja2 templates**.

Example:

```
templates/nginx.conf.j2
```

Templates allow dynamic configuration.

Example:

```
server {
  listen {{ nginx_port }};
}
```

---

#### files/

Contains static files.

Example:

```
files/index.html
```

These files can be copied to servers.

Example task:

```yaml
copy:
  src: index.html
  dest: /var/www/html/index.html
```

---

#### defaults/

Contains **default variables**.

File:

```
defaults/main.yml
```

Example:

```yaml
nginx_port: 80
```

These variables have the **lowest priority**.

---

#### vars/

Contains role-specific variables.

File:

```
vars/main.yml
```

Example:

```yaml
nginx_package: nginx
```

These variables have **higher precedence** than defaults.

---

#### meta/

Contains role metadata.

Example:

```
meta/main.yml
```

Used to define:

- role dependencies
- author
- supported platforms

Example:

```yaml
dependencies:
  - role: common
```

---

### 8. Creating a Role

Ansible provides a command to create roles.

Command:

```
ansible-galaxy init my-role
```

Example:

```
ansible-galaxy init nginx
```

Generated structure:

```
roles/nginx/
├── defaults
├── files
├── handlers
├── meta
├── tasks
├── templates
└── vars
```

---

### 9. Writing Tasks in the Role

Example file:

```
roles/nginx/tasks/main.yml
```

Example tasks:

```yaml
- name: Install Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Start Nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

---

### 10. Using Roles in a Playbook

Roles are referenced inside playbooks.

Example:

```yaml
- name: Setup Web Server
  hosts: webservers
  become: true

  roles:
    - nginx
```

Ansible automatically executes:

```
roles/nginx/tasks/main.yml
```

---

### 11. Complete Real World Example

Project structure:

```
project/
│
├── inventory
├── playbook.yml
└── roles/
    └── nginx/
        ├── tasks/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── templates/
        │   └── nginx.conf.j2
        ├── files/
        │   └── index.html
        ├── defaults/
        │   └── main.yml
        └── vars/
            └── main.yml
```

---

### 12. Example Playbook Using Roles

File:

```
webserver.yml
```

Example:

```yaml
---
- name: Setup Web Server
  hosts: web
  become: true

  roles:
    - nginx
```

---

### 13. Running the Playbook

Command:

```
ansible-playbook webserver.yml
```

Ansible executes the nginx role.

---

### 14. Role Dependencies

Roles can depend on other roles.

Example:

```
nginx depends on common role
```

Example:

```
meta/main.yml
```

```yaml
dependencies:
  - role: common
```

Ansible will run the **common role first**.

---

### 15. Using Community Roles (Ansible Galaxy)

Ansible Galaxy is the **central repository for roles**.

Website:

```
https://galaxy.ansible.com
```

Example install command:

```
ansible-galaxy install geerlingguy.nginx
```

This downloads a community role.

Example usage:

```yaml
roles:
  - geerlingguy.nginx
```

---

### 16. Real World Role Architecture

Example enterprise infrastructure:

```
roles/
├── common
├── nginx
├── mysql
├── redis
├── docker
└── monitoring
```

Each role manages **one infrastructure component**.

---

### 17. Advantages of Roles

Roles provide many benefits.

| Benefit | Explanation |
|---|---|
Modularity | break automation into components |
Reusability | use roles across projects |
Maintainability | easier to update |
Team collaboration | different teams manage roles |
Testing | roles can be tested independently |

---

### 18. Common Role Gotchas

#### Hardcoding variables

Avoid hardcoding values inside tasks.

Instead use variables.

---

#### Mixing responsibilities

A role should manage **one component only**.

Example bad practice:

```
nginx role installing mysql
```

---

#### Forgetting defaults

Always define default values in:

```
defaults/main.yml
```

---

### 19. Best Practices for Roles

1. One role = one responsibility  
2. Keep roles small and reusable  
3. Store roles in Git  
4. Use variables instead of hardcoding  
5. Use Ansible Galaxy roles when possible  

---

### 20. Summary

Roles are a **modular packaging system for Ansible automation**.

They allow us to:

- organize playbooks
- reuse automation code
- separate infrastructure components
- share automation with the community

Key elements of roles:

```
tasks
handlers
templates
files
vars
defaults
meta
```

Roles are essential for **scalable DevOps automation** and are widely used in **production infrastructure management**.

