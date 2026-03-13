## Ansible Conditionals — Complete Guide from Absolute Beginner to Advanced

Conditionals are one of the most important features in Ansible because they allow playbooks to make **decisions during execution**.

Instead of running every task on every server, conditionals allow Ansible to run tasks **only when certain conditions are true**.

This makes automation:

- smarter
- safer
- environment-aware
- reusable across different systems

In real DevOps environments, conditionals are used for:

- OS specific tasks
- environment based deployment
- configuration differences between servers
- checking system state before executing tasks
- error handling and validations

In this guide we will learn **everything about Ansible conditionals step by step**, including:

- What conditionals are
- The `when` statement
- Using facts with conditionals
- Boolean logic
- Conditionals with variables
- Conditionals with loops
- Conditionals with registered variables
- Multiple conditions
- Using `failed_when` and `changed_when`
- Real-world examples
- Best practices and gotchas

---

### 1. What Are Ansible Conditionals?

Conditionals allow Ansible to **execute a task only if a condition is true**.

They are implemented using the `when` keyword.

Example:

```yaml
- name: Install nginx only on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"
```

Meaning:

```
Run this task only if OS is Ubuntu
```

---

### 2. Basic Syntax of Conditionals

The syntax is simple.

```
when: condition
```

Example:

```yaml
when: ansible_os_family == "Debian"
```

Important rule:

Do **not** wrap condition inside `{{ }}`.

Correct:

```
when: ansible_distribution == "Ubuntu"
```

Incorrect:

```
when: "{{ ansible_distribution == 'Ubuntu' }}"
```

---

### 3. Using Facts in Conditionals

Facts provide system information.

Example:

```yaml
when: ansible_distribution == "Ubuntu"
```

Example playbook:

```yaml
- name: Install nginx on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"
```

---

### 4. Multi-OS Example

One playbook supporting multiple OS.

```yaml
- name: Install nginx on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"

- name: Install nginx on CentOS
  yum:
    name: nginx
    state: present
  when: ansible_os_family == "RedHat"
```

This is very common in production automation.

---

### 5. Boolean Conditionals

Boolean variables can be used directly.

Example:

```yaml
vars:
  install_nginx: true
```

Task:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
  when: install_nginx
```

If variable is true → task runs.

---

### 6. Multiple Conditions (AND)

Use `and`.

Example:

```yaml
when:
  - ansible_distribution == "Ubuntu"
  - ansible_memtotal_mb > 1024
```

Meaning:

```
Run task only if OS = Ubuntu AND memory > 1GB
```

---

### 7. Multiple Conditions (OR)

Use `or`.

Example:

```yaml
when: ansible_distribution == "Ubuntu" or ansible_distribution == "Debian"
```

Meaning:

```
Run task if OS is Ubuntu OR Debian
```

---

### 8. Conditionals with Variables

Example:

```yaml
vars:
  env: production
```

Task:

```yaml
- name: Run only in production
  debug:
    msg: "Deploying production configuration"
  when: env == "production"
```

---

### 9. Conditionals with Registered Variables

Sometimes tasks depend on results of previous tasks.

Example:

```yaml
- name: Check nginx version
  command: nginx -v
  register: nginx_result
```

Conditional task:

```yaml
- name: Show version
  debug:
    var: nginx_result.stdout
  when: nginx_result.rc == 0
```

Explanation:

```
rc = return code
0 = success
```

---

### 10. Conditionals with Loops

Example variable:

```yaml
users:
  - alice
  - bob
  - carol
```

Example task:

```yaml
- name: Create users
  user:
    name: "{{ item }}"
  loop: "{{ users }}"
  when: item != "bob"
```

Result:

```
alice created
bob skipped
carol created
```

---

### 11. Using Conditionals in Templates

Templates also support conditionals.

Example:

```
{% if environment == "production" %}
worker_processes auto;
{% else %}
worker_processes 1;
{% endif %}
```

---

### 12. Conditionals with Groups

Example:

```
groups['webservers']
```

Example task:

```yaml
- name: Run only on first web server
  debug:
    msg: "Primary server"
  when: inventory_hostname == groups['webservers'][0]
```

---

### 13. Using `changed_when`

Sometimes a command always reports changed.

Example:

```yaml
- name: Run command
  command: echo hello
```

Force no change:

```yaml
changed_when: false
```

Example:

```yaml
- name: Run health check
  command: curl localhost
  changed_when: false
```

---

### 14. Using `failed_when`

This allows custom failure conditions.

Example:

```yaml
- name: Check disk usage
  command: df -h
  register: disk_output
  failed_when: "'100%' in disk_output.stdout"
```

Meaning:

```
Fail task if disk usage reaches 100%
```

---

### 15. Real World Example

Example infrastructure:

```
webservers
databases
monitoring
```

Playbook:

```yaml
- name: Install monitoring agent
  apt:
    name: node-exporter
    state: present
  when: "'monitoring' in group_names"
```

Meaning:

```
Install agent only on monitoring servers
```

---

### 16. Conditional Execution with Tags

Example:

```yaml
- name: Run only for upgrade
  debug:
    msg: "Upgrade step"
  when: upgrade | default(false)
```

Run playbook:

```
ansible-playbook playbook.yml -e "upgrade=true"
```

---

### 17. Real Production Example

Scenario:

Deploy different configs depending on environment.

Variables:

```yaml
env: staging
```

Task:

```yaml
- name: Deploy staging config
  template:
    src: staging.conf.j2
    dest: /etc/app/config
  when: env == "staging"
```

---

### 18. Common Conditional Gotchas

#### Using `{{ }}` inside when

Wrong:

```
when: "{{ env == 'prod' }}"
```

Correct:

```
when: env == "prod"
```

---

#### Comparing numbers as strings

Wrong:

```
when: memory == "1024"
```

Correct:

```
when: memory == 1024
```

---

#### Undefined variables

Use default filter.

```
when: env | default("dev") == "prod"
```

---

### 19. Best Practices

1. Use conditionals to support multiple OS
2. Avoid complex conditions in tasks
3. Use variables for environment logic
4. Keep conditionals readable
5. Use `failed_when` for validation checks
