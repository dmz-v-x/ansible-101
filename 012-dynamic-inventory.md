## Ansible Dynamic Inventory — Complete Guide from Absolute Beginner to Advanced

In modern cloud environments, servers are **not static**. They are constantly:

- created
- destroyed
- scaled up
- scaled down

Because of this, maintaining a **manual list of servers (static inventory)** becomes impractical.

Dynamic inventory solves this problem by allowing Ansible to **automatically discover infrastructure**.

---

### 1. What is Dynamic Inventory?

Dynamic Inventory means:

> The inventory of hosts is generated **automatically at runtime** instead of being manually written.

Example traditional static inventory:

```
[webservers]
192.168.1.10
192.168.1.11
```

But in cloud infrastructure:

- IP addresses change
- Instances are created automatically
- Instances are destroyed automatically

Dynamic inventory solves this by **querying infrastructure APIs**.

Example sources:

- AWS
- Azure
- Google Cloud
- Kubernetes
- VMware

Instead of storing IPs in a file, Ansible **fetches them dynamically**.

---

### 2. Real World Problem with Static Inventory

Consider an AWS Auto Scaling Group.

Infrastructure:

```
Load Balancer
     |
Auto Scaling Group
     |
EC2 Instances
```

Example:

```
Instance1 → created
Instance2 → created
Instance3 → terminated
Instance4 → created
```

Static inventory becomes outdated quickly.

Example static file:

```
[webservers]
54.10.1.2
54.10.1.3
```

If AWS launches a new instance:

```
54.10.1.4
```

Ansible will **not know about it**.

Dynamic inventory automatically discovers:

- new instances
- removed instances
- updated IP addresses

---

### 3. What Dynamic Inventory Does

Dynamic inventory connects to a **cloud provider API** and retrieves:

- instance IDs
- public IP addresses
- private IP addresses
- instance tags
- instance states
- DNS names

Example AWS data retrieved:

```
instance-id
instance-type
public-ip
private-ip
tags
region
```

Ansible then converts this information into **inventory groups automatically**.

---

### 4. Dynamic Inventory Workflow

Dynamic inventory works like this:

```
Ansible
   |
Inventory Plugin
   |
Cloud API (AWS / Azure / GCP)
   |
Retrieve Instances
   |
Generate Inventory
```

Result:

Ansible now has a **live inventory of servers**.

---

### 5. Two Methods for Dynamic Inventory

Ansible supports two dynamic inventory approaches.

| Method | Description |
|------|------|
Inventory Plugins | Modern method (YAML based) |
Inventory Scripts | Older method (Python scripts) |

Today the **recommended approach is inventory plugins**.

---

# INVENTORY PLUGINS

### 6. What are Inventory Plugins?

Inventory plugins are built-in components that fetch infrastructure data.

Examples:

| Plugin | Purpose |
|---|---|
aws_ec2 | AWS EC2 inventory |
azure_rm | Azure VMs |
gcp_compute | Google Cloud VMs |
kubernetes | Kubernetes nodes |

Plugins are configured using **YAML files**.

---

### 7. AWS EC2 Dynamic Inventory Plugin

One of the most common plugins is:

```
aws_ec2
```

This plugin:

- connects to AWS API
- retrieves EC2 instances
- generates Ansible inventory automatically

---

### 8. Prerequisites for AWS Dynamic Inventory

Before using AWS dynamic inventory we must install dependencies.

Required versions:

```
Python >= 3.6
boto3 >= 1.26.0
botocore >= 1.29.0
```

Install them:

```
pip install boto3 botocore
```

These libraries allow Ansible to communicate with AWS APIs.

---

### 9. Step 1 — Enable the aws_ec2 Plugin

In Ansible configuration we enable the plugin.

Create or edit:

```
ansible.cfg
```

Example:

```ini
[defaults]
enable_plugins = aws_ec2
remote_user = ec2-user
private_key_file = ~/.ssh/id_rsa
```

Explanation:

| Option | Meaning |
|---|---|
enable_plugins | activate inventory plugin |
remote_user | SSH user for EC2 |
private_key_file | SSH key for authentication |

---

### 10. Step 2 — Create Plugin Configuration File

