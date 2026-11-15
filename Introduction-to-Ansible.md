# Introduction to Ansible

## What is Ansible?

Ansible is an open-source IT automation tool used for:

- Configuration management
- Application deployment
- Infrastructure provisioning
- Orchestration
- Continuous delivery

It helps system administrators and DevOps engineers automate repetitive tasks across multiple servers efficiently and consistently. One of Ansible's key advantages is its **agentless architecture**: no additional agent software needs to be installed on managed nodes.

---

## Why Ansible?

Managing servers manually becomes difficult as infrastructure grows.

### Challenges of manual administration

- Time-consuming operations
- Human errors
- Inconsistent configurations
- Difficult maintenance
- Reduced scalability

Ansible automates these tasks using YAML-based playbooks and SSH connectivity. For example, instead of manually installing Apache on 100 servers, Ansible can install it across all of them with a single playbook execution.

---

## Ansible architecture

Ansible uses an agentless architecture. The control node runs Ansible and connects to managed nodes, typically over SSH.

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

### Components

#### 1. Control node

The machine where Ansible is installed and executed. It runs commands and playbooks, connects to managed nodes, and applies configurations.

```bash
ansible all -m ping
```

#### 2. Managed nodes

Managed nodes are the target systems controlled by Ansible, such as Linux servers, web servers, database servers, cloud instances, and virtual machines.

Typical requirements include SSH access and Python for most modules. Ansible itself does not need to be installed on managed nodes.

#### 3. Inventory

An inventory lists the hosts and groups that Ansible manages.

Static inventory example:

```ini
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

Dynamic inventory can discover resources from cloud platforms such as AWS, Azure, and Google Cloud Platform (GCP).

#### 4. Modules

Modules are reusable programs that perform tasks on managed nodes.

| Module | Purpose |
| --- | --- |
| `ping` | Check connectivity |
| `copy` | Copy files |
| `file` | Manage files and directories |
| `yum` | Install packages on RHEL/CentOS |
| `apt` | Install packages on Ubuntu/Debian |
| `service` | Manage services |

Examples:

```bash
ansible all -m ping
ansible webservers -m service -a "name=httpd state=started"
```

#### 5. Playbooks

Playbooks are YAML files that describe automation workflows and tasks.

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

Playbooks are human-readable, reusable, maintainable, and suitable for version control.

#### 6. Plugins

Plugins extend Ansible's functionality. Examples include:

- **Connection plugins:** connect to managed nodes, for example SSH and WinRM
- **Callback plugins:** customize command output
- **Lookup plugins:** retrieve data from external sources

#### 7. Facts

Facts are system information gathered from managed nodes.

```bash
ansible all -m setup
```

Examples include hostname, IP address, operating system, CPU, and memory information. A fact can be referenced in a playbook as `{{ ansible_hostname }}`.

#### 8. Roles

Roles provide a standardized directory structure for organizing playbooks and configuration.

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

Roles improve reusability, organization, maintainability, and scalability.

---

## YAML basics

Ansible playbooks are written in YAML.

```yaml
name: Srinivas
course: DevOps
experience: 1
```

YAML guidelines:

- Use spaces instead of tabs.
- Maintain correct indentation.
- Separate keys and values with a colon (`:`).

---

## Ad-hoc commands

Ad-hoc commands are one-line commands for quick automation tasks.

```bash
# Ping all hosts
ansible all -m ping

# Check disk space
ansible all -a "df -h"

# Restart a service
ansible webservers -m service -a "name=httpd state=restarted"
```

---

## Ansible workflow

1. Create an inventory:

   ```ini
   [webservers]
   192.168.1.10
   ```

2. Create a playbook:

   ```yaml
   ---
   - hosts: webservers
     tasks:
       - name: Install Apache
         yum:
           name: httpd
           state: present
   ```

3. Execute the playbook:

   ```bash
   ansible-playbook apache.yml
   ```

4. Verify that Ansible connected over SSH and applied the desired configuration.

---

## Advantages of Ansible

- Agentless architecture
- Easy-to-learn YAML syntax
- Open source
- Fast deployment
- Scalable automation
- Secure communication via SSH
- Cloud integration support
- Reduced human error
- Improved infrastructure consistency

---

## Interview questions

### 1. What is Ansible?

Ansible is an open-source automation tool for configuration management, application deployment, infrastructure provisioning, and orchestration.

### 2. Why is Ansible called agentless?

It does not require agent software on managed nodes. It communicates using SSH or, for Windows systems, WinRM.

### 3. What is an inventory in Ansible?

An inventory contains details of the hosts and host groups managed by Ansible.

### 4. What are playbooks?

Playbooks are YAML files that define automation tasks and configurations.

### 5. What is the difference between modules and playbooks?

| Modules | Playbooks |
| --- | --- |
| Perform a task | Organize multiple tasks into a workflow |
| Reusable units of work | YAML automation definitions |
| Can be executed directly | Run tasks and modules in sequence |

### 6. Which protocols does Ansible use to communicate?

- SSH for Linux/Unix systems
- WinRM for Windows systems

---

## Quick revision

```text
Ansible
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

## Conclusion

Ansible is a popular DevOps automation tool because of its simplicity, agentless architecture, and powerful automation capabilities. It helps organizations manage infrastructure, deploy applications, and maintain consistency across environments using YAML-based playbooks.
