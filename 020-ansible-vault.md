## Ansible Vault

In real infrastructure automation, playbooks often contain **sensitive information** such as:

- database passwords  
- API keys  
- SSH private keys  
- cloud credentials  
- TLS certificates  
- application secrets  

Storing these secrets in **plain text** inside playbooks or variable files is extremely dangerous.

Anyone with access to the repository could see them.

To solve this problem, Ansible provides **Ansible Vault**, a feature that allows you to **encrypt sensitive data inside Ansible projects**.

Vault enables you to safely store secrets while still allowing automation to run normally.

---

### 1. Why Secret Management is Necessary

Imagine the following playbook:

```yaml
vars:
  db_user: admin
  db_password: mysecretpassword
```

If this playbook is pushed to GitHub:

```
anyone can see the password
```

This creates serious security risks:

- database compromise  
- production system compromise  
- leaked credentials  

To avoid this, secrets must be **encrypted**.

---

### 2. What is Ansible Vault?

**Ansible Vault** is a feature that encrypts files and variables using strong encryption.

It allows secrets to be stored safely inside an Ansible project.

Encrypted files look like this:

```
$ANSIBLE_VAULT;1.1;AES256
6166383631343532633730316331346630...
```

Only users with the **Vault password** can decrypt the content.

---

### 3. How Ansible Vault Works

Workflow:

```
Sensitive Data
      |
ansible-vault encrypt
      |
Encrypted File
      |
Playbook execution
      |
Ansible decrypts using vault password
```

Encryption algorithm used:

```
AES256
```

This is a strong industry-grade encryption standard.

---

### 4. Basic Vault Commands

Ansible Vault provides several commands.

| Command | Purpose |
|---|---|
encrypt | encrypt file |
decrypt | decrypt file |
edit | edit encrypted file |
view | view encrypted file |
rekey | change vault password |

---

### 5. Encrypting a File

Example file:

```
secrets.yml
```

Content:

```yaml
db_user: admin
db_password: secret123
```

Encrypt it:

```
ansible-vault encrypt secrets.yml
```

You will be prompted for a password.

After encryption the file becomes unreadable.

---

### 6. Viewing an Encrypted File

To view encrypted content:

```
ansible-vault view secrets.yml
```

You must provide the vault password.

---

### 7. Editing an Encrypted File

Instead of decrypting and editing manually, use:

```
ansible-vault edit secrets.yml
```

This opens a temporary decrypted editor session.

When saved, Ansible **automatically re-encrypts the file**.

---

### 8. Decrypting a File

If necessary, decrypt a file permanently:

```
ansible-vault decrypt secrets.yml
```

Warning: this exposes secrets.

Usually not recommended.

---

### 9. Using Vault in Playbooks

Example encrypted variable file:

```
vars/secrets.yml
```

Playbook:

```yaml
- name: Deploy application
  hosts: webservers

  vars_files:
    - vars/secrets.yml

  tasks:
    - name: Print database user
      debug:
        msg: "{{ db_user }}"
```

Run playbook:

```
ansible-playbook playbook.yml --ask-vault-pass
```

Ansible will prompt for the vault password.

---

### 10. Using Vault Password File

Typing password every time is inconvenient.

Instead use a password file.

Example file:

```
vault_pass.txt
```

Content:

```
mypassword
```

Run playbook:

```
ansible-playbook playbook.yml --vault-password-file vault_pass.txt
```

Important:

Never commit this password file to Git.

---

### 11. Encrypting Only Variables

Instead of encrypting the entire file, we can encrypt specific variables.

Command:

```
ansible-vault encrypt_string 'secret123' --name 'db_password'
```

Output:

```yaml
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          633137373032343438333765...
```

Now the file remains readable except for the secret.

---

### 12. Using Encrypted Variables

Example variable file:

```yaml
db_user: admin

db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          623761343833...
```

Playbooks can still use:

```
{{ db_password }}
```

Ansible decrypts automatically during execution.

---

### 13. Rekeying Vault Files

If vault password needs to change:

```
ansible-vault rekey secrets.yml
```

You will be asked:

- old password
- new password

The file is re-encrypted with the new password.

---

### 14. Using Multiple Vault IDs

Large organizations often have **multiple secret environments**.

Examples:

```
dev
staging
production
```

Vault IDs allow multiple vault passwords.

Example:

```
ansible-playbook playbook.yml \
--vault-id dev@prompt \
--vault-id prod@prompt
```

Each vault can have a separate password.

---

### 15. Real World Project Structure

Example production project:

```
ansible-project/
│
├── inventory/
│
├── playbooks/
│   └── deploy.yml
│
├── group_vars/
│   ├── dev/
│   │   └── vault.yml
│   └── prod/
│       └── vault.yml
│
└── roles/
```

Each environment contains its own encrypted secrets.

---

### 16. Real World Example

Example encrypted file:

```
group_vars/prod/vault.yml
```

Content:

```yaml
db_user: prod_admin
db_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          323734626432...
```

Playbook uses it automatically.

---

### 17. Best Practices for Vault

Follow these rules in production environments.

1. Never store secrets in plain text  
2. Encrypt all sensitive variable files  
3. Use vault IDs for multiple environments  
4. Never commit vault password files to Git  
5. Use environment-based vault files  
6. Rotate vault passwords periodically  

---

### 18. Common Vault Mistakes

#### Committing password files

Never commit:

```
vault_pass.txt
```

to version control.

---

#### Encrypting entire playbooks

Encrypt only **secret files**, not playbooks.

---

#### Sharing vault password insecurely

Vault passwords should be shared through:

- secure password manager
- secret management system

---

### 19. Vault vs Secret Managers

Ansible Vault is good for:

- small teams
- simple projects

Large organizations often use external secret managers.

Examples:

| Tool | Purpose |
|---|---|
HashiCorp Vault | enterprise secret management |
AWS Secrets Manager | cloud secret storage |
Azure Key Vault | Azure secrets |
Google Secret Manager | GCP secrets |

Ansible can integrate with these tools.
