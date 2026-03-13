## Ansible Collections 

As Ansible grew, thousands of modules, plugins, and roles were created. Managing all of these inside the core Ansible project became difficult.  
To solve this, Ansible introduced **Collections**, a packaging system that organizes and distributes Ansible content.

Collections are now the **modern standard way of packaging and sharing Ansible automation**.

They allow developers and organizations to bundle together:

- modules
- roles
- plugins
- playbooks
- documentation

into reusable packages.


---

### 1. Why Ansible Collections Were Introduced

Before collections existed, Ansible content was distributed in two main ways:

1. **Core Modules**
2. **Roles (via Ansible Galaxy)**

Problems with the old model:

- All modules were stored inside the Ansible core repository
- Updating modules required upgrading the entire Ansible installation
- Third-party modules were difficult to distribute
- Namespace conflicts occurred between modules

Collections solved these problems by introducing a **modular distribution system**.

---

### 2. What is an Ansible Collection?

An **Ansible Collection** is a structured package that contains automation content.

A collection may include:

- modules
- roles
- plugins
- playbooks
- documentation

Collections are published on **Ansible Galaxy** or private repositories.

Example collections:

```
amazon.aws
community.general
kubernetes.core
```

Each collection belongs to a **namespace** and a **name**.

Example:

```
namespace.collection
amazon.aws
```

---

### 3. Collection Naming Structure

Collections follow a two-part naming format:

```
namespace.collection_name
```

Example:

```
amazon.aws
community.mysql
kubernetes.core
```

Where:

| Part | Meaning |
|-----|------|
namespace | organization or author |
collection | package name |

Example:

```
amazon.aws
```

```
amazon → namespace
aws → collection
```

---

### 4. What Can a Collection Contain?

Collections organize many types of Ansible components.

Common components include:

| Component | Purpose |
|------|------|
modules | perform tasks |
roles | reusable automation logic |
plugins | extend Ansible functionality |
playbooks | automation workflows |
documentation | usage guides |

Example collection structure:

```
collection_name/
│
├── plugins/
│   ├── modules/
│   ├── inventory/
│   ├── filter/
│
├── roles/
│
├── playbooks/
│
├── docs/
│
└── galaxy.yml
```

---

### 5. Real Example of Popular Collections

Commonly used collections include:

| Collection | Purpose |
|------|------|
amazon.aws | manage AWS resources |
community.docker | manage Docker containers |
kubernetes.core | manage Kubernetes clusters |
community.mysql | manage MySQL databases |

Example:

Install AWS collection:

```
ansible-galaxy collection install amazon.aws
```

---

### 6. Installing Collections

Collections are installed using the **ansible-galaxy** command.

Example:

```
ansible-galaxy collection install community.general
```

This downloads the collection and installs it locally.

Installed location:

```
~/.ansible/collections/
```

or

```
/usr/share/ansible/collections/
```

---

### 7. Listing Installed Collections

To view installed collections:

```
ansible-galaxy collection list
```

Example output:

```
amazon.aws        6.0.0
community.general 7.3.0
kubernetes.core   3.1.0
```

---

### 8. Using Modules from Collections

Modules inside collections must reference the collection namespace.

Example:

Old module usage:

```
ec2
```

New usage:

```
amazon.aws.ec2_instance
```

Example playbook:

```yaml
- name: Launch EC2 instance
  amazon.aws.ec2_instance:
    name: webserver
    instance_type: t2.micro
```

---

### 9. Importing Collections in Playbooks

You can import collections in playbooks to avoid long module names.

Example:

```yaml
collections:
  - amazon.aws
```

Now modules can be referenced like:

```
ec2_instance
```

instead of:

```
amazon.aws.ec2_instance
```

Example playbook:

```yaml
- hosts: localhost
  collections:
    - amazon.aws

  tasks:
    - name: Create EC2 instance
      ec2_instance:
        name: test-server
```

---

### 10. Installing Collections via requirements.yml

Large projects often manage dependencies using a requirements file.

Example:

```
requirements.yml
```

```yaml
collections:
  - name: amazon.aws
  - name: community.docker
  - name: kubernetes.core
```

Install all collections:

```
ansible-galaxy collection install -r requirements.yml
```

---

### 11. Creating Your Own Collection

Organizations often create custom collections.

Create a new collection:

```
ansible-galaxy collection init mycompany.mycollection
```

Example structure:

```
mycompany/mycollection/
│
├── docs/
├── plugins/
├── roles/
├── playbooks/
└── galaxy.yml
```

---

### 12. galaxy.yml File

Every collection contains metadata inside `galaxy.yml`.

Example:

```yaml
namespace: mycompany
name: webautomation
version: 1.0.0
author: DevOps Team
description: Internal automation tools
```

This defines the collection metadata.

---

### 13. Building a Collection Package

After creating a collection, build the package:

```
ansible-galaxy collection build
```

This creates a `.tar.gz` package.

Example:

```
mycompany-webautomation-1.0.0.tar.gz
```

---

### 14. Installing a Local Collection

Install locally built collections:

```
ansible-galaxy collection install mycompany-webautomation-1.0.0.tar.gz
```

---

### 15. Publishing Collections

Collections can be published to:

- Ansible Galaxy
- private artifact repositories
- internal DevOps platforms

Upload command:

```
ansible-galaxy collection publish mycollection.tar.gz
```

---

### 16. Real World Example

Example automation project:

```
company-infrastructure/
│
├── collections/
│   └── requirements.yml
│
├── playbooks/
│
└── roles/
```

requirements.yml:

```yaml
collections:
  - amazon.aws
  - community.docker
  - kubernetes.core
```

Pipeline installs dependencies before running playbooks.

---

### 17. Collections vs Roles

Roles and collections serve different purposes.

| Feature | Roles | Collections |
|------|------|------|
Reusable tasks | Yes | Yes |
Package modules | No | Yes |
Package plugins | No | Yes |
Namespace support | No | Yes |

Collections are **larger packaging units**.

Roles can exist **inside collections**.

---

### 18. Benefits of Collections

Collections provide many advantages:

- modular automation distribution
- better dependency management
- versioned automation packages
- easier sharing of modules and roles
- namespace isolation

They allow organizations to build **internal automation libraries**.

---

### 19. Common Mistakes

### Not specifying collection dependencies

Always define dependencies using:

```
requirements.yml
```

---

### Hardcoding modules

Use collection namespaces for clarity.

Example:

```
amazon.aws.ec2_instance
```

---

### Not versioning collections

Collections should follow semantic versioning:

```
1.0.0
1.1.0
2.0.0
```

