## Ansible Error Handling 

In automation, failures are inevitable. Servers might be unreachable, packages might fail to install, services may not start, or configuration changes might cause errors.

If Ansible stopped every time something failed, automation would become unreliable and difficult to manage.  
To handle these situations, Ansible provides **error handling mechanisms** that allow playbooks to:

- ignore certain errors
- define custom failure conditions
- recover from failures
- ensure cleanup tasks always run
- retry failed operations
- continue automation safely

---

### 1. Default Behavior of Ansible When Errors Occur

By default, when a task fails on a host:

```
That host stops executing further tasks in the play.
```

Example:

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

If installation fails on a host:

```
Remaining tasks for that host are skipped.
```

However, other hosts continue execution.

Example output:

```
host1 FAILED
host2 OK
host3 OK
```

This default behavior is often desirable, but sometimes we need **more control**.

---

### 2. Ignoring Errors (`ignore_errors`)

Sometimes a task failure is acceptable.

Example:

Trying to stop a service that may not exist.

```yaml
- name: Stop nginx service
  service:
    name: nginx
    state: stopped
  ignore_errors: yes
```

Even if the task fails:

```
Playbook continues execution.
```

Important note:

`ignore_errors` only works for **task failures**, not unreachable hosts.

---

### 3. Handling Unreachable Hosts

If a host cannot be reached via SSH, Ansible marks it as **UNREACHABLE**.

Example:

```
UNREACHABLE! => Failed to connect
```

By default:

```
Host removed from execution
```

To continue using unreachable hosts later:

```
meta: clear_host_errors
```

Example:

```yaml
- meta: clear_host_errors
```

This allows the playbook to attempt communication again.

---

### 4. Defining Custom Failure Conditions (`failed_when`)

Sometimes a command returns success but actually indicates a failure.

Example command:

```
app-status check
```

Output:

```
status: ERROR
```

Even though command executed successfully.

We can define custom failure logic.

Example:

```yaml
- name: Check application status
  command: app-status check
  register: app_status
  failed_when: "'ERROR' in app_status.stdout"
```

Now the task fails if:

```
ERROR appears in output
```

---

### 5. Defining Custom Change Conditions (`changed_when`)

Sometimes commands report changes even when nothing changed.

Example:

```yaml
- name: Run health check
  command: curl localhost
```

Ansible may mark it as changed.

To avoid this:

```yaml
changed_when: false
```

Example:

```yaml
- name: Health check
  command: curl localhost
  changed_when: false
```

Now the task will always show:

```
ok
```

instead of:

```
changed
```

---

### 6. Using Blocks for Error Handling

Ansible provides **block structures** similar to programming languages.

Blocks allow grouping tasks and handling errors gracefully.

Example structure:

```
block
rescue
always
```

---

### 7. Basic Block Example

```yaml
- block:

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started

  rescue:

    - name: Log installation failure
      debug:
        msg: "Nginx installation failed"

  always:

    - name: Print completion message
      debug:
        msg: "Playbook execution finished"
```

Explanation:

| Section | Purpose |
|------|------|
block | normal tasks |
rescue | runs if block fails |
always | runs regardless of success or failure |

---

### 8. Real World Block Example

Example deployment:

```yaml
- block:

    - name: Deploy application
      git:
        repo: https://github.com/company/app.git
        dest: /app

    - name: Restart application
      service:
        name: app
        state: restarted

  rescue:

    - name: Rollback deployment
      command: /scripts/rollback.sh
```

If deployment fails:

```
Rollback automatically executes
```

---

### 9. Retrying Failed Tasks (`until`)

Some tasks fail temporarily.

Example:

- service not ready
- database not started
- network delays

Ansible supports retries.

Example:

```yaml
- name: Wait for application
  command: curl http://localhost:8080
  register: result
  until: result.rc == 0
  retries: 10
  delay: 5
```

Meaning:

```
Retry command every 5 seconds
Maximum 10 attempts
Stop when return code = 0
```

---

### 10. Fail Module (Explicit Failure)

You can explicitly fail a playbook.

Example:

```yaml
- name: Stop deployment if disk full
  fail:
    msg: "Disk space is insufficient"
  when: disk_usage > 90
```

This stops execution.

---

### 11. Assert Module (Validation)

Assert validates conditions.

Example:

```yaml
- name: Validate memory requirement
  assert:
    that:
      - ansible_memtotal_mb > 2048
```

If condition fails:

```
Playbook stops
```

Useful for pre-deployment checks.

---

### 12. Using `any_errors_fatal`

Normally failure affects only one host.

To stop entire play:

```yaml
any_errors_fatal: true
```

Example:

```yaml
- hosts: webservers
  any_errors_fatal: true
```

If one host fails:

```
All hosts stop execution
```

Useful in sensitive deployments.

---

### 13. Using `max_fail_percentage`

Controls failure tolerance.

Example:

```yaml
max_fail_percentage: 20
```

Meaning:

```
If more than 20% hosts fail → stop playbook
```

Useful for large clusters.

---

### 14. Real World Deployment Example

Example production deployment:

```yaml
- hosts: webservers

  tasks:

    - block:

        - name: Deploy new version
          git:
            repo: https://github.com/company/app.git
            dest: /app

        - name: Restart application
          service:
            name: app
            state: restarted

      rescue:

        - name: Rollback deployment
          command: /scripts/rollback.sh

      always:

        - name: Notify monitoring system
          debug:
            msg: "Deployment attempt finished"
```

This ensures:

- rollback occurs if deployment fails
- monitoring notification always happens

---

### 15. Common Error Handling Gotchas

#### Ignoring too many errors

Overusing `ignore_errors` can hide serious issues.

Use carefully.

---

#### Not using retries

Network and service startup failures are often temporary.

Use `until` with retries.

---

#### Missing validation checks

Use `assert` to verify prerequisites before deployment.

---

### 16. Best Practices for Error Handling

1. Use `failed_when` for custom failure detection  
2. Use `changed_when` for command-based tasks  
3. Use `block + rescue` for recovery logic  
4. Use `until` for retry logic  
5. Use `assert` for validation  
6. Avoid excessive `ignore_errors`
