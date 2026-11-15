# Ansible – Introduction & Architecture Components

## 1. Introduction to Ansible

Ansible is an open-source automation tool used for configuration management, application deployment, infrastructure automation, and orchestration.

It helps automate repetitive administrative tasks across multiple servers without manually connecting to each machine.

### Why do we use Ansible?

Imagine an environment with 20 Linux servers. We need to:
- Install packages.
- Create users.
- Configure services.
- Deploy applications.
- Manage permissions.

Performing these tasks manually on every server takes time and can cause configuration inconsistencies.

Ansible allows us to define these operations once and execute them across multiple servers.

### Common Use Cases

- Configuration Management
- Application Deployment
- Infrastructure Provisioning
- Cloud Automation
- Network Automation
- Security and Compliance
- Disaster Recovery
- CI/CD Automation

---

## 2. Key Features of Ansible

### 2.1 Agentless Architecture

Ansible does not normally require an Ansible agent to be installed on managed nodes.

- Linux managed nodes commonly use SSH.
- Windows managed nodes commonly use WinRM or supported PowerShell remoting.
- The control node initiates communication with managed nodes.

**Advantages:**
- No continuously running Ansible agent required.
- Easier deployment and maintenance.
- Reduced agent-management overhead.

Note: Many Linux modules require Python on the managed node.

### 2.2 Idempotency

Idempotency means executing the same automation multiple times should maintain the desired state without making unnecessary changes.

Example:

```yaml
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
```

- First execution: Installs Nginx.
- Second execution: Detects that Nginx is already installed.
- Further executions: No unnecessary installation.

This helps maintain consistent configurations.

### 2.3 Declarative Approach

Ansible describes what the final configuration should look like rather than requiring us to specify every command.

Example:

```yaml
- name: Ensure Apache is installed
  ansible.builtin.apt:
    name: apache2
    state: present
```

We specify the desired state, and Ansible determines whether changes are required.

### 2.4 Extensibility

Ansible supports:
- Built-in modules.
- Custom modules.
- Plugins.
- Collections.

These allow Ansible to support different infrastructure environments and automation requirements.

---

## 3. Ansible Architecture

Ansible architecture consists of three primary components:

1. Control Node
2. Inventory
3. Managed Nodes

### Architecture Diagram

```text
                  CONTROL NODE
                (Ansible Installed)
                         |
                 Inventory + Playbook
                         |
                  Connection Method
                    (SSH / WinRM)
                         |
             +-----------+-----------+
             |           |           |
          Node 1       Node 2      Node 3
          Web          App         Database
          Server       Server      Server
```

### 3.1 Control Node

The Control Node is the machine where Ansible is installed and from which automation commands are executed.

Examples:
- Linux workstation.
- Linux server.
- AWS EC2 instance.
- Azure Virtual Machine.
- Windows machine using WSL.

Common commands:

```bash
ansible --version

ansible-inventory --list

ansible all -m ansible.builtin.ping

ansible-playbook -i inventory.ini site.yml
```

### 3.2 Managed Nodes

Managed Nodes are the remote machines that Ansible manages.

Examples:
- AWS EC2 instances.
- Azure Virtual Machines.
- Linux servers.
- Windows servers.
- Network devices.

Important points:

- Ansible is normally not installed on managed nodes.
- Linux nodes commonly use SSH.
- Remote authentication must be configured.
- Python is required for many Linux/POSIX modules.

### 3.3 Inventory

Inventory contains information about the managed nodes.

It tells Ansible:
- Which machines are available.
- Which machines belong to specific groups.
- How to connect to those machines.
- Which variables apply to hosts or groups.

There are two main inventory approaches.

**A. Static Inventory**

A manually maintained list of managed nodes.

Example: `inventory.ini`

```ini
[webservers]
web1 ansible_host=192.0.2.10
web2 ansible_host=192.0.2.11

[appservers]
app1 ansible_host=192.0.2.20
```

Explanation:

- `[webservers]` – Group name.
- `web1`, `web2` – Inventory hostnames.
- `ansible_host` – Actual IP address.

