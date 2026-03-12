## Managing AWS EC2 Servers with Ansible — Inventory Using DNS, SSH Keys, and Python Interpreter Configuration

In previous sections we learned:

- how to install Ansible
- how to prepare managed servers
- how to create an inventory file
- how to connect to servers using ad-hoc commands

Now we will extend that setup to **cloud infrastructure**, specifically **AWS EC2 instances**.



---

### 1. Creating EC2 Instances in AWS

First we need servers that Ansible will manage.

In AWS these servers are called **EC2 instances**.

Example setup:

| Instance | Purpose |
|---|---|
| EC2-1 | Web server |
| EC2-2 | Application server |

Steps (high level):

1. Login to AWS console
2. Navigate to **EC2**
3. Click **Launch Instance**
4. Select an OS (for example Amazon Linux or Ubuntu)
5. Create or choose a **key pair**
6. Launch two instances

After launching, AWS provides important information:

- Public IP
- Private IP
- Public DNS

Example:

```
Public DNS:
ec2-18-192-209-223.eu-central-1.compute.amazonaws.com
ec2-18-193-88-102.eu-central-1.compute.amazonaws.com
```

These servers will be managed by Ansible.

---

### 2. IP Address vs DNS Name in Ansible Inventory

In the inventory file we usually specify servers like this:

```
157.230.29.95
157.230.29.113
```

However Ansible also allows using **DNS names**.

Example:

```
ec2-18-192-209-223.eu-central-1.compute.amazonaws.com
```

Both approaches work.

| Method | Example |
|---|---|
| IP Address | 157.230.29.95 |
| DNS Name | ec2-18-192-209-223.eu-central-1.compute.amazonaws.com |

DNS names are often preferred because:

- cloud IPs may change
- DNS names remain consistent

---

### 3. Updating the Ansible Inventory File

Now we update the **hosts inventory file** to include both:

- DigitalOcean droplets
- AWS EC2 instances

Example inventory:

```
[droplet]
157.230.29.95
157.230.29.113

[droplet:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=root

[ec2]
ec2-18-192-209-223.eu-central-1.compute.amazonaws.com
ec2-18-193-88-102.eu-central-1.compute.amazonaws.com

[ec2:vars]
ansible_ssh_private_key_file=~/Downloads/ansible.pem
ansible_user=ec2-user
```

Explanation:

| Section | Purpose |
|---|---|
| droplet | DigitalOcean servers |
| ec2 | AWS servers |
| droplet:vars | variables for droplet group |
| ec2:vars | variables for ec2 group |

Now Ansible knows:

- which servers belong to which cloud platform
- which SSH key to use
- which user to login with

---

### 4. Understanding the EC2 SSH User

Unlike other Linux servers, AWS images have **default usernames**.

Examples:

| OS | Default User |
|---|---|
| Amazon Linux | ec2-user |
| Ubuntu | ubuntu |
| Debian | admin |
| CentOS | centos |

For Amazon Linux:

```
ansible_user=ec2-user
```

This tells Ansible to connect using:

```
ssh ec2-user@server
```

---

### 5. Using the PEM Key for AWS Authentication

When launching an EC2 instance AWS creates a **key pair**.

Example:

```
ansible.pem
```

This file is the **SSH private key** used for authentication.

So we specify it in inventory:

```
ansible_ssh_private_key_file=~/Downloads/ansible.pem
```

Now Ansible will connect like this internally:

```
ssh -i ~/Downloads/ansible.pem ec2-user@server
```

---

### 6. Fixing AWS PEM File Permissions

AWS requires strict permissions on private keys.

If permissions are too open, SSH will refuse to use the key.

Example error:

```
Permissions 0644 for 'ansible.pem' are too open
```

To fix this we restrict the permissions.

Command:

```
chmod 400 /location/to/pemfile
```

Example:

```
chmod 400 ~/Downloads/ansible.pem
```

Permission meaning:

| Permission | Meaning |
|---|---|
| 400 | read-only for owner |

