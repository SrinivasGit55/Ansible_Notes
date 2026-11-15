# Introduction to Ansible

## What is Ansible?

Ansible is an open-source IT automation tool used for:

- Configuration Management
- Application Deployment
- Infrastructure Provisioning
- Orchestration
- Continuous Delivery

It helps system administrators and DevOps engineers automate repetitive tasks across multiple servers efficiently and consistently.

One of the key advantages of Ansible is its **agentless architecture**, meaning no additional software needs to be installed on the managed nodes.

---

## Why Ansible?

Managing servers manually becomes difficult as the infrastructure grows.

### Challenges of Manual Administration

- Time-consuming operations
- Human errors
- Inconsistent configurations
- Difficult maintenance
- Reduced scalability

### How Ansible Helps

Ansible automates these tasks using simple YAML-based playbooks and SSH connectivity.

### Example

Instead of manually installing Apache on 100 servers, Ansible can automate the installation on all servers with a single playbook execution.

---

# Ansible Architecture

Ansible follows a simple agentless architecture.

```text
+------------------+
|  Control Node    |
| (Ansible Server) |
+---------+--------+
          |
          | SSH
          |
  -------------------
  |       |         |
  v       v         v
Server1 Server2 Server3
Managed Nodes
```

---

## Components of Ansible Architecture

### 1. Control Node

The machine where Ansible is installed and executed.

#### Responsibilities

- Executes Ansible commands
- Runs playbooks
- Connects to managed nodes
- Pushes configurations and updates

#### Example

```bash
ansible all -m ping
```

---

### 2. Managed Nodes

Managed nodes are the target systems controlled by Ansible.

Examples:

- Linux Servers
- Web Servers
- Database Servers
- Cloud Instances
- Virtual Machines

#### Requirements

- SSH access
- Python installed (for most modules)

> Note: Ansible does not need to be installed on managed nodes.

---

### 3. Inventory

The Inventory file contains information about target hosts that Ansible manages.

#### Static Inventory Example

```ini
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

#### Dynamic Inventory

Dynamic inventory automatically discovers resources from cloud platforms such as:

- AWS
- Azure
- Google Cloud Platform (GCP)

---

### 4. Modules

Modules are reusable programs that perform specific tasks on managed nodes.

#### Common Modules

| Module | Purpose |
|----------|----------|
| ping | Check connectivity |
| copy | Copy files |
| file | Manage files and directories |
| yum | Install packages on RHEL/CentOS |
| apt | Install packages on Ubuntu/Debian |
| service | Manage services |

#### Example

```bash
ansible all -m ping
```

```bash
ansible webservers -m service -a "name=httpd state=started"
```

---

### 5. Playbooks

Playbooks are YAML files that describe automation workflows and tasks.

#### Example Playbook

```yaml
---
- name: Install Apache Web Server
  hosts: webservers
  become: yes

  tasks:
    - name: Install Apache Package
      yum:
        name: httpd
        state: present

    - name: Start Apache Service
      service:
        name: httpd
        state: started
```

#### Benefits of Playbooks

- Human-readable
- Easy to maintain
- Reusable
- Version-controlled

---

### 6. Plugins

Plugins extend Ansible's functionality.

#### Types of Plugins

##### Connection Plugins

Used to connect with managed nodes.

Examples:

- SSH
- WinRM

##### Callback Plugins

Control and customize command output.

##### Lookup Plugins

Retrieve data from external sources.

---

### 7. Facts

Facts are system-related information automatically gathered from managed nodes.

#### Collect Facts

```bash
ansible all -m setup
```

#### Examples of Facts

- Hostname
- IP Address
- Operating System
- CPU Information
- Memory Information

#### Example Variable

```yaml
{{ ansible_hostname }}
```

---

### 8. Roles

Roles provide a standardized directory structure for organizing playbooks and configurations.

#### Role Structure

```text
my_role/
├── tasks/
├── handlers/
├── templates/
├── files/
├── vars/
├── defaults/
└── meta/
```

#### Advantages

- Reusability
- Better organization
- Easier maintenance
- Improved scalability

---

## YAML Basics

Ansible Playbooks are written in YAML.

### Example

```yaml
name: Srinivas
course: DevOps
experience: 1
```

### YAML Rules

- Use spaces instead of tabs
- Maintain proper indentation
- Use key-value pairs separated by a colon (`:`)

---

## Ad-Hoc Commands

Ad-hoc commands are one-line commands used for quick automation tasks.

### Ping All Hosts

```bash
ansible all -m ping
```

### Check Disk Space

```bash
ansible all -a "df -h"
```

### Restart a Service

```bash
ansible webservers -m service -a "name=httpd state=restarted"
```

---

## Ansible Workflow

### Step 1: Create an Inventory

```ini
[webservers]
192.168.1.10
```

### Step 2: Create a Playbook

```yaml
---
- hosts: webservers
  tasks:
    - name: Install Apache
      yum:
        name: httpd
        state: present
```

### Step 3: Execute the Playbook

```bash
ansible-playbook apache.yml
```

### Step 4: Verify Configuration

Ansible connects through SSH and applies the required configuration to the target servers.

---

# Advantages of Ansible

✅ Agentless Architecture

✅ Easy-to-Learn YAML Syntax

✅ Open Source

✅ Fast Deployment

✅ Scalable Automation

✅ Secure Communication via SSH

✅ Cloud Integration Support

✅ Reduced Human Errors

✅ Improved Infrastructure Consistency

---

# Interview Questions

## 1. What is Ansible?

Ansible is an open-source automation tool used for configuration management, application deployment, infrastructure provisioning, and orchestration.

---

## 2. Why is Ansible called an agentless tool?

Because it does not require any agent software to be installed on managed nodes. It communicates using SSH or WinRM.

---

## 3. What is an Inventory in Ansible?

An Inventory is a file containing details of hosts and groups of hosts managed by Ansible.

---

## 4. What are Playbooks?

Playbooks are YAML files that define automation tasks and configurations.

---

## 5. Difference Between Modules and Playbooks?

| Modules | Playbooks |
|-----------|-----------|
| Perform a single task | Contain multiple tasks |
| Reusable units of work | Automation workflows |
| Executed directly | Execute multiple modules |

---

## 6. Which protocol does Ansible use to communicate?

- SSH for Linux/Unix systems
- WinRM for Windows systems

---

# Quick Revision

```text
Ansible
│
├── Control Node
├── Managed Nodes
├── Inventory
├── Modules
├── Playbooks
├── Roles
├── Plugins
└── Facts

Communication : SSH
Language      : YAML
Architecture  : Agentless
Purpose       : Automation
```

---

## Conclusion

Ansible is one of the most popular DevOps automation tools due to its simplicity, agentless architecture, and powerful automation capabilities. It enables organizations to manage infrastructure, deploy applications, and maintain consistency across environments efficiently using simple YAML-based playbooks.
