## Ansible Variables 

Variables are one of the **most important concepts in Ansible**.  
Without variables, playbooks become rigid and hard to reuse.

Variables allow playbooks to become:

- reusable
- dynamic
- environment-specific
- scalable for real infrastructure

In real DevOps environments variables are used for:

- ports
- usernames
- file paths
- package versions
- environment configuration (dev / staging / prod)
- secrets
- host-specific settings

---

### 1. Why Variables Are Important in Ansible

Imagine we write a playbook like this:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

Now suppose we want to install **Apache instead of nginx**.

We would have to modify the playbook.

Instead we use variables.

```
package_name: nginx
```

Then our playbook becomes reusable.

```yaml
apt:
  name: "{{ package_name }}"
  state: present
```

Now we can run the same playbook for:

| Environment | Package |
|---|---|
Dev | nginx |
Staging | apache |
Production | nginx |

---

### 2. Variable Syntax in Ansible

Variables are referenced using **Jinja2 templating syntax**.

```
{{ variable_name }}
```

Example:

```yaml
msg: "The server name is {{ server_name }}"
```

Important rule:

Always use quotes when embedding variables inside strings.

Correct:

```
msg: "Hello {{ username }}"
```

Incorrect:

```
msg: Hello {{ username }}
```

---

### 3. Defining Variables in a Playbook

The simplest way to define variables is using the **vars block**.

Example:

```yaml
---
- name: Example using vars
  hosts: localhost

  vars:
    my_name: Himanshu
    city: Bangalore

  tasks:
    - name: Print variables
      debug:
        msg: "Hello {{ my_name }} from {{ city }}"
```

Output:

```
Hello Himanshu from Bangalore
```

---

### 4. The Debug Module (Inspecting Variables)

The **debug module** is extremely useful when working with variables.

It allows us to print:

- messages
- variable values
- dictionaries
- facts

Example:

```yaml
- name: Print variable
  debug:
    var: my_name
```

Output:

```
my_name: Himanshu
```

Another example:

```yaml
- name: Custom message
  debug:
    msg: "Hello {{ my_name }}"
```

---

### 5. Registering Variables from Task Output

One powerful feature of Ansible is capturing output from tasks.

This is done using **register**.

Example:

```yaml
- name: Get hostname of the machine
  command: hostname
  register: my_hostname
```

This stores the output of the command in a variable called:

```
my_hostname
```

---

### 6. Understanding Registered Variables

Registered variables are stored as **dictionaries**.

Example structure:

```
my_hostname:
  stdout: server1
  stderr:
  rc: 0
  changed: false
```

Important fields:

| Field | Meaning |
|---|---|
stdout | actual output |
stderr | error output |
rc | return code |
changed | whether change occurred |

Most of the time we use:

```
stdout
```

Example:

```yaml
- name: Show hostname
  debug:
    var: my_hostname.stdout
```

---

### 7. Accessing Dictionary Variables

Since registered variables are dictionaries, we use **dot notation**.

Example:

```
{{ my_hostname.stdout }}
```

Other examples:

```
{{ my_hostname.rc }}
{{ my_hostname.stderr }}
```

---

### 8. What is set_fact?

`set_fact` is a module used to **create or update variables during playbook execution**.

Example:

```yaml
- name: Set a fact variable
  set_fact:
    city: "New York"
```

Now the variable `city` becomes available for the rest of the play.

Example usage:

```yaml
- name: Use variable
  debug:
    msg: "City is {{ city }}"
```

---

### 9. Why set_fact is Useful

set_fact is useful when:

- computing dynamic values
- storing intermediate results
- modifying variables during execution

Example:

```yaml
- name: Calculate new port
  set_fact:
    new_port: "{{ base_port + 1 }}"
```

---

### 10. Parameterized Playbooks

A **parameterized playbook** accepts values at runtime.

This makes playbooks reusable.

Example playbook:

```yaml
---
- name: Install a package
  hosts: all
  become: yes

  vars:
    pkg_name: httpd

  tasks:
    - name: Install package
      apt:
        name: "{{ pkg_name }}"
        state: present
```

