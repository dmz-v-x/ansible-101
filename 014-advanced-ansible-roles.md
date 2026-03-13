## Advanced Ansible Roles — Variables, Handlers, Templates, Dependencies, and Production Best Practices

In the previous section we learned the **foundation of Ansible Roles**:

- What roles are
- Why they exist
- Role directory structure
- Creating roles
- Using roles in playbooks
- Using community roles from Ansible Galaxy

Now we will go **much deeper** into advanced role concepts used in real production infrastructure.

---

### 1. Role Variables (Core Concept)

Roles support multiple places to define variables.

Inside a role we typically define variables in these directories:

```
roles/
  nginx/
    defaults/
      main.yml
    vars/
      main.yml
```

Both files define variables but they behave differently.

---

### 2. Role Defaults (defaults/main.yml)

This file contains **default values for variables**.

Location:

```
roles/nginx/defaults/main.yml
```

Example:

```yaml
nginx_port: 80
nginx_worker_processes: auto
nginx_user: www-data
```

Characteristics:

- Lowest priority variables
- Can be overridden easily
- Recommended place for configurable values

Example usage in template:

```
listen {{ nginx_port }};
```

---

### 3. Role Vars (vars/main.yml)

Location:

```
roles/nginx/vars/main.yml
```

Example:

```yaml
nginx_package_name: nginx
```

Characteristics:

- Higher precedence than defaults
- Harder to override
- Typically used for internal role variables

Best practice:

```
Use defaults for configuration
Use vars for internal values
```

---

### 4. Variable Precedence Inside Roles

Variables in Ansible follow precedence rules.

Simplified order (highest → lowest):

```
Extra vars (--extra-vars)
Task vars
set_fact
Play vars
Host vars
Group vars
Role vars
Role defaults
```

Example:

If `nginx_port` exists in:

```
defaults/main.yml → 80
playbook vars → 8080
```

The playbook value wins.

---

### 5. Passing Variables to Roles

Roles can accept variables when used.

Example playbook:

```yaml
- hosts: webservers
  roles:
    - role: nginx
      nginx_port: 8080
```

Now the role will use port `8080` instead of the default.

This makes roles **fully reusable**.

---

### 6. Role Handlers

Handlers are special tasks triggered by changes.

Example use case:

```
Restart nginx only if configuration changes
```

Handlers are defined here:

```
roles/nginx/handlers/main.yml
```

Example:

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

---

### 7. Triggering Handlers

Handlers are triggered using `notify`.

Example task:

```yaml
- name: Copy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx
```

Workflow:

```
Task changes file → handler triggered → nginx restarted
```

If no change occurs, handler does not run.

---

### 8. Role Templates (Dynamic Configuration)

Templates allow dynamic configuration files.

Location:

```
roles/nginx/templates/nginx.conf.j2
```

Example template:

```
server {
    listen {{ nginx_port }};
    server_name {{ server_name }};
}
```

Example task:

```yaml
- name: Deploy nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

Templates use **Jinja2 templating syntax**.

---

### 9. Role Files (Static Files)

Static files go inside:

```
roles/nginx/files/
```

Example:

```
files/index.html
```

Task example:

```yaml
- name: Copy web page
  copy:
    src: index.html
    dest: /var/www/html/index.html
```

Difference from templates:

| Feature | Files | Templates |
|-------|------|------|
Static content | Yes | No |
Dynamic variables | No | Yes |

---

### 10. Role Dependencies

Roles can depend on other roles.

Example:

```
nginx role requires common role
```

Define dependency:

```
roles/nginx/meta/main.yml
```

Example:

```yaml
dependencies:
  - role: common
```

Execution order:

```
common role runs first
nginx role runs after
```

---

### 11. Role Parameterization

Roles can be parameterized for different environments.

Example infrastructure:

```
Dev environment
Staging environment
Production environment
```

Example role variable:

```
nginx_port
```

Playbook usage:

Dev:

```yaml
roles:
  - role: nginx
    nginx_port: 8080
```

Production:

```yaml
roles:
  - role: nginx
    nginx_port: 80
```

Same role works everywhere.

---

### 12. Tagging Roles

Roles can be tagged to run specific tasks.

Example:

```yaml
roles:
  - role: nginx
    tags: web
```

Run only web role:

```
ansible-playbook playbook.yml --tags web
```

---

### 13. Real World Role Architecture

Example production project:

```
ansible-project/
│
├── inventory/
│   ├── dev
│   ├── staging
│   └── prod
│
├── roles/
│   ├── common
│   ├── nginx
│   ├── mysql
│   ├── redis
│   ├── docker
│   └── monitoring
│
├── playbooks/
│   ├── webservers.yml
│   ├── database.yml
│   └── infrastructure.yml
```

This structure scales to **hundreds of servers**.

---

### 14. Real World Deployment Example

Scenario:

Deploy web application infrastructure.

Components:

```
Load Balancer
Web Servers
Database
Cache
```

Roles used:

```
roles/
├── nginx
├── mysql
├── redis
├── docker
└── monitoring
```

Playbook:

```yaml
- hosts: webservers
  roles:
    - nginx
    - docker

- hosts: databases
  roles:
    - mysql
```

Automation becomes modular.

---

### 15. Role Testing (Production Practice)

Roles should be tested independently.

Tools commonly used:

- **Molecule**
- **Testinfra**
- **Docker containers for testing**

Example test workflow:

```
Write role
Run molecule test
Verify infrastructure behavior
Deploy role to production
```

---

### 16. Using Roles from Ansible Galaxy

Install community role:

```
ansible-galaxy install geerlingguy.nginx
```

Role location:

```
~/.ansible/roles/
```

Use in playbook:

```yaml
roles:
  - geerlingguy.nginx
```

Galaxy contains thousands of roles.

---

### 17. Best Practices for Production Roles

Follow these guidelines.

1. One role = one responsibility  
2. Avoid hardcoding values  
3. Use defaults for configuration variables  
4. Use templates for configuration files  
5. Use handlers for service restarts  
6. Keep roles reusable across projects  
7. Use version control for roles  

---

### 18. Common Role Gotchas

#### Hardcoded values

Bad:

```
listen 80
```

Good:

```
listen {{ nginx_port }}
```

---

#### Large roles doing too many things

Example bad role:

```
nginx role installing docker + mysql
```

Break roles into separate responsibilities.

---

#### Not using handlers

Restarting services unnecessarily causes downtime.

Always use handlers.

---

### 19. When Roles Are Absolutely Necessary

Roles become essential when:

- infrastructure contains many services
- multiple environments exist
- teams collaborate
- automation grows beyond a few playbooks

In real companies, **almost all automation uses roles**.
