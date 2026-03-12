## Advanced Usage of Ansible Modules — Parameters, Loops, Conditionals, and Real Production Automation

In the previous section we learned:

- What **Ansible modules** are  
- How modules execute tasks  
- Commonly used modules (`apt`, `service`, `file`, `copy`, `user`)  
- What **Ansible collections** are  
- How to install and use collections  

Now we move to **advanced usage of modules inside playbooks**, which is how real-world automation is written.


---

### 1. Real-World Scenario: Deploying a Web Application

Imagine a startup deploying a simple web application.

Infrastructure:

```
Internet
   |
Load Balancer
   |
Web Servers
 |        |
server1  server2
```

Each web server must:

1. Install nginx
2. Create application directory
3. Create multiple subdirectories
4. Copy application files
5. Start nginx
6. Restart nginx if configuration changes

We will automate this entire process using **Ansible modules**.

---

### 2. Understanding Module Parameters

Every Ansible module accepts **parameters**.

Parameters tell the module **how to perform the task**.

Example module usage:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

Here:

| Parameter | Meaning |
|---|---|
| name | package name |
| state | desired state |

Common states:

| State | Meaning |
|---|---|
present | install package |
latest | install latest version |
absent | remove package |

Example remove nginx:

```yaml
apt:
  name: nginx
  state: absent
```

---

### 3. Using Variables with Modules

Instead of hardcoding values, we can use **variables**.

Example:

```yaml
---
- name: Install packages
  hosts: webserver

  vars:
    web_package: nginx

  tasks:
    - name: Install web server
      apt:
        name: "{{ web_package }}"
        state: present
```

Explanation:

```
{{ variable_name }}
```

This syntax is called **Jinja2 templating**.

Variables make playbooks:

- reusable
- configurable
- easier to maintain

---

### 4. Creating Multiple Directories Using Loops

Real-world deployments often require multiple directories.

Example directories:

```
/var/www/app
/var/www/app/logs
/var/www/app/config
```

Instead of repeating tasks we use **loops**.

Example:

```yaml
- name: Create application directories
  file:
    path: "{{ item }}"
    state: directory
  loop:
    - /var/www/app
    - /var/www/app/logs
    - /var/www/app/config
```

Explanation:

| Element | Meaning |
|---|---|
loop | list of values |
item | current value |

Execution becomes:

```
create /var/www/app
create /var/www/app/logs
create /var/www/app/config
```

---

### 5. Using Conditionals with Modules

Sometimes tasks should only run **under certain conditions**.

Example:

Install nginx **only on Ubuntu servers**.

Example playbook:

```yaml
- name: Install nginx on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"
```

Where does `ansible_distribution` come from?

From **gather facts module**.

---

### 6. Using Gathered Facts

At the start of every playbook Ansible runs:

```
Gathering Facts
```

This collects system information.

Example facts:

| Fact | Example |
|---|---|
ansible_distribution | Ubuntu |
ansible_hostname | web1 |
ansible_processor_vcpus | 4 |
ansible_memory_mb | RAM info |

Example usage:

```yaml
- name: Display server OS
  debug:
    msg: "Server OS is {{ ansible_distribution }}"
```

---

### 7. Registering Module Output

Sometimes we want to **capture module output**.

We use `register`.

Example:

```yaml
- name: Check nginx version
  command: nginx -v
  register: nginx_output
```

Now the output is stored in:

```
nginx_output
```

We can print it:

```yaml
- name: Show nginx version
  debug:
    var: nginx_output
```

---

### 8. Example: Restart Service Only When Needed

Real-world requirement:

Restart nginx **only if configuration changed**.

Example:

```yaml
- name: Copy nginx config
  copy:
    src: nginx.conf
    dest: /etc/nginx/nginx.conf
  register: config_status

- name: Restart nginx if config changed
  service:
    name: nginx
    state: restarted
  when: config_status.changed
```

Explanation:

```
config_status.changed
```

becomes **true** if file changed.

This prevents unnecessary restarts.

---

### 9. Real-World Complete Deployment Playbook

Now we combine everything we learned.

Goal:

Automate full web server setup.

### Project Structure

```
ansible/
│
├── hosts
├── ansible.cfg
├── deploy.yml
└── files/
    └── index.html
```

---

### 10. Complete Deployment Playbook

```yaml
---
- name: Deploy Web Application
  hosts: webserver

  vars:
    app_dir: /var/www/app

  tasks:

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Create application directories
      file:
        path: "{{ item }}"
        state: directory
      loop:
        - "{{ app_dir }}"
        - "{{ app_dir }}/logs"
        - "{{ app_dir }}/config"

    - name: Copy website page
      copy:
        src: files/index.html
        dest: "{{ app_dir }}/index.html"

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
```

---

### 11. Running the Playbook

Execute:

```
ansible-playbook -i hosts deploy.yml
```

Process:

```
1 Gather facts
2 Install nginx
3 Create directories
4 Copy files
5 Start nginx
```

All servers are configured automatically.

---

### 12. Verifying Deployment

SSH into a server:

```
ssh root@server-ip
```

Check nginx:

```
systemctl status nginx
```

Check web directory:

```
ls /var/www/app
```

---

### 13. Why This Matters in Real Production

Using modules with loops and conditions allows automation to scale.

Example real infrastructure:

```
10 web servers
5 application servers
3 database servers
```

Automation benefits:

| Without Automation | With Ansible |
|---|---|
Manual configuration | Automated |
Hours of work | Minutes |
Human errors | Consistent configuration |

---

### 14. Advanced Production Usage

In large infrastructures modules automate tasks such as:

### Cloud Infrastructure

Create EC2 servers

```
amazon.aws.ec2_instance
```

---

### Container Deployment

Manage Docker containers

```
community.docker.docker_container
```

---

### Kubernetes

Deploy applications

```
kubernetes.core.k8s
```

---

### 15. Summary

We expanded our understanding of **Ansible modules** into real-world automation.

New concepts learned:

### Module Parameters

Control how modules execute tasks.

### Variables

Allow dynamic playbooks.

### Loops

Execute tasks multiple times.

### Conditionals

Run tasks only when needed.

### Register

Store module output.

### Gather Facts

Provide system information.

Using these techniques, Ansible can automate:

- server provisioning
- application deployment
- container infrastructure
- cloud resources
- full DevOps pipelines