Next we create a **plugin configuration file**.

File name must end with:

```
aws_ec2.yaml
```

Example:

```
inventory_aws_ec2.yaml
```

Example content:

```yaml
plugin: aws_ec2
regions:
  - eu-central-1
  - ap-south-1
  - us-east-1
```

Explanation:

| Parameter | Meaning |
|---|---|
plugin | plugin name |
regions | AWS regions to query |

Ansible will scan these regions and find EC2 instances.

---

### 11. Step 3 — Test the Dynamic Inventory

We can test the inventory using:

```
ansible-inventory -i inventory_aws_ec2.yaml --list
```

This command prints the generated inventory.

Example output:

```
{
 "_meta": {
   "hostvars": {}
 },
 "aws_ec2": {
   "hosts": [
      "18.192.209.223",
      "18.193.88.102"
   ]
 }
}
```

This confirms Ansible discovered the EC2 instances.

---

### 12. Step 4 — Using Dynamic Inventory in Playbooks

Now we can run playbooks using dynamic inventory.

Example:

```
ansible-playbook -i inventory_aws_ec2.yaml deploy.yml
```

Inside the playbook we target:

```
hosts: aws_ec2
```

Example playbook:

```yaml
---
- name: Configure AWS servers
  hosts: aws_ec2

  tasks:
    - name: Install Docker
      apt:
        name: docker.io
        state: present
```

This runs tasks on all EC2 instances discovered.

---

### 13. Making Dynamic Inventory Default

Instead of specifying inventory every time:

```
-i inventory_aws_ec2.yaml
```

We can make it default.

Edit:

```
ansible.cfg
```

Example:

```ini
[defaults]
inventory = inventory_aws_ec2.yaml
```

Now we can simply run:

```
ansible-playbook deploy.yml
```

---

### 14. Filtering Instances Using Tags

Often we don't want to manage **all instances**.

Instead we filter by **tags**.

Example AWS tags:

```
Name = dev-server-1
Name = dev-server-2
Name = prod-server-1
```

We configure filters.

Example:

```yaml
plugin: aws_ec2
regions:
  - eu-central-1
  - ap-south-1
  - us-east-1

filters:
  tag:Name: dev*
  instance-state-name: running
```

Explanation:

| Filter | Meaning |
|---|---|
tag:Name | match instance tag |
dev* | prefix match |
instance-state-name | only running instances |

Result:

Only instances like:

```
dev-server-1
dev-server-2
```

are included.

---

### 15. Real World Dynamic Inventory Architecture

Example production environment:

```
AWS Account
   |
Auto Scaling Group
   |
EC2 Instances
```

Instances dynamically scale:

```
2 → 5 → 10 → 3
```

Dynamic inventory automatically updates.

Playbook example:

```
ansible-playbook deploy-app.yml
```

New instances automatically receive:

- application installation
- configuration
- updates

No manual inventory updates required.

---

### 16. Inventory Scripts (Legacy Method)

Before inventory plugins existed, dynamic inventory used **Python scripts**.

Example:

```
aws_inventory.py
```

Script outputs JSON inventory.

Example:

```
python aws_inventory.py --list
```

However this method is now **deprecated in favor of plugins**.

Plugins are easier to maintain.

---

### 17. Common Dynamic Inventory Gotchas

#### AWS Credentials Missing

If credentials are missing:

```
Unable to locate credentials
```

Fix by configuring:

```
~/.aws/credentials
```

---

#### Wrong Region

If region not specified correctly:

No instances found.

Always verify:

```
regions:
  - ap-south-1
```

---

#### Plugin Name Incorrect

Plugin config file must contain:

```
plugin: aws_ec2
```

Otherwise inventory fails.

---

### 18. Best Practices for Dynamic Inventory

1. Always filter instances using tags
2. Separate dev/staging/prod environments
3. Use IAM roles for authentication
4. Avoid managing all instances blindly
5. Store inventory configuration in Git

Example structure:

```
inventory/
   aws_ec2.yaml
playbooks/
   deploy.yml
ansible.cfg
```



Dynamic inventory is essential for **cloud-native DevOps automation**, enabling Ansible to manage **constantly changing infrastructure** without manual updates.

```
