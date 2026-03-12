## Handling SSH Host Key Checking in Ansible — Secure vs Automated Approaches

When automating infrastructure using Ansible, one important SSH behavior can interrupt automation:

**Host Key Checking Prompt**

If not configured properly, Ansible automation may stop and wait for **manual confirmation** when connecting to a server for the first time.

---

### 1. The Problem: SSH Host Verification Prompt

When connecting to a server using SSH for the **first time**, you often see this prompt:

```
The authenticity of host '157.230.29.113' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

You must type:

```
yes
```

Then SSH connects successfully.

Example command:

```
ssh root@157.230.29.113
```

This behavior also affects Ansible because Ansible uses **SSH internally** to connect to servers.

If the prompt appears, automation stops and waits for manual confirmation.

For example, if we automate configuration for **100 servers**, we would have to type:

```
yes
```

100 times.

This breaks automation.

---

### 2. What is Host Key Checking?

**Host Key Checking** is a security feature in SSH.

It protects against:

- **Server spoofing**
- **Man-in-the-middle attacks**

It verifies that the server you are connecting to is actually the correct server.

Without this mechanism, an attacker could impersonate a server and intercept communication.

Because Ansible uses SSH, this feature is **enabled by default in Ansible**.

---

### 3. How SSH Host Authentication Works

To understand host key checking, we must understand how SSH authentication works internally.

When a machine connects to a remote server:

```
Local Machine (Control Node)
        |
        | SSH
        |
Remote Server
```

Two verifications occur:

1. **Server Authentication**
2. **Client Authentication**

---

### 4. Server Authentication

Your local machine must verify:

> "Is this server really the server I expect?"

SSH does this using **host keys**.

When connecting to a server for the first time, SSH asks:

```
Do you trust this server?
```

If you type:

```
yes
```

SSH stores the server's identity inside a file called:

```
~/.ssh/known_hosts
```

This file stores fingerprints of servers that your machine trusts.

---

### 5. The known_hosts File

Location:

```
~/.ssh/known_hosts
```

This file contains entries like:

```
157.230.29.113 ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
```

Next time you connect to that server:

```
ssh root@157.230.29.113
```

SSH checks the stored fingerprint and automatically trusts the server.

No prompt appears again.

---

### 6. Why Host Key Checking Interrupts Ansible Automation

If a server is not present in the `known_hosts` file, SSH asks for confirmation.

Since Ansible executes tasks automatically, it **cannot type "yes"**.

Therefore automation stops.

Example situation:

- 100 new servers
- none exist in `known_hosts`
- automation stops for each one

To solve this, we configure host key checking behavior.

There are **two common approaches**.

---

# Approach 1 — Pre-Register the Server Key (Secure Method)

This is the **secure approach** and recommended for **long-running servers**.

Example:

- production servers
- stable infrastructure
- long-term machines

The idea is simple:

Add the server's SSH key to `known_hosts` **before Ansible connects**.

---

### 7. Adding Server Keys Using ssh-keyscan

We can add host fingerprints using:

```
ssh-keyscan
```

Example:

```
ssh-keyscan -H 161.35.81.154 >> ~/.ssh/known_hosts
```

Explanation:

| Part | Meaning |
|---|---|
| ssh-keyscan | retrieves SSH public host key |
| -H | hashes host entry |
| IP | server address |
| >> | append to file |
| ~/.ssh/known_hosts | trusted hosts list |

After running this command, the server is trusted.

Now SSH will not ask for confirmation.

---

### 8. Verifying the Entry

Check the file:

```
cat ~/.ssh/known_hosts
```

You should see an entry for the server.

Now try connecting:

```
ssh root@161.35.81.154
```

The confirmation prompt will **not appear**.

---

### 9. Allowing the Server to Authenticate You

The server must also trust **your machine**.

SSH authentication typically uses **public/private keys**.

Example:

```
private key → ~/.ssh/id_rsa
public key → ~/.ssh/id_rsa.pub
```

The server must store the **public key**.

Location on server:

```
~/.ssh/authorized_keys
```

---

### 10. Copying Public Key to Server

We can copy the public key using:

```
ssh-copy-id
```

Example:

```
ssh-copy-id root@178.128.241.118
```

Explanation:

| Part | Meaning |
|---|---|
| ssh-copy-id | copies public key |
| root | server user |
| IP | server address |

This command:

1. connects to the server
2. asks for password
3. adds your public key to:

```
~/.ssh/authorized_keys
```

After this, SSH login works without passwords.

---

### 11. Result of Proper SSH Setup

Once both sides trust each other:

```
Control Node
     |
     | SSH (key authentication)
     |