**B. Dynamic Inventory**

Dynamic inventory automatically retrieves host information from external sources.

Example:
- AWS EC2 instances.
- Azure Virtual Machines.
- Other supported infrastructure platforms.

Dynamic inventory is useful when infrastructure changes frequently.

Ansible supports inventory plugins, which are generally preferred when a suitable plugin is available.

Verify inventory:

```bash
ansible-inventory -i inventory.ini --list
```

List hosts:

```bash
ansible webservers -i inventory.ini --list-hosts
```

---

## 4. Ansible Modules

Modules are individual units of work used by Ansible to perform specific operations.

Examples:

| Module | Purpose |
|---|---|
| `ansible.builtin.apt` | Install packages on Ubuntu/Debian |
| `ansible.builtin.dnf` | Manage packages using DNF |
| `ansible.builtin.service` | Manage services |
| `ansible.builtin.copy` | Copy files |
| `ansible.builtin.user` | Manage users |
| `ansible.builtin.file` | Manage files and directories |
| `ansible.builtin.ping` | Test Ansible connectivity |

### Types of Modules

**Built-in Modules**

Modules provided by Ansible's built-in collection.

**Collection Modules**

Modules provided by additional collections.

**Custom Modules**

Modules developed for specific organizational requirements.

### Example

```yaml
- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present
```

Here:
- `apt` is the module.
- `name` specifies the package.
- `state: present` ensures the package is installed.

---

## 5. Ansible Playbooks

A Playbook is a YAML file containing automation instructions.

It defines:
- Which hosts to target.
- Which tasks to execute.
- Which modules to use.
- What configuration should be achieved.

### Playbook Structure

```text
Playbook
   |
   +---- Play
           |
           +---- Tasks
                   |
                   +---- Module
                   |
                   +---- Module
                   |
                   +---- Module
```

### Example

Create `webserver.yml`:

```yaml
---
- name: Configure Web Server
  hosts: webservers
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.apt:
        name: apache2
        state: present

    - name: Start Apache
      ansible.builtin.service:
        name: apache2
        state: started
        enabled: true
```

Execute:

```bash
ansible-playbook -i inventory.ini webserver.yml
```

Explanation:

| Keyword | Meaning |
|---|---|
| `name` | Description of the play or task |
| `hosts` | Target managed nodes |
| `become` | Privilege escalation |
| `tasks` | List of automation operations |
| `apt` | Package management module |
| `service` | Service management module |
| `state: present` | Ensure package is installed |
| `state: started` | Ensure service is running |
| `enabled: true` | Enable service at boot |

---

## 6. Ansible Plugins

Plugins extend or customize Ansible functionality.

Common plugin types:

| Plugin | Purpose |
|---|---|
| Connection Plugin | Establish connections to managed nodes |
| Callback Plugin | Customize execution output |
| Inventory Plugin | Load inventory information |
| Lookup Plugin | Retrieve external or local data |
| Filter Plugin | Transform data |
| Become Plugin | Support privilege escalation |

### Difference Between Modules and Plugins

| Modules | Plugins |
|---|---|
| Perform automation tasks | Extend Ansible functionality |
| Install packages | Customize connections |
| Manage services | Customize output |
| Copy files | Retrieve and transform data |

---

## 7. How Ansible Works

Ansible generally follows this execution process:

**Step 1: Inventory Preparation**

Define the managed nodes and organize them into groups.

**Step 2: Playbook Creation**

Write automation instructions in YAML.

**Step 3: Execute Playbook**

Run the playbook from the control node.

**Step 4: Host Selection**

Ansible identifies the target hosts using the inventory and `hosts` pattern.

**Step 5: Establish Connection**

Ansible connects to the managed nodes using SSH or another supported connection method.

**Step 6: Gather Facts**

Ansible collects system information, such as:
- Operating system.
- IP addresses.
- Hostname.
- Memory.
- Processor information.

**Step 7: Execute Modules**

Ansible executes the required modules to perform automation tasks.

**Step 8: Return Results**

Ansible displays execution results.

Common statuses:

| Status | Meaning |
|---|---|
| `ok` | No changes required |
| `changed` | System configuration changed |
| `failed` | Task failed |
| `unreachable` | Cannot communicate with host |
| `skipped` | Task skipped |

---

## 8. Practical Example – Apache Installation

Let's automate Apache installation on Ubuntu managed nodes.

### Step 1: Create Inventory

`inventory.ini`

```ini
[webservers]
web1 ansible_host=192.0.2.10 ansible_user=ansible
web2 ansible_host=192.0.2.11 ansible_user=ansible
```

### Step 2: Test Connectivity

```bash
ansible webservers -i inventory.ini -m ansible.builtin.ping
```

Successful response:

```json
{
    "changed": false,
    "ping": "pong"
}
```

The Ansible ping module checks Ansible connectivity and module execution. It is not the same as the operating system's ICMP ping command.

### Step 3: Create Playbook

`apache.yml`

```yaml
---
- name: Install and configure Apache
  hosts: webservers
  become: true
  gather_facts: true

  tasks:

    - name: Install Apache
      ansible.builtin.apt:
        name: apache2
        state: present
        update_cache: true

    - name: Start Apache
      ansible.builtin.service:
        name: apache2
        state: started
        enabled: true

    - name: Create custom webpage
      ansible.builtin.copy:
        dest: /var/www/html/index.html
        content: |
          <h1>Welcome to Ansible Automation!</h1>
        owner: root
        group: root
        mode: "0644"
```

### Step 4: Validate Syntax

```bash
ansible-playbook -i inventory.ini apache.yml --syntax-check
```

### Step 5: Preview Changes

```bash
ansible-playbook -i inventory.ini apache.yml --check --diff
```

### Step 6: Execute Playbook

```bash
ansible-playbook -i inventory.ini apache.yml
```

If sudo requires a password:

```bash
ansible-playbook -i inventory.ini apache.yml --ask-become-pass
```

### Step 7: Verify Apache

Check service:

```bash
sudo systemctl status apache2
```

Check webpage:

```bash
curl http://localhost
```

The webpage can also be accessed remotely if the network, firewall, and cloud security rules permit HTTP traffic.

---

## 9. Ubuntu vs Amazon Linux

The package manager and Apache package name differ between operating systems.

| Configuration | Ubuntu | Amazon Linux |
|---|---|---|
| Package Manager | APT | DNF (current Amazon Linux releases) |
| Apache Package | apache2 | httpd |
| Service Name | apache2 | httpd |
| Web Root | /var/www/html | /var/www/html |

Amazon Linux example:

```yaml
- name: Install Apache
  ansible.builtin.dnf:
    name: httpd
    state: present

- name: Start Apache
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: true
```

Always identify the operating system before choosing the appropriate package-management module.

---

## 10. Ansible in DevOps and Multi-Cloud

Ansible can be integrated into a CI/CD workflow.

Example workflow:

```text
Developer
    |
    v
Git Repository
    |
    v
Jenkins Pipeline
    |
    v
Build and Test
    |
    v
Ansible Automation
    |
    v
Deployment Servers
    |
    v
Application Running
```

### Real-world scenario

Suppose an organization operates applications on AWS and Azure.

Ansible can help:
- Configure Linux virtual machines.
- Install application dependencies.
- Deploy application artifacts.
- Configure Nginx or Apache.
- Manage services.
- Maintain consistent configurations.

This makes Ansible useful in multi-cloud environments.

---

## 11. Troubleshooting

| Problem | Possible Solution |
|---|---|
| SSH connection failure | Verify SSH connectivity, username, and key |
| Permission denied | Check SSH authentication and sudo permissions |
| Python interpreter error | Install Python or configure the interpreter path |
| Package installation failure | Verify package manager and package name |
| YAML syntax error | Check indentation and formatting |
| Host unreachable | Check IP address, routing, and firewall |
| Apache not accessible | Check service, port 80, firewall, and security group |

Useful commands:

```bash
ansible --version

ansible-inventory -i inventory.ini --list

ansible webservers -i inventory.ini --list-hosts

ansible webservers -i inventory.ini -m ansible.builtin.ping

ansible-playbook -i inventory.ini apache.yml --syntax-check
```

