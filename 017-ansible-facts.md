## Ansible Facts 

Ansible Facts are one of the most powerful mechanisms in Ansible because they allow playbooks to **discover information about target systems automatically**.

This enables playbooks to become **intelligent and adaptive** instead of static.

For example, a playbook can automatically decide:

- which package manager to use
- which configuration to apply
- how many CPUs a machine has
- what OS it runs
- which network interfaces exist

Facts make Ansible automation **context-aware**.

---

### 1. What Are Ansible Facts?

Ansible Facts are **automatically collected information about managed nodes**.

These facts describe the system configuration.

Examples of information collected:

- Operating system
- Kernel version
- Hostname
- CPU details
- Memory size
- Network interfaces
- Disk devices
- IP addresses

Facts are stored as **variables** that playbooks can use.

Example fact:

```
ansible_distribution
```

Example value:

```
Ubuntu
```

---

### 2. Why Facts Are Important

Facts allow playbooks to behave differently depending on the system.

Example problem:

You want to install nginx.

But different Linux distributions use different package managers.

| OS | Package Manager |
|---|---|
Ubuntu | apt |
CentOS | yum |

Using facts we can automatically detect the OS.

Example:

```
if Ubuntu → use apt
if CentOS → use yum
```

Without facts we would need **multiple playbooks**.

---

### 3. How Ansible Facts Are Gathered

At the start of every playbook run, Ansible executes the **setup module** automatically.

This module collects system information.

Example playbook run:

```
TASK [Gathering Facts]
ok: [web1]
ok: [web2]
```

This step runs before all other tasks.

The gathered data becomes available as **variables**.

---

### 4. Example Playbook Showing Facts

Example:

```yaml
- name: Show OS information
  hosts: all

  tasks:
    - name: Display OS
      debug:
        msg: "Operating system is {{ ansible_distribution }}"
```

Example output:

```
Operating system is Ubuntu
```

---

### 5. Viewing All Facts

You can view all collected facts using:

```
ansible all -m setup
```

This prints a large JSON output.

Example snippet:

```
"ansible_distribution": "Ubuntu"
"ansible_kernel": "5.15.0"
"ansible_memtotal_mb": 8192
```

These values are accessible in playbooks.

---

### 6. Structure of Ansible Facts

Facts are stored in a dictionary called:

```
ansible_facts
```

Example structure:

```
ansible_facts:
  distribution: Ubuntu
  kernel: 5.15
  memory_mb: 8192
```

However, Ansible automatically exposes them as top-level variables.

So both work:

```
ansible_distribution
ansible_facts['distribution']
```

---

### 7. Commonly Used Facts

Below are some important facts used frequently.

| Fact | Description |
|---|---|
ansible_hostname | hostname of system |
ansible_distribution | OS distribution |
ansible_os_family | OS family |
ansible_kernel | kernel version |
ansible_processor_vcpus | CPU count |
ansible_memtotal_mb | total memory |
ansible_default_ipv4.address | main IP |

Example usage:

```
Server IP: {{ ansible_default_ipv4.address }}
```

---

### 8. Using Facts for Conditional Tasks

Facts are commonly used with **conditionals**.

Example:

```yaml
- name: Install nginx on Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Ubuntu"
```

Example for RedHat:

```yaml
- name: Install nginx on CentOS
  yum:
    name: nginx
    state: present
  when: ansible_os_family == "RedHat"
```

This allows one playbook to work across multiple OS types.

---

### 9. Using Facts in Templates

Facts can also be used inside templates.

Example template:

```
Server hostname: {{ ansible_hostname }}
Operating system: {{ ansible_distribution }}
```

Rendered output:

```
Server hostname: web1
Operating system: Ubuntu
```

---

### 10. Disabling Fact Gathering

Fact gathering adds overhead.

In large infrastructures it can slow execution.

You can disable it.

Example:

```yaml
- name: Disable facts
  hosts: all
  gather_facts: false
```

Now the setup module will not run.

Only do this when facts are not needed.

---

### 11. Manually Gathering Facts

If facts are disabled, you can gather them manually.

Example:

```yaml
- name: Collect facts manually
  setup:
```

This runs the setup module explicitly.

---

### 12. Custom Facts

You can define your own facts.

These are called **local facts**.

Example location:

```
/etc/ansible/facts.d/custom.fact
```

Example file content:

```
{
  "app_name": "inventory-service",
  "version": "1.2.3"
}
```

These become available as:

```
ansible_local.custom.app_name
```

Example usage:

```
{{ ansible_local.custom.app_name }}
```

---

### 13. Example Playbook Using Custom Facts

```yaml
- name: Show custom fact
  hosts: all

  tasks:
    - debug:
        msg: "App name is {{ ansible_local.custom.app_name }}"
```

---

### 14. Fact Caching

Fact gathering can be slow when managing **hundreds of servers**.

Ansible supports **fact caching**.

Facts are stored and reused instead of being collected every run.

Example configuration:

```
ansible.cfg
```

```
[defaults]
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_cache
```

Benefits:

- faster execution
- reduced network calls

---

### 15. Real World Example

Example scenario:

A playbook deploys software differently depending on CPU count.

Example:

```yaml
- name: Configure worker processes
  debug:
    msg: "CPU count is {{ ansible_processor_vcpus }}"
```

Example template:

```
worker_processes {{ ansible_processor_vcpus }};
```

This automatically scales configuration.

---

### 16. Example — Multi-OS Deployment

Example playbook:

```yaml
- name: Install nginx on multiple OS
  hosts: all

  tasks:

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

One playbook supports multiple operating systems.

---

### 17. Debugging Facts

Use debug module:

```yaml
- debug:
    var: ansible_hostname
```

Print multiple facts:

```yaml
- debug:
    msg: "{{ ansible_distribution }} {{ ansible_kernel }}"
```

---

### 18. Common Fact Gotchas

#### Facts not available

Occurs when:

```
gather_facts: false
```

---

#### Wrong fact name

Always verify using:

```
ansible all -m setup
```

---

#### Large fact output

Facts can be very large.

Avoid printing all facts unnecessarily.

---

### 19. Best Practices

Follow these guidelines.

1. Use facts for conditional automation  
2. Disable fact gathering if not required  
3. Cache facts in large environments  
4. Avoid excessive fact usage in loops  
5. Use custom facts for application metadata  


