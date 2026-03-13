## Ansible Strategies and Parallel Execution 

When Ansible runs a playbook across many servers, it must decide **how tasks are executed across hosts**.

Questions like these arise:

- Should tasks run **on all servers at the same time**?
- Should tasks run **one server at a time**?
- Should tasks run **in batches**?
- What happens if one server fails?
- How do we control rollout speed during deployments?

These behaviors are controlled using **Ansible execution strategies and parallelism settings**.

Understanding strategies is critical in real infrastructure automation because production environments often contain:

- dozens of servers
- hundreds of servers
- sometimes thousands of servers

Running automation incorrectly could cause **mass outages**.

---

### 1. How Ansible Executes Tasks by Default

Consider this playbook:

```yaml
- hosts: webservers

  tasks:

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Start nginx
      service:
        name: nginx
        state: started
```

Assume the inventory contains:

```
web1
web2
web3
```

Default behavior:

```
TASK 1 runs on ALL hosts
TASK 2 runs on ALL hosts
```

Execution order:

```
Task1 → web1
Task1 → web2
Task1 → web3

Task2 → web1
Task2 → web2
Task2 → web3
```

This is the **default linear execution strategy**.

---

### 2. What is Parallel Execution?

Parallel execution means Ansible can run tasks **on multiple hosts simultaneously**.

Example:

```
Install nginx on 10 servers at once
```

instead of:

```
Install nginx one server at a time
```

Parallelism significantly speeds up automation.

---

### 3. Controlling Parallelism with Forks

Ansible uses a concept called **forks**.

Forks determine **how many hosts Ansible manages simultaneously**.

Default forks value:

```
5
```

Meaning:

```
Ansible can manage 5 hosts at the same time
```

Example:

If inventory has 20 hosts:

```
5 hosts processed
next 5 hosts processed
next 5 hosts processed
next 5 hosts processed
```

---

### 4. Configuring Forks

Forks can be configured in **ansible.cfg**.

Example:

```
[defaults]
forks = 20
```

Or via CLI:

```
ansible-playbook playbook.yml -f 20
```

Higher forks = faster execution.

But too high forks can overload:

- control node
- network
- managed servers

---

### 5. Ansible Execution Strategies

Strategies control **how tasks are executed across hosts**.

Common strategies:

| Strategy | Description |
|---|---|
linear | default sequential strategy |
free | hosts run independently |
debug | step-through execution |

---

### 6. Linear Strategy (Default)

This is the default strategy.

Example:

```
strategy: linear
```

Execution pattern:

```
Task1 → all hosts
Task2 → all hosts
Task3 → all hosts
```

Visualization:

```
Task1: web1 web2 web3
Task2: web1 web2 web3
Task3: web1 web2 web3
```

All hosts complete one task before moving to the next.

This ensures **predictable execution order**.

---

### 7. Free Strategy

Free strategy allows hosts to run **tasks independently**.

Example:

```yaml
- hosts: webservers
  strategy: free
```

Execution pattern:

```
web1 → task1 → task2 → task3
web2 → task1 → task2 → task3
web3 → task1 → task2 → task3
```

Hosts do **not wait for each other**.

This can significantly speed up execution.

However:

```
Execution order becomes unpredictable
```

---

### 8. Example Comparing Linear vs Free

Inventory:

```
web1
web2
web3
```

Linear strategy:

```
Task1 → web1 web2 web3
Task2 → web1 web2 web3
Task3 → web1 web2 web3
```

Free strategy:

```
web1 → task1 task2 task3
web2 → task1 task2 task3
web3 → task1 task2 task3
```

Hosts progress independently.

---

### 9. Serial Execution (Rolling Updates)

Sometimes we **should not update all servers simultaneously**.

Example:

Updating 100 production web servers.

Updating all at once could cause downtime.

Instead we perform **rolling updates**.

Use:

```
serial
```

Example:

```yaml
- hosts: webservers
  serial: 2
```

Meaning:

```
2 servers updated at a time
```

Execution:

```
Batch1 → web1 web2
Batch2 → web3 web4
Batch3 → web5 web6
```

This ensures service availability.

---

### 10. Percentage-Based Serial

Serial can also use percentages.

Example:

```yaml
serial: 20%
```

If there are 50 servers:

```
10 servers updated per batch
```

Useful for large clusters.

---

### 11. Real World Rolling Deployment Example

Example playbook:

```yaml
- hosts: webservers
  serial: 2

  tasks:

    - name: Deploy new application version
      git:
        repo: https://github.com/company/app.git
        dest: /app

    - name: Restart application
      service:
        name: app
        state: restarted
```

This ensures:

```
Only 2 servers restart at a time
```

Production traffic continues uninterrupted.

---

### 12. Using Throttle

Throttle limits concurrency for a specific task.

Example:

```yaml
- name: Restart application
  service:
    name: app
    state: restarted
  throttle: 1
```

Meaning:

```
Only 1 host executes this task at a time
```

Useful for sensitive operations.

---

### 13. Combining Serial and Strategies

Example:

```yaml
- hosts: webservers
  serial: 5
  strategy: linear
```

Execution:

```
5 hosts at a time
Each batch executes tasks sequentially
```

This is common in large deployments.

---

### 14. Real Production Deployment Scenario

Infrastructure:

```
Load Balancer
       |
Web Server Cluster (50 nodes)
```

Deployment plan:

```
Update 5 servers at a time
Ensure application restarts gradually
```

Playbook:

```yaml
- hosts: webservers
  serial: 5

  tasks:

    - name: Pull latest code
      git:
        repo: https://github.com/company/app.git
        dest: /app

    - name: Restart application
      service:
        name: app
        state: restarted
```

Traffic continues flowing to remaining servers.

---

### 15. Debug Strategy

Debug strategy allows **step-by-step execution**.

Example:

```yaml
strategy: debug
```

Useful for troubleshooting.

Each task requires manual confirmation.

---

### 16. Common Parallel Execution Pitfalls

#### Running tasks on all servers simultaneously

Example:

```
Restart database cluster on all nodes
```

This could cause complete outage.

Use:

```
serial
```

---

#### High forks value

Too many forks can cause:

- SSH overload
- CPU exhaustion
- network congestion

---

#### Not using rolling deployments

Always use **serial execution for production updates**.

---

### 17. Best Practices for Production Automation

1. Use `serial` for production deployments  
2. Avoid restarting all services simultaneously  
3. Use forks carefully based on infrastructure size  
4. Use `free` strategy only for independent tasks  
5. Combine strategies with rolling updates  