This ensures the key is secure.

---

### 7. Testing Connectivity to EC2 Servers

Now we test if Ansible can connect to the EC2 instances.

Command:

```
ansible ec2 -i hosts -m ping
```

Explanation:

| Component | Meaning |
|---|---|
| ec2 | target EC2 group |
| -i hosts | inventory file |
| -m ping | ping module |

Example output:

```
ec2-18-192-209-223 | SUCCESS
ec2-18-193-88-102 | SUCCESS
```

This confirms:

- SSH connection works
- Python is available
- Ansible modules can run

---

### 8. Understanding the Python Interpreter Warning

Sometimes Ansible shows a warning like this:

```
[WARNING]: Platform linux on host xyz is using the discovered Python interpreter at /usr/bin/python3
```

Why this happens:

Ansible has a **Python interpreter discovery system**.

When connecting to a server, Ansible tries to find:

```
which python should be used
```

Possible interpreters:

```
/usr/bin/python
/usr/bin/python3
/usr/bin/python3.8
/usr/bin/python3.9
```

If multiple interpreters exist, Ansible automatically chooses one.

The warning simply indicates:

> Ansible discovered a Python interpreter automatically.

It is not an error.

But in production environments we usually **define it explicitly**.

---

### 9. Explicitly Defining the Python Interpreter

To avoid interpreter discovery warnings, we specify the interpreter manually.

We do this in the inventory file.

Example:

```
ansible_python_interpreter=/usr/bin/python3.9
```

Note: correct spelling is

```
ansible_python_interpreter
```

(not "interpretor")

---

### 10. Updated Inventory File with Python Interpreter

```
[droplet]
157.230.29.95
157.230.29.113

[droplet:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_user=root

[ec2]
ec2-18-192-209-223.eu-central-1.compute.amazonaws.com
ec2-18-193-88-102.eu-central-1.compute.amazonaws.com

[ec2:vars]
ansible_ssh_private_key_file=~/Downloads/ansible.pem
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python3.9
```

Now Ansible will always use:

```
/usr/bin/python3.9
```

on EC2 machines.

---

### 11. Running the Ping Command Again

Now run:

```
ansible ec2 -i hosts -m ping
```

This time:

- no interpreter warning
- successful connection

Example output:

```
ec2-18-192-209-223 | SUCCESS
ec2-18-193-88-102 | SUCCESS
```

This confirms that:

- SSH authentication works
- PEM key permissions are correct
- Python interpreter is correctly configured

---

### 12. Using Domain Names as Hosts

Inventory does not require IP addresses.

We can also use:

- DNS names
- domain names
- load balancer hostnames

Example:

```
[webservers]
myapp.com
api.myapp.com
```

This is common when managing:

- production web servers
- domain-based infrastructure

Ansible resolves the DNS automatically.

---

### 13. Example Multi-Cloud Inventory

A real infrastructure may contain servers across multiple platforms.

Example:

```
[digitalocean]
157.230.29.95
157.230.29.113

[aws]
ec2-18-192-209-223.eu-central-1.compute.amazonaws.com
ec2-18-193-88-102.eu-central-1.compute.amazonaws.com

[webservers]
web1.myapp.com
web2.myapp.com

[databases]
db1.myapp.com
```

This allows targeting:

- specific cloud providers
- specific roles
- specific environments

---

### 14. Summary

We expanded our Ansible setup to manage **AWS EC2 servers**.

Key concepts covered:

**Inventory can use:**

- IP addresses
- DNS names
- domain names

**AWS authentication requires:**

- PEM private key
- correct permissions

```
chmod 400 key.pem
```

**Default AWS SSH user:**

```
ec2-user
```

**Testing connectivity:**

```
ansible ec2 -i hosts -m ping
```

**Python interpreter warnings** occur because Ansible auto-detects Python.

To avoid them we specify:

```
ansible_python_interpreter=/usr/bin/python3
```

Now Ansible can manage servers across:

- DigitalOcean
- AWS
- any other infrastructure