Now we can change package dynamically.

---

### 11. Passing Variables via Command Line

Variables can be passed when running the playbook.

Example:

```
ansible-playbook playbook.yml --extra-vars "username=bob age=40"
```

Inside playbook:

```yaml
debug:
  msg: "User {{ username }} is {{ age }} years old"
```

Output:

```
User bob is 40 years old
```

---

### 12. Using External Variable Files

Large projects store variables in **separate files**.

Example project:

```
my-ansible-project/
├── playbook.yml
├── vars/
│   └── myvars.yml
```

---

### 13. Creating a Variable File

vars/myvars.yml

```yaml
username: alice
port: 8080
city: "New York"
```

---

### 14. Loading Variable Files

Inside playbook:

```yaml
---
- name: Example Playbook
  hosts: localhost

  vars_files:
    - vars/myvars.yml

  tasks:
    - name: Show variables
      debug:
        msg: "User {{ username }} from {{ city }} uses port {{ port }}"
```

---

### 15. Variable Scope

Variable scope determines **where variables are accessible**.

Common scopes:

| Scope | Description |
|---|---|
Play scope | variables defined inside play |
Host scope | variables for specific host |
Group scope | variables for host group |
Global scope | available everywhere |

---

### 16. Host Variables

Host variables apply to a specific host.

Example inventory:

```
[webserver]
server1 ansible_host=192.168.1.10 http_port=80
server2 ansible_host=192.168.1.11 http_port=8080
```

Now each host has different port.

---

### 17. Group Variables

Group variables apply to all hosts in a group.

Example:

```
[webserver]
server1
server2

[webserver:vars]
http_port=80
```

Now all webservers share the same port.

---

### 18. Variable Precedence (Important Advanced Topic)

Sometimes variables are defined in multiple places.

Ansible follows **precedence rules**.

Highest priority:

```
1 extra vars (--extra-vars)
2 task vars
3 set_fact
4 play vars
5 host vars
6 group vars
7 inventory vars
8 role defaults
```

Example:

If variable defined in playbook and CLI:

CLI value wins.

---

### 19. Real World Example — Environment Configuration

Example infrastructure:

```
Dev
Staging
Production
```

Variable files:

```
vars/dev.yml
vars/staging.yml
vars/prod.yml
```

Example:

```
db_host: dev-db.company.com
```

Playbook loads environment file.

---

### 20. Using Variables in Loops

Example:

```yaml
vars:
  packages:
    - nginx
    - git
    - curl
```

Task:

```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop: "{{ packages }}"
```

---

### 21. Variable Gotchas (Common Mistakes)

Important mistakes beginners make.

#### Missing Quotes

Wrong:

```
msg: Hello {{ user }}
```

Correct:

```
msg: "Hello {{ user }}"
```

---

#### Undefined Variables

If variable doesn't exist:

Ansible throws error.

Example fix:

```
{{ variable | default("value") }}
```

---

#### Wrong Scope

Variable defined in task cannot be used in previous tasks.

---

### 22. Debugging Variables

Useful commands:

```
ansible-playbook playbook.yml -vvv
```

Print variables:

```yaml
debug:
  var: variable_name
```

List all facts:

```
ansible all -m setup
```

---

### 23. Real World Variable Usage Example

Complete playbook:

```yaml
---
- name: Deploy web application
  hosts: webserver

  vars:
    app_dir: /var/www/app
    package_name: nginx

  tasks:

    - name: Install package
      apt:
        name: "{{ package_name }}"
        state: present

    - name: Create directory
      file:
        path: "{{ app_dir }}"
        state: directory

    - name: Show configuration
      debug:
        msg: "Application installed in {{ app_dir }}"
```

---

### 24. Key Takeaways

Variables make Ansible automation **dynamic and reusable**.

Important concepts covered:

- variable syntax `{{ }}`
- vars block
- register variables
- dictionary access
- set_fact module
- parameterized playbooks
- CLI variables
- variable files
- host & group variables
- variable precedence
- debugging variables

Variables are used everywhere in Ansible including:

- playbooks
- roles
- templates
- inventories
- cloud automation