Target Server
```

Connections occur without:

- password
- host key prompt

This is the **ideal secure configuration**.

---

# Approach 2 — Disable Host Key Checking (Less Secure)

This approach disables host verification completely.

It is commonly used in **ephemeral infrastructure**.

---

### 12. What is Ephemeral Infrastructure?

Ephemeral servers are **temporary servers**.

Examples:

- CI/CD environments
- auto-scaling groups
- container hosts
- dynamic cloud instances

Servers may be:

- created automatically
- destroyed quickly

Because servers change frequently, managing `known_hosts` entries becomes impractical.

In such cases, disabling host key checking makes sense.

---

### 13. Disabling Host Key Checking in Ansible

We disable host key checking using the **Ansible configuration file**.

Configuration file name:

```
ansible.cfg
```

---

### 14. Default Ansible Configuration File Locations

Ansible looks for configuration files in several locations.

Common locations include:

```
/etc/ansible/ansible.cfg
~/.ansible.cfg
```

Modern installations may not create these automatically.

So we can create them manually.

---

### 15. Creating the Configuration File

Example:

```
vim ~/.ansible.cfg
```

Add the following configuration:

```
[defaults]
host_key_checking = False
```

Explanation:

| Setting | Meaning |
|---|---|
| defaults | configuration section |
| host_key_checking | SSH verification |
| False | disable checking |

Save the file.

---

### 16. Testing the Configuration

Now run an Ansible command:

```
ansible droplet -i hosts -m ping
```

Since host key checking is disabled:

- no confirmation prompt appears
- automation runs immediately

Ansible reads the configuration file and applies the setting.

---

# How Ansible Finds Configuration Files

Ansible searches for configuration files in a specific order.

The order is extremely important.

```
1. ANSIBLE_CONFIG (environment variable)
2. ansible.cfg (current directory)
3. ~/.ansible.cfg (home directory)
4. /etc/ansible/ansible.cfg
```

Ansible **uses the first file it finds** and ignores the rest.

---

### 17. Configuration Source 1 — Environment Variable

You can explicitly specify the config file.

Example:

```
export ANSIBLE_CONFIG=/opt/project/ansible.cfg
```

Now Ansible will use this file.

Example project:

```
/opt/project/
   ansible.cfg
   hosts
   playbook.yml
```

Running any Ansible command inside this environment will use that config file.

---

### 18. Configuration Source 2 — Current Directory

If a file named `ansible.cfg` exists in the **current working directory**, Ansible uses it.

Example project:

```
project/
   ansible.cfg
   hosts
   deploy.yml
```

Running:

```
ansible-playbook deploy.yml
```

Ansible will automatically use:

```
project/ansible.cfg
```

---

### 19. Configuration Source 3 — Home Directory

Location:

```
~/.ansible.cfg
```

Example:

```
/home/user/.ansible.cfg
```

This configuration applies to **all Ansible commands run by that user**.

Example contents:

```
[defaults]
host_key_checking = False
inventory = ~/ansible/hosts
```

---

### 20. Configuration Source 4 — Global Configuration

Location:

```
/etc/ansible/ansible.cfg
```

This file applies **system-wide**.

Example environment:

```
multiple DevOps engineers share one server
```

Global configuration ensures consistent behavior.

---

### 21. Example Configuration Search Scenario

Suppose these files exist:

```
/etc/ansible/ansible.cfg
~/.ansible.cfg
./ansible.cfg
```

And you run Ansible inside the project directory.

Search order:

```
1. ANSIBLE_CONFIG → not set
2. ./ansible.cfg → found
```

Ansible stops searching and uses:

```
./ansible.cfg
```

The other files are ignored.

---

# Summary

Host Key Checking is a **security mechanism in SSH** that protects against:

- server spoofing
- man-in-the-middle attacks

However, it can interrupt automation.

Two approaches exist:

**Approach 1 — Secure (Recommended for long-lived servers)**

- add server fingerprint using:

```
ssh-keyscan
```

- store entries in:

```
~/.ssh/known_hosts
```

- copy public key using:

```
ssh-copy-id
```

**Approach 2 — Disable Checking (Common for ephemeral infrastructure)**

Create configuration file:

```
~/.ansible.cfg
```

Add:

```
[defaults]
host_key_checking = False
```

Ansible configuration files are discovered in this order:

```
ANSIBLE_CONFIG
ansible.cfg (current directory)
~/.ansible.cfg
/etc/ansible/ansible.cfg
```

Understanding these mechanisms ensures that Ansible automation runs **securely and without interruption**.
