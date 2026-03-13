## Ansible Tags

As Ansible playbooks grow larger, they often contain **dozens or even hundreds of tasks**.  
Running the entire playbook every time can be inefficient.

Sometimes you may want to:

- Run only **installation tasks**
- Run only **configuration tasks**
- Skip **database tasks**
- Run only **debug or testing tasks**

Ansible **Tags** solve this problem by allowing you to **selectively run parts of a playbook**.

Tags are widely used in real DevOps workflows such as:

- CI/CD pipelines
- infrastructure upgrades
- partial deployments
- troubleshooting automation

---

### 1. What Are Ansible Tags?

An **Ansible tag** is a label assigned to a task, role, or play.

Tags allow you to run **specific tasks instead of the entire playbook**.

Example:

```
install
configure
deploy
restart
debug
```

When executing a playbook, you can run:

```
only tasks with specific tags
```

---

### 2. Why Tags Are Useful

Consider a playbook with many tasks:

```
1 Install packages
2 Configure application
3 Deploy application code
4 Restart services
5 Run database migrations
```

Sometimes you only want to:

```
Deploy new code
```

Without tags you must run the entire playbook.

With tags you run only the deployment step.

---

### 3. Basic Tag Example

Example playbook:

```yaml
- name: Setup web server
  hosts: webservers

  tasks:

    - name: Install nginx
      apt:
        name: nginx
        state: present
      tags: install

    - name: Deploy configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      tags: configure

    - name: Restart nginx
      service:
        name: nginx
        state: restarted
      tags: restart
```

Each task now has a tag.

---

### 4. Running Tasks with Tags

To run only install tasks:

```
ansible-playbook playbook.yml --tags install
```

Only tasks with tag `install` will run.

Example result:

```
TASK [Install nginx]
```

Other tasks are skipped.

---

### 5. Running Multiple Tags

You can run multiple tags.

Example:

```
ansible-playbook playbook.yml --tags "install,configure"
```

This runs tasks tagged with:

```
install
configure
```

---

### 6. Skipping Tags

Instead of selecting tasks, you can **skip certain tags**.

Example:

```
ansible-playbook playbook.yml --skip-tags restart
```

This runs everything except tasks tagged with `restart`.

---

### 7. Tagging an Entire Play

Tags can be applied at the play level.

Example:

```yaml
- name: Setup web server
  hosts: webservers
  tags: websetup

  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

Run:

```
ansible-playbook playbook.yml --tags websetup
```

All tasks in that play execute.

---

### 8. Tagging Multiple Tasks

Instead of repeating tags, we can group them.

Example:

```yaml
tasks:

- name: Install nginx
  apt:
    name: nginx
    state: present
  tags:
    - install
    - web

- name: Install curl
  apt:
    name: curl
    state: present
  tags:
    - install
```

Now tasks have multiple tags.

---

### 9. Tagging Roles

Roles can also have tags.

Example playbook:

```yaml
- hosts: webservers
  roles:
    - role: nginx
      tags: web
```

Run only role tasks:

```
ansible-playbook playbook.yml --tags web
```

---

### 10. Tags Inside Roles

Roles can define tags inside their tasks.

Example:

```
roles/nginx/tasks/main.yml
```

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
  tags: install
```

Playbook:

```yaml
roles:
  - nginx
```

Run only installation tasks:

```
ansible-playbook playbook.yml --tags install
```

---

### 11. Special Tags

Ansible provides special tags.

| Tag | Meaning |
|----|----|
always | always run |
never | never run unless explicitly called |

---

### 12. Using the `always` Tag

Example:

```yaml
- name: Print message
  debug:
    msg: "Always runs"
  tags: always
```

Even if you run specific tags:

```
ansible-playbook playbook.yml --tags install
```

This task still runs.

---

### 13. Using the `never` Tag

Example:

```yaml
- name: Debug information
  debug:
    var: ansible_hostname
  tags: never
```

This task will **not run normally**.

Run it explicitly:

```
ansible-playbook playbook.yml --tags never
```

Useful for debugging.

---

### 14. Listing Available Tags

Before running a playbook, you can list tags.

Command:

```
ansible-playbook playbook.yml --list-tags
```

Example output:

```
TASK TAGS: [install, configure, restart]
```

---

### 15. Listing Tagged Tasks

You can list tasks without executing them.

```
ansible-playbook playbook.yml --list-tasks
```

Output shows tasks and their tags.

---

### 16. Tags in Real DevOps Workflows

Tags are commonly used in CI/CD pipelines.

Example pipeline:

```
1 Infrastructure setup
2 Application deployment
3 Configuration update
4 Restart services
```

Playbook tags:

```
infra
deploy
config
restart
```

CI/CD example:

```
ansible-playbook deploy.yml --tags deploy
```

Only deployment tasks run.

---

### 17. Real World Example

Example infrastructure automation playbook:

```yaml
- name: Deploy web application
  hosts: webservers

  tasks:

    - name: Install dependencies
      apt:
        name: nginx
        state: present
      tags: install

    - name: Deploy application code
      copy:
        src: app/
        dest: /var/www/app
      tags: deploy

    - name: Restart web service
      service:
        name: nginx
        state: restarted
      tags: restart
```

Deployment scenario:

```
ansible-playbook deploy.yml --tags deploy
```

Only application deployment runs.

---

### 18. Common Tag Gotchas

#### Forgetting tags on dependent tasks

Example:

```
config task tagged
restart task not tagged
```

Restart will not run.

---

#### Too many tags

Avoid creating too many tags.

Keep tag structure simple.

---

#### Tags not applied to roles

Roles require explicit tagging if selective execution is needed.

---

### 19. Best Practices

1. Use meaningful tag names  
2. Group related tasks with same tag  
3. Tag roles for modular automation  
4. Use tags in CI/CD pipelines  
5. Avoid excessive tagging complexity  

Example recommended tags:

```
install
config
deploy
restart
cleanup
debug
```

