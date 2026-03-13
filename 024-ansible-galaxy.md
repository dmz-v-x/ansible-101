## Ansible Galaxy 

As Ansible adoption grew, thousands of developers and organizations began creating reusable automation components such as **roles and collections**. To make sharing and discovering this automation easier, Ansible introduced **Ansible Galaxy**.

Ansible Galaxy is the **central repository for Ansible automation content**, where users can:

- discover reusable roles and collections
- install automation components
- publish their own roles and collections
- share infrastructure automation with the community

It functions similarly to package repositories like:

- **npm (Node.js)**
- **PyPI (Python)**
- **Docker Hub**
- **Maven Central**

But specifically for **Ansible automation content**.

---

### 1. What is Ansible Galaxy?

**Ansible Galaxy** is a public platform that hosts reusable Ansible content.

Official website:

```
https://galaxy.ansible.com
```

It allows developers to:

- share roles
- distribute collections
- reuse automation components

Example automation packages available:

- nginx setup
- docker installation
- kubernetes cluster configuration
- AWS infrastructure automation

Instead of writing automation from scratch, teams can **reuse existing Galaxy content**.

---

### 2. Why Ansible Galaxy Exists

Before Galaxy existed, sharing automation was difficult.

Teams had to:

- copy playbooks manually
- duplicate roles across projects
- maintain internal repositories

Galaxy solved these issues by providing:

- a centralized automation registry
- versioned packages
- dependency management
- standardized packaging

This dramatically improved **automation reuse across the DevOps community**.

---

### 3. Types of Content in Ansible Galaxy

Galaxy hosts two main types of automation content.

| Type | Description |
|-----|-------------|
Roles | reusable task packages |
Collections | larger packages containing roles, modules, and plugins |

Roles were the original Galaxy content type.

Collections are the **modern packaging system introduced later**.

---

### 4. What Are Galaxy Roles?

A **role** is a structured set of Ansible tasks and files used to perform a specific function.

Example roles:

```
geerlingguy.nginx
geerlingguy.mysql
geerlingguy.docker
```

These roles automate common infrastructure setups.

Example role responsibilities:

- installing nginx
- configuring firewall
- deploying applications

Roles are reusable building blocks for automation.

---

### 5. Installing Roles from Galaxy

Roles can be installed using the **ansible-galaxy command**.

Example:

```
ansible-galaxy role install geerlingguy.nginx
```

This downloads the role into:

```
roles/
```

directory or default role path.

---

### 6. Using Installed Roles

Example playbook using a Galaxy role:

```yaml
- hosts: webservers
  roles:
    - geerlingguy.nginx
```

This role automatically:

- installs nginx
- configures nginx
- starts the service

All tasks defined in the role execute automatically.

---

### 7. Managing Multiple Roles with requirements.yml

Large projects often depend on many roles.

Instead of installing roles individually, we define them in a **requirements file**.

Example:

```
requirements.yml
```

```yaml
roles:
  - name: geerlingguy.nginx
  - name: geerlingguy.mysql
  - name: geerlingguy.docker
```

Install all roles:

```
ansible-galaxy role install -r requirements.yml
```

This ensures consistent dependency installation across environments.

---

### 8. Installing Collections from Galaxy

Collections are installed similarly.

Example:

```
ansible-galaxy collection install amazon.aws
```

Installed collections are stored in:

```
~/.ansible/collections/
```

or

```
/usr/share/ansible/collections/
```

---

### 9. Using requirements.yml for Collections

Collections can also be managed using requirements files.

Example:

```yaml
collections:
  - name: amazon.aws
  - name: community.docker
  - name: kubernetes.core
```

Install them:

```
ansible-galaxy collection install -r requirements.yml
```

---

### 10. Searching Ansible Galaxy

You can search Galaxy for roles and collections.

Example command:

```
ansible-galaxy search nginx
```

Or browse on the website:

```
https://galaxy.ansible.com
```

Galaxy displays:

- popularity
- downloads
- documentation
- supported platforms

This helps identify reliable automation components.

---

### 11. Creating Your Own Role

Developers can publish their own roles.

Create a role:

```
ansible-galaxy role init myrole
```

Generated structure:

```
myrole/
├── defaults
├── files
├── handlers
├── meta
├── tasks
├── templates
└── vars
```

Develop automation inside this structure.

---

### 12. Publishing Roles to Galaxy

Steps to publish a role:

1. Create role
2. Push role to GitHub
3. Import repository into Ansible Galaxy

Galaxy automatically pulls role code from GitHub.

Users can then install your role.

---

### 13. Creating Your Own Collection

Collections are created using:

```
ansible-galaxy collection init namespace.collection_name
```

Example:

```
ansible-galaxy collection init mycompany.infrastructure
```

Generated structure:

```
mycompany/infrastructure/
├── plugins/
├── roles/
├── docs/
└── galaxy.yml
```

---

### 14. Building a Collection Package

Build the collection:

```
ansible-galaxy collection build
```

This creates:

```
namespace-collection-version.tar.gz
```

Example:

```
mycompany-infrastructure-1.0.0.tar.gz
```

---

### 15. Publishing Collections

Publish collections to Galaxy:

```
ansible-galaxy collection publish mycollection.tar.gz
```

Once published, others can install it.

---

### 16. Private Galaxy Servers

Organizations often host **private Galaxy repositories**.

Reasons:

- internal automation libraries
- security restrictions
- proprietary infrastructure modules

Examples:

- Red Hat Automation Hub
- Artifactory
- internal DevOps platforms

This allows companies to share automation internally.

---

### 17. Real World DevOps Usage

Example enterprise project:

```
company-automation/
│
├── playbooks/
│
├── roles/
│
├── collections/
│
└── requirements.yml
```

requirements.yml:

```yaml
roles:
  - geerlingguy.nginx
  - geerlingguy.mysql

collections:
  - amazon.aws
  - community.docker
```

CI/CD pipeline installs dependencies before deployment.

---

### 18. Best Practices for Using Galaxy

1. Use trusted and well-maintained roles
2. Pin specific versions of roles and collections
3. Use requirements files for dependency management
4. Review role code before using in production
5. Prefer collections for modern automation

---

### 19. Common Mistakes

**Blindly installing roles**

Always review role code before using it.

---

**Not version-locking dependencies**

Roles may change behavior across versions.

---

**Using outdated roles**

Prefer actively maintained Galaxy projects.
