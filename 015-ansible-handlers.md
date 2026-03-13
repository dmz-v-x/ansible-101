## Ansible Handlers

Handlers are one of the most important mechanisms in Ansible for writing **efficient and safe automation**.

They allow Ansible to perform actions **only when something changes**, preventing unnecessary operations like restarting services repeatedly.

Handlers are used heavily in **real production infrastructure**, especially when managing services such as:

- nginx
- apache
- mysql
- docker
- systemd services
- applications

---

### 1. The Problem Handlers Solve

Imagine this playbook:

```yaml
- name: Copy nginx config
  copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf

- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

Problem:

Every time the playbook runs, nginx will restart.

Even if the configuration **did not change**.

This causes issues:

- unnecessary restarts
- downtime
- performance impact
- unstable systems

Handlers solve this problem.

---

### 2. What is an Ansible Handler?

A **handler** is a special task that runs **only when notified by another task**.

Handlers execute **only if something changed**.

Example concept:

```
Task changes something → notify handler → handler runs
Task no change → handler not triggered
```

Handlers are most commonly used for:

- restarting services
- reloading configurations
- restarting applications

---

### 3. Basic Handler Example

Example playbook:

```yaml
---
- name: Configure nginx
  hosts: webservers

  tasks:

    - name: Copy nginx configuration
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

  handlers:

    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

Explanation:

If the nginx config file changes, then:

```
restart nginx handler executes
```

Otherwise:

```
handler does not run
```

---

### 4. How Handlers Work Internally

Workflow:

```
1 Task runs
2 Task detects change
3 notify triggers handler
4 Handler added to queue
5 Handlers run at end of play
```

Important rule:

Handlers execute **after all tasks in the play finish**.

---

### 5. Example Workflow

Example playbook:

```yaml
tasks:

- name: Copy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- name: Copy web page
  copy:
    src: index.html
    dest: /var/www/html/index.html
```

Handlers run only **after all tasks finish**.

---

### 6. Handlers Run Only Once

If multiple tasks notify the same handler:

The handler runs **only once**.

Example:

```yaml
tasks:

- name: Update nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx

- name: Update another config
  template:
    src: security.conf.j2
    dest: /etc/nginx/security.conf
  notify: restart nginx
```

Even if both tasks changed files:

```
restart nginx runs once
```

This prevents repeated restarts.

---

### 7. Multiple Handlers

Playbooks can define multiple handlers.

Example:

```yaml
handlers:

- name: restart nginx
  service:
    name: nginx
    state: restarted

- name: reload nginx
  service:
    name: nginx
    state: reloaded
```

Tasks can notify different handlers.

Example:

```yaml
notify: reload nginx
```

---

### 8. Multiple Notifications

Tasks can notify multiple handlers.

Example:

```yaml
notify:
  - restart nginx
  - clear cache
```

Handlers:

```yaml
handlers:

- name: restart nginx
  service:
    name: nginx
    state: restarted

- name: clear cache
  command: rm -rf /var/cache/app
```

---

### 9. Using Handlers with Roles

Handlers are commonly used inside roles.

Example role structure:

```
roles/
  nginx/
    tasks/
      main.yml
    handlers/
      main.yml
```

Example handler file:

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

Example task inside role:

```yaml
- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: restart nginx
```

---

### 10. Using Listen Instead of Name

Handlers can use `listen`.

Example:

```yaml
handlers:

- name: restart nginx service
  listen: restart nginx
  service:
    name: nginx
    state: restarted
```

Task:

```yaml
notify: restart nginx
```

Multiple handlers can listen to the same event.

---

### 11. Handler Execution Order

Handlers execute in the order they are defined.

Example:

```yaml
handlers:

- name: reload nginx
  service:
    name: nginx
    state: reloaded

- name: restart nginx
  service:
    name: nginx
    state: restarted
```

Order matters when dependencies exist.

---

### 12. Forcing Handlers to Run Immediately

Normally handlers run **at end of play**.

Sometimes we want them immediately.

Use:

```
meta: flush_handlers
```

Example:

```yaml
- name: Update config
  template:
    src: config.j2
    dest: /etc/app/config
  notify: restart app

- meta: flush_handlers
```

Now handler runs immediately.

---

### 13. Handlers with Loops

Example task:

```yaml
- name: Update multiple configs
  template:
    src: "{{ item }}"
    dest: /etc/nginx/
  loop:
    - nginx.conf.j2
    - security.conf.j2
  notify: restart nginx
```

Even if multiple files change:

```
handler runs once
```

---

### 14. Real World Production Example

Example: Web server deployment.

Infrastructure:

```
Load Balancer
      |
Web Servers
```

Playbook:

```yaml
- hosts: webservers

  tasks:

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Deploy nginx config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: restart nginx

    - name: Deploy website
      copy:
        src: index.html
        dest: /var/www/html/index.html

  handlers:

    - name: restart nginx
      service:
        name: nginx
        state: restarted
```

Result:

```
nginx restarts only if configuration changes
```

---

### 15. Best Practices for Handlers

Follow these guidelines.

1. Use handlers for service restarts  
2. Avoid restarting services inside normal tasks  
3. Use notify instead of direct restart  
4. Name handlers clearly  
5. Use roles to organize handlers  

Example good practice:

```
notify: restart nginx
```

Instead of:

```
state: restarted
```

inside task.

---

### 16. Common Handler Gotchas

#### Handler not triggered

Occurs if task does not report change.

Example:

```
copy module sees file unchanged
```

Handler not executed.

---

#### Wrong handler name

Example:

```
notify: restart_nginx
```

But handler name:

```
restart nginx
```

Names must match.

---

#### Handler defined after role tasks

Handlers should exist in:

```
handlers/main.yml
```

---

### 17. Debugging Handler Issues

Use verbose mode:

```
ansible-playbook playbook.yml -vvv
```

This shows:

- tasks changed
- handlers triggered

---

### 18. Handlers vs Normal Tasks

| Feature | Task | Handler |
|------|------|------|
Runs every play | Yes | No |
Runs only on change | No | Yes |
Triggered by notify | No | Yes |

Handlers are ideal for:

```
restart services
reload configuration
clear caches
```

---

### 19. When Handlers Are Essential

Handlers are critical when managing:

- web servers
- databases
- application servers
- systemd services

Without handlers, automation becomes inefficient.