---

## 12. Interview Questions

**1. What is Ansible?**

Ansible is an open-source automation tool used for configuration management, application deployment, orchestration, and infrastructure automation.

**2. What is an agentless architecture?**

Ansible generally does not require a continuously running agent on managed nodes. It commonly uses SSH to communicate with Linux systems.

**3. What is a Control Node?**

The machine where Ansible is installed and automation is executed.

**4. What are Managed Nodes?**

The remote systems controlled by Ansible.

**5. What is Inventory?**

Inventory defines the managed hosts and groups that Ansible can target.

**6. What is the difference between static and dynamic inventory?**

Static inventory is manually maintained, while dynamic inventory retrieves host information from external sources.

**7. What is a Module?**

A module is a unit of work used to perform an automation operation.

**8. What is a Playbook?**

A YAML file that defines automation plays and tasks.

**9. What is Idempotency?**

Repeated execution maintains the desired state without unnecessary changes.

**10. What is the difference between Modules and Plugins?**

Modules perform automation tasks, whereas plugins extend or customize Ansible functionality.

**11. What is privilege escalation?**

Privilege escalation allows Ansible to execute tasks with elevated permissions, commonly using sudo through `become: true`.

**12. Does Ansible require Python on managed nodes?**

Many POSIX modules require Python. Requirements vary by platform and module.

---

## 13. Technical Verification – Original Concepts vs Current Practices

| Concept | Current understanding | Classification |
|---|---|---|
| Agentless architecture | Still valid; managed-node prerequisites remain | Still Valid |
| Control Node | Supported Linux/UNIX-like environment; Windows users can use WSL | Clarification |
| Inventory location | `/etc/ansible/hosts` is a common default, not mandatory | Clarification |
| Dynamic inventory scripts | Inventory plugins are supported and preferred when available | Alternative Approach |
| Module naming | Prefer FQCN such as `ansible.builtin.apt` | Alternative Approach |
| Module execution | Execution details depend on the module and connection mechanism | Clarification |
| Fact gathering | Enabled by default in normal plays, unless disabled | Still Valid |
| Boolean values | Prefer `true` and `false` in YAML examples | Alternative Approach |
| Apache installation | Ubuntu uses `apache2`; Amazon Linux uses `httpd` | Clarification |

### Important Technical Notes

- Ansible's normal push-based architecture is agentless.
- Dynamic inventory is not restricted to scripts.
- FQCNs make module references explicit.
- Check mode does not guarantee accurate previews for every module.
- Ansible can manage infrastructure after provisioning; it can also provision supported cloud resources through collections and APIs.

---

## 14. Official Documentation

- [Ansible Getting Started](https://docs.ansible.com/projects/ansible/latest/getting_started/index.html)
- [Ansible Basic Concepts](https://docs.ansible.com/projects/ansible/latest/getting_started/basic_concepts.html)
- [Ansible Installation Guide](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html)
- [Ansible Inventory](https://docs.ansible.com/projects/ansible/latest/getting_started/get_started_inventory.html)
- [Ansible Playbooks](https://docs.ansible.com/projects/ansible/latest/getting_started/get_started_playbook.html)
- [Ansible APT Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/apt_module.html)
- [Ansible Service Module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/service_module.html)

---

## 15. Key Takeaways

1. Ansible automates repetitive infrastructure tasks.
2. Control Node runs Ansible; Managed Nodes are the target systems.
3. Inventory defines the managed infrastructure.
4. Modules perform individual automation operations.
5. Playbooks define automation workflows using YAML.
6. Plugins extend Ansible functionality.
7. Idempotency helps maintain consistent configurations.
8. Ansible supports cloud automation and CI/CD integration.
9. Platform differences must be considered when writing automation.
10. Ansible is useful for managing infrastructure across AWS, Azure, and on-premises environments.

**Learning perspective:** Understanding Ansible architecture is the foundation for writing playbooks, managing multiple servers, and integrating infrastructure automation into DevOps workflows.
