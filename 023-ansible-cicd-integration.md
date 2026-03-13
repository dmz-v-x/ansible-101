## Ansible CI/CD Integration

Modern software delivery requires applications to be **built, tested, and deployed automatically**.  
This process is known as **CI/CD (Continuous Integration and Continuous Deployment/Delivery)**.

Ansible is commonly used as the **deployment and infrastructure automation tool inside CI/CD pipelines**.

Instead of manually deploying applications to servers, CI/CD systems trigger Ansible playbooks that:

- provision infrastructure
- configure servers
- deploy applications
- update services
- run migrations
- verify deployments

---

## 1. What is CI/CD?

CI/CD is a development practice that automates software delivery.

Two main components exist:

### Continuous Integration (CI)

CI ensures that code changes are:

- automatically built
- automatically tested
- verified before merging

Example workflow:

```
Developer pushes code
      ↓
Build application
      ↓
Run automated tests
      ↓
Create build artifact
```

---

### Continuous Deployment / Continuous Delivery (CD)

CD automates application deployment.

Example workflow:

```
Build artifact created
      ↓
Deploy to staging
      ↓
Run tests
      ↓
Deploy to production
```

---

## 2. Where Ansible Fits in CI/CD

Ansible is mainly used in the **deployment phase**.

Example CI/CD pipeline:

```
Code Commit
     ↓
CI Build + Test
     ↓
Artifact Created
     ↓
Ansible Deployment
     ↓
Infrastructure Configuration
     ↓
Application Running
```

Ansible handles:

- server configuration
- application deployment
- infrastructure changes

---

## 3. Why Use Ansible in CI/CD

Ansible is widely used in pipelines because it provides:

- **Infrastructure as Code**
- **Idempotent deployments**
- **Environment consistency**
- **Multi-server automation**
- **Agentless architecture**

Example scenario:

Deploying a new version of an application to **50 servers**.

Instead of manual deployment:

```
SSH into each server
Deploy manually
Restart services
```

CI/CD triggers an Ansible playbook:

```
ansible-playbook deploy.yml
```

Deployment happens automatically.

---

## 4. Typical CI/CD Architecture with Ansible

Example architecture:

```
Developers
     |
Git Repository
     |
CI/CD Pipeline (Jenkins / GitHub Actions / GitLab CI)
     |
Ansible Playbooks
     |
Target Infrastructure
(Web Servers, DB Servers, Cloud Instances)
```

Pipeline triggers Ansible automation.

---

## 5. Example CI/CD Pipeline Stages

Typical pipeline stages:

```
1 Source Code Checkout
2 Build Application
3 Run Tests
4 Package Artifact
5 Deploy Using Ansible
6 Run Post-Deployment Tests
```

Example flow:

```
git push
      ↓
CI build
      ↓
Docker image built
      ↓
Ansible deploys containers
```

---

## 6. Tools Commonly Used with Ansible in CI/CD

Many CI/CD tools integrate with Ansible.

Common tools include:

| Tool | Purpose |
|----|----|
Jenkins | CI/CD server |
GitHub Actions | CI/CD pipeline automation |
GitLab CI | CI/CD integrated with GitLab |
Azure DevOps | Microsoft CI/CD platform |
CircleCI | Cloud CI/CD tool |

Ansible works with all of them.

---

## 7. Example Project Structure for CI/CD

Typical repository structure:

```
project/
│
├── ansible/
│   ├── inventory/
│   │   ├── dev
│   │   └── prod
│   │
│   ├── roles/
│   │   ├── nginx
│   │   └── app
│   │
│   ├── deploy.yml
│   └── ansible.cfg
│
├── app/
│
├── Dockerfile
│
└── pipeline.yml
```

CI/CD pipeline executes playbooks inside `ansible/`.

---

## 8. Example Deployment Playbook

Example deployment playbook:

```yaml
- name: Deploy Application
  hosts: webservers
  become: true

  tasks:

    - name: Pull latest code
      git:
        repo: https://github.com/company/app.git
        dest: /app

    - name: Install dependencies
      npm:
        path: /app

    - name: Restart application
      service:
        name: app
        state: restarted
```

CI/CD pipeline runs:

```
ansible-playbook deploy.yml
```

---

## 9. Example Jenkins Pipeline Using Ansible

Example Jenkins pipeline:

```
pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/company/app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Deploy') {
            steps {
                sh 'ansible-playbook ansible/deploy.yml'
            }
        }

    }

}
```

Pipeline automatically deploys application using Ansible.

---

## 10. Example GitHub Actions Workflow

Example GitHub Actions pipeline:

```
.github/workflows/deploy.yml
```

```yaml
name: Deploy Application

on:
  push:
    branches:
      - main

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout code
        uses: actions/checkout@v3

      - name: Install Ansible
        run: sudo apt install ansible -y

      - name: Run playbook
        run: ansible-playbook ansible/deploy.yml
```

Whenever code is pushed to `main`, deployment runs.

---

## 11. Deploying Docker Applications with Ansible

Example playbook:

```yaml
- name: Deploy container
  hosts: webservers

  tasks:

    - name: Pull docker image
      docker_container:
        name: app
        image: company/app:latest
        state: started
```

CI pipeline builds the image.

Ansible deploys the container.

---

## 12. Rolling Deployments with Ansible

Production deployments should avoid downtime.

Use rolling updates:

```yaml
- hosts: webservers
  serial: 2
```

This ensures:

```
2 servers updated at a time
```

Traffic remains available.

---

## 13. Secrets Management in CI/CD

Secrets such as passwords must not be exposed.

Common approaches:

```
Ansible Vault
CI/CD secret storage
Environment variables
Secret managers
```

Example:

```
ansible-playbook deploy.yml --vault-password-file vault_pass.txt
```

---

## 14. Post Deployment Validation

After deployment, pipelines often run verification tasks.

Example:

```yaml
- name: Check application health
  uri:
    url: http://localhost:8080/health
    status_code: 200
```

Ensures application started successfully.

---

## 15. Real World CI/CD Deployment Example

Example infrastructure:

```
GitHub Repository
        |
GitHub Actions Pipeline
        |
Docker Build
        |
Push Image to Registry
        |
Ansible Playbook Deploys Containers
        |
Production Servers Updated
```

Everything happens automatically.

---

## 16. Best Practices for Ansible CI/CD Integration

1. Store playbooks in version control  
2. Separate dev, staging, and production inventories  
3. Use Ansible roles for modular automation  
4. Use vault for secrets  
5. Implement rolling deployments  
6. Add post-deployment health checks  
7. Keep CI/CD pipelines simple and reliable  

---

## 17. Common CI/CD Mistakes

### Deploying directly to production

Always use environments:

```
dev → staging → production
```

---

### Hardcoding secrets

Use:

```
Ansible Vault
CI/CD secret storage
```

---

### Running entire playbooks every time

Use **tags** for partial deployments.

---

## 18. Example Full CI/CD Flow with Ansible

Example flow:

```
Developer pushes code
       ↓
CI builds application
       ↓
Run automated tests
       ↓
Docker image created
       ↓
Image pushed to registry
       ↓
Ansible deploys container
       ↓
Rolling deployment executed
       ↓
Health checks performed
```

This is a common **DevOps automation pipeline**.

---

## 19. Advantages of Using Ansible in CI/CD

Ansible provides:

- automated deployments
- infrastructure consistency
- faster releases
- safer production updates
- reproducible infrastructure

It bridges the gap between:

```
Application code
Infrastructure automation
```
