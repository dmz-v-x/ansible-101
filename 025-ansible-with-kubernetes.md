## Ansible with Kubernetes

Modern infrastructure often runs applications on **Kubernetes clusters** instead of directly on servers.  
While Kubernetes manages containers and orchestration, Ansible can automate:

- Kubernetes cluster setup
- application deployment
- configuration management
- environment provisioning
- CI/CD automation
- operational tasks

In other words:

```
Ansible = automation engine
Kubernetes = container orchestration platform
```

Ansible can interact with Kubernetes through **modules and collections** to manage Kubernetes resources programmatically.

---

### 1. Quick Kubernetes Fundamentals

Before understanding Ansible integration, we must understand basic Kubernetes components.

Kubernetes manages **containerized applications**.

Basic architecture:

```
Control Plane
     |
Worker Nodes
     |
Pods (Containers)
```

Key Kubernetes objects:

| Object | Purpose |
|------|------|
Pod | smallest deployable container unit |
Deployment | manages pods |
Service | exposes applications |
ConfigMap | configuration data |
Secret | sensitive data |
Namespace | logical cluster separation |

These resources are typically defined in **YAML manifests**.

Example Kubernetes Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
```

Normally applied using:

```
kubectl apply -f deployment.yaml
```

Ansible can automate the same process.

---

### 2. Why Use Ansible with Kubernetes?

Kubernetes already has `kubectl`, so why use Ansible?

Ansible provides additional automation benefits:

| Benefit | Explanation |
|------|------|
Infrastructure automation | create clusters and configure nodes |
Application deployment | automate container deployments |
CI/CD integration | pipelines deploy via Ansible |
Multi-environment management | dev, staging, production |
Idempotent deployments | safe repeated execution |

Example automation:

```
Provision infrastructure
Install Kubernetes
Deploy application
Configure monitoring
```

All done using **Ansible playbooks**.

---

### 3. Kubernetes Automation Approaches

Ansible can manage Kubernetes in two main ways:

### Approach 1: Using kubectl commands

Example:

```yaml
- name: Apply Kubernetes manifest
  command: kubectl apply -f deployment.yaml
```

This approach works but is not ideal.

---

### Approach 2: Using Kubernetes modules (Recommended)

Ansible provides native modules for Kubernetes resources.

Example:

```
kubernetes.core.k8s
```

This module allows managing Kubernetes resources directly.

---

### 4. Required Ansible Collection

Kubernetes automation requires the **kubernetes.core collection**.

Install it:

```
ansible-galaxy collection install kubernetes.core
```

This collection contains modules for managing Kubernetes objects.

---

### 5. Prerequisites

Before using Ansible with Kubernetes:

Required tools:

```
Kubernetes cluster
kubectl configured
Python Kubernetes client
Ansible installed
```

Install Python Kubernetes client:

```
pip install kubernetes
```

---

### 6. Using the k8s Module

The most important module is:

```
kubernetes.core.k8s
```

It allows managing any Kubernetes resource.

Example playbook:

```yaml
- name: Deploy nginx to Kubernetes
  hosts: localhost
  tasks:

    - name: Create deployment
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: apps/v1
          kind: Deployment
          metadata:
            name: nginx-deployment
          spec:
            replicas: 3
```

This creates a Kubernetes deployment.

---

### 7. Deploying Kubernetes YAML Files

Instead of embedding YAML inside playbooks, we can deploy manifest files.

Example:

```yaml
- name: Apply Kubernetes manifest
  kubernetes.core.k8s:
    state: present
    src: deployment.yaml
```

Equivalent to:

```
kubectl apply -f deployment.yaml
```

But fully automated through Ansible.

---

### 8. Creating Kubernetes Services

Example playbook:

```yaml
- name: Create service
  kubernetes.core.k8s:
    state: present
    definition:
      apiVersion: v1
      kind: Service
      metadata:
        name: nginx-service
      spec:
        selector:
          app: nginx
```

This exposes the application.

---

### 9. Managing Namespaces

Namespaces separate environments.

Example:

```yaml
- name: Create namespace
  kubernetes.core.k8s:
    state: present
    definition:
      apiVersion: v1
      kind: Namespace
      metadata:
        name: staging
```

Applications can then be deployed to this namespace.

---

### 10. Deploying Applications with Ansible

Example deployment playbook:

```yaml
- name: Deploy application
  hosts: localhost

  tasks:

    - name: Create namespace
      kubernetes.core.k8s:
        state: present
        src: namespace.yaml

    - name: Deploy application
      kubernetes.core.k8s:
        state: present
        src: deployment.yaml

    - name: Expose service
      kubernetes.core.k8s:
        state: present
        src: service.yaml
```

This automates application deployment.

---

### 11. Real DevOps Deployment Workflow

Example automation pipeline:

```
Developer pushes code
       |
CI pipeline builds Docker image
       |
Image pushed to container registry
       |
Ansible playbook runs
       |
Application deployed to Kubernetes
```

Ansible becomes the deployment automation layer.

---

### 12. Managing Kubernetes Secrets

Example:

```yaml
- name: Create secret
  kubernetes.core.k8s:
    state: present
    definition:
      apiVersion: v1
      kind: Secret
      metadata:
        name: db-secret
```

Often combined with **Ansible Vault**.

---

### 13. Scaling Applications

Example:

```yaml
- name: Scale deployment
  kubernetes.core.k8s:
    kind: Deployment
    name: nginx-deployment
    namespace: default
    replicas: 5
```

This updates replica count.

Equivalent to:

```
kubectl scale deployment nginx-deployment --replicas=5
```

---

### 14. Rolling Updates with Ansible

Ansible can trigger rolling updates.

Example:

```yaml
- name: Update image version
  kubernetes.core.k8s:
    state: present
    src: deployment.yaml
```

If the container image changes, Kubernetes performs rolling update.

---

### 15. Real World Example Architecture

Typical enterprise automation:

```
Git Repository
       |
CI/CD Pipeline
       |
Docker Image Build
       |
Container Registry
       |
Ansible Deployment
       |
Kubernetes Cluster
       |
Pods Running Application
```

Ansible acts as the **deployment orchestrator**.

---

### 16. Advanced Kubernetes Automation

Ansible can also automate:

- cluster provisioning
- node configuration
- Helm deployments
- cluster upgrades
- monitoring setup

Example tools used with Ansible:

```
Kubespray
Helm
ArgoCD
Terraform
```

---

### 17. Kubespray (Kubernetes Installation via Ansible)

Kubespray is a popular project that uses Ansible to install Kubernetes clusters.

Architecture:

```
Ansible control node
       |
SSH
       |
Cluster nodes
```

It automates:

- etcd setup
- control plane
- worker nodes
- networking
- storage

---

### 18. Best Practices

1. Use Kubernetes collections for automation  
2. Store manifests in version control  
3. Use Ansible roles for deployments  
4. Integrate with CI/CD pipelines  
5. Use namespaces for environments  

---

### 19. Common Mistakes

### Using kubectl inside playbooks

Prefer native modules:

```
kubernetes.core.k8s
```

---

### Hardcoding configuration

Use templates for dynamic manifests.

---

### Ignoring secrets management

Use:

```
Ansible Vault
Kubernetes Secrets
```
