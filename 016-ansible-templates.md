## Ansible Templates (Jinja2) 

Templates are one of the **most powerful features in Ansible**.  
They allow you to generate **dynamic configuration files** for servers.

Instead of copying the same static configuration file to every server, templates allow you to **customize configuration based on variables, conditions, loops, and host information**.

Templates in Ansible use a templating engine called **Jinja2**.

---

### 1. The Problem Templates Solve

Imagine you manage **10 web servers**.

Each server requires an Nginx configuration file.

Example configuration:

```
server {
    listen 80;
    server_name example.com;
}
```

Now suppose different servers require different configurations.

Example:

| Server | Domain | Port |
|------|------|------|
web1 | app1.company.com | 80 |
web2 | app2.company.com | 8080 |

If you copy the same config file to every server, it will not work.

You would need **different config files for each server**.

This becomes impossible to manage at scale.

Templates solve this problem.

---

### 2. What is an Ansible Template?

An **Ansible template** is a file that contains:

- normal text
- variables
- logic (loops and conditions)

Templates are processed by **Jinja2** before being copied to the server.

Workflow:

```
Template file
      |
Jinja2 rendering
      |
Final configuration file
      |
Copied to server
```

Example template:

```
server {
    listen {{ nginx_port }};
    server_name {{ domain_name }};
}
```

When executed, variables are replaced with real values.

---

### 3. What is Jinja2?

**Jinja2** is a Python-based templating engine.

It allows us to embed programming-like logic inside text files.

Example:

```
Hello {{ username }}
```

Jinja2 replaces the variable with the value.

Example result:

```
Hello Himanshu
```

Jinja2 is used in many tools including:

- Ansible
- Flask (Python web framework)
- SaltStack
- Pelican static site generator

---

### 4. Jinja2 Syntax Basics

Jinja2 uses special syntax.

Three main types:

| Syntax | Purpose |
|------|------|
`{{ }}` | Print variables |
`{% %}` | Control logic |
`{# #}` | Comments |

Example:

```
Hello {{ username }}
```

Logic example:

```
{% if port == 80 %}
Default HTTP port
{% endif %}
```

Comment example:

```
{# This is a comment #}
```

---

### 5. Template File Extension

Ansible template files usually end with:

```
.j2
```

Example:

```
nginx.conf.j2
```

This indicates the file contains **Jinja2 syntax**.

---

### 6. Using Templates in Ansible

Templates are applied using the **template module**.

Example playbook task:

```yaml
- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

Explanation:

| Parameter | Meaning |
|---|---|
src | template file |
dest | destination path |

---

### 7. Basic Template Example

Template file:

```
templates/nginx.conf.j2
```

Example content:

```
server {
    listen {{ nginx_port }};
    server_name {{ domain_name }};
}
```

Playbook:

```yaml
vars:
  nginx_port: 80
  domain_name: example.com
```

Rendered file on server:

```
server {
    listen 80;
    server_name example.com;
}
```

---

### 8. Using Variables in Templates

Variables inside templates come from:

- playbook variables
- inventory variables
- role variables
- facts

Example template:

```
Server name: {{ inventory_hostname }}
```

Example output:

```
Server name: web1
```

---

### 9. Using Conditionals in Templates

Jinja2 allows conditional logic.

Example template:

```
{% if nginx_port == 80 %}
Default HTTP configuration
{% else %}
Custom port configuration
{% endif %}
```

If port = 80:

```
Default HTTP configuration
```

---

### 10. Using Loops in Templates

Loops allow repeated content generation.

Example template:

```
upstream backend {
{% for server in backend_servers %}
    server {{ server }};
{% endfor %}
}
```

Variables:

```yaml
backend_servers:
  - 10.0.0.1
  - 10.0.0.2
  - 10.0.0.3
```

Rendered output:

```
upstream backend {
    server 10.0.0.1;
    server 10.0.0.2;
    server 10.0.0.3;
}
```

---

### 11. Jinja2 Filters

Filters modify variables.

Syntax:

```
{{ variable | filter }}
```

Example:

```
{{ username | upper }}
```

Output:

```
HIMANSHU
```

Common filters:

| Filter | Purpose |
|------|------|
upper | uppercase |
lower | lowercase |
default | fallback value |
length | count elements |
join | join lists |

Example:

```
{{ users | join(',') }}
```

---

### 12. Using Default Values

If a variable may not exist, use `default`.

Example:

```
{{ port | default(80) }}
```

If `port` undefined → 80 used.

---

### 13. Using Facts in Templates

Templates can use Ansible facts.

Example:

```
Operating system: {{ ansible_distribution }}
```

Example output:

```
Operating system: Ubuntu
```

Facts come from **gather facts module**.

---

### 14. Templates Inside Roles

Typical role structure:

```
roles/
  nginx/
    templates/
      nginx.conf.j2
    tasks/
      main.yml
```

Task:

```yaml
- name: Deploy nginx configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

Ansible automatically looks in:

```
roles/nginx/templates/
```

---

### 15. Real World Template Example

Example: load balancer configuration.

Template:

```
upstream backend {
{% for host in groups['webservers'] %}
    server {{ host }};
{% endfor %}
}
```

This dynamically builds backend server list.

If inventory changes:

```
web1
web2
web3
```

The config updates automatically.

---

### 16. Template vs Copy Module

| Feature | Template | Copy |
|------|------|------|
Dynamic content | Yes | No |
Variables | Yes | No |
Loops/logic | Yes | No |
Static file | No | Yes |

Use template when configuration must be **dynamic**.

---

### 17. Real World DevOps Scenario

Example infrastructure:

```
Load Balancer
      |
Web Servers
```

Inventory:

```
[webservers]
web1
web2
web3
```

Template generates:

```
upstream backend {
    server web1;
    server web2;
    server web3;
}
```

This configuration updates automatically as servers scale.

---

### 18. Common Template Gotchas

#### Missing quotes

Wrong:

```
server_name {{ domain }}
```

Correct:

```
server_name "{{ domain }}"
```

---

#### Indentation errors

Jinja2 loops must maintain indentation.

---

#### Undefined variables

Use default filter:

```
{{ port | default(80) }}
```

---

### 19. Debugging Templates

Use:

```
ansible-playbook playbook.yml -vvv
```

To test template rendering locally:

```
ansible localhost -m template -a "src=test.j2 dest=/tmp/test"
```

---

### 20. Best Practices

1. Store templates inside roles  
2. Avoid hardcoding values  
3. Use variables for configuration  
4. Use loops for dynamic lists  
5. Use filters to sanitize variables  

