# Ansible Day 1 – Introduction, Architecture and Practical Setup

## 1. Introduction to Ansible

Ansible is an open-source automation and configuration management tool used to manage servers, configure systems, deploy applications and automate repetitive IT operations.

In DevOps environments, managing multiple servers manually can become difficult. For example, consider an environment containing:

- 5 development servers
- 5 testing servers
- 10 production servers

If we need to install Git, Maven, Apache or other packages on every server, performing the same operations manually takes time and increases the possibility of human errors.

Ansible helps automate these activities from a central machine.

### 1.1 What is Configuration Management?

Configuration management is the process of maintaining and managing the desired configuration of systems.

It includes:

- Installing and updating software packages.
- Managing users and groups.
- Configuring files and directories.
- Managing services.
- Maintaining system settings.
- Applying consistent configurations across multiple servers.

**Example:**

Suppose an organization has ten Linux servers. All ten servers need Git and Apache installed.

Without configuration management, an administrator must connect to every server and execute the installation commands individually.

With Ansible, the administrator can execute a single command against all ten servers.

```bash
ansible all -m ansible.builtin.package -a "name=git state=present"
```

This approach reduces repetitive manual work and helps maintain consistent configurations.

### 1.2 History of Ansible

Ansible was created by Michael DeHaan and initially released in 2012.

Red Hat acquired Ansible, Inc. in 2015.

Ansible is now part of Red Hat's automation ecosystem.

### 1.3 Important Features of Ansible

**Agentless Architecture**

Ansible normally does not require a separate agent installation on managed Linux servers. It uses SSH to communicate with them.

**Simple Configuration**

Ansible uses YAML for writing playbooks. YAML is designed to represent structured data in a human-readable format.

**Python-Based**

Ansible is implemented primarily in Python. Python is important for the control node and for executing most standard modules on managed Linux nodes.

**Automation**

Ansible can automate repetitive operations such as package installation, service management, file creation and application deployment.

**Idempotency**

Many Ansible modules are designed to maintain a desired state.

For example, if a package is already installed, running a task that ensures the package is present normally does not reinstall it unnecessarily.

**Multi-Platform Management**

Ansible supports managing different Linux distributions and can also manage Windows systems through supported connection methods.

### 1.4 What Problems Does Ansible Solve?

| Without Ansible | With Ansible |
|---|---|
| Manual server connections | Centralized automation |
| Repeated commands | Reusable tasks |
| More human errors | Consistent configuration |
| Difficult multi-server management | Group-based management |
| Manual application configuration | Automated configuration |

---

## 2. Ansible Architecture and Components

Ansible follows a control-node and managed-node architecture.

The control node communicates with managed nodes and executes automation tasks.

### 2.1 Architecture Diagram

```text
                   Ansible Control Node
                     (Main Server)
                           |
                    SSH Communication
                           |
                     Inventory File
                           |
                    Ansible Playbooks
                           |
             +-------------+-------------+
             |             |             |
          Dev Server 1  Dev Server 2  Test Server 1
                                         |
                                      Test Server 2
```

### 2.2 Main Components

**Control Node**

The machine where Ansible is installed and commands are executed.

**Managed Nodes**

The remote servers controlled by Ansible.

**Inventory File**

Contains the IP addresses, hostnames and groups of managed nodes.

**Playbooks**

YAML files that define automation tasks.

**Modules**

Components that perform operations on managed nodes.

Examples:

- Package management
- File management
- User management
- Service management
- Command execution

**Plugins**

Extend Ansible functionality, including connection handling, inventory processing and output processing.

### 2.3 How Ansible Works

The general execution flow is:

1. An administrator executes an Ansible command on the control node.
2. Ansible reads the inventory to identify the target servers.
3. Ansible establishes SSH connections to the managed nodes.
4. Ansible transfers and executes the required module code.
5. Managed nodes perform the requested operation.
6. Ansible collects and displays the results.

---

## 3. Control Node and Managed Nodes

### 3.1 Control Node

The control node is the central machine used to manage remote systems.

Responsibilities:

- Installing Ansible.
- Maintaining inventory files.
- Maintaining playbooks.
- Managing SSH private keys.
- Executing Ansible commands.
- Collecting execution results.

### 3.2 Managed Nodes

Managed nodes are the remote servers controlled by Ansible.

Responsibilities:

- Receiving remote automation tasks.
- Executing modules.
- Returning task results.

Ansible does not normally need to be installed on managed Linux nodes.

Most standard Linux modules require Python on the managed nodes.

### 3.3 Example

Consider the following infrastructure:

| Server | Role |
|---|---|
| Main Server | Ansible Control Node |
| Dev Server 1 | Development |
| Dev Server 2 | Development |
| Test Server 1 | Testing |
| Test Server 2 | Testing |

The main server controls all four managed servers.

---

## 4. SSH Communication

SSH stands for Secure Shell.

It is a network protocol used to securely access remote machines.

Ansible normally uses SSH to communicate with Linux managed nodes.

### 4.1 Why SSH Is Required

Suppose Ansible is installed on one server and needs to install Git on another server.

Ansible must establish a remote connection to execute the installation command.

SSH provides this connection.

### 4.2 SSH Connection Example

```bash
ssh root@192.168.1.10
```

Here:

- `ssh` is the SSH client command.
- `root` is the remote username.
- `192.168.1.10` is the remote server IP address.

### 4.3 SSH Authentication Methods

SSH supports different authentication methods, including:

- Password authentication.
- Public-key authentication.

For automated Ansible communication, SSH key-based authentication is commonly used.

### 4.4 Important SSH Configuration

The SSH server configuration is normally located at:

```text
/etc/ssh/sshd_config
```

For this root-based lab, check:

```text
PermitRootLogin yes
PubkeyAuthentication yes
```

`PermitRootLogin` controls root SSH access.

`PubkeyAuthentication` controls public-key authentication.

Changing these settings does not automatically install a public key. The key must also be configured correctly.

---

## 5. Playbooks and Inventory

Ansible uses inventory files and playbooks to organize automation.

### 5.1 Inventory

An inventory file defines the servers Ansible manages.

It can contain:

- Server IP addresses.
- Hostnames.
- Connection usernames.
- Host groups.
- Connection variables.

Example:

```ini
[dev]
192.168.1.10
192.168.1.11

[test]
192.168.1.20
192.168.1.21
```

Here:

- `[dev]` represents development servers.
- `[test]` represents testing servers.

### 5.2 Playbooks

A playbook is a YAML file that defines one or more plays and tasks.

Playbooks are used to describe automation in a structured and reusable way.

Example:

```yaml
---
- name: Install Git on development servers
  hosts: dev
  tasks:
    - name: Install Git
      ansible.builtin.package:
        name: git
        state: present
```

This playbook ensures Git is installed on all hosts belonging to the `dev` group.

### 5.3 Inventory vs Playbook

| Inventory | Playbook |
|---|---|
| Defines managed servers | Defines automation tasks |
| Contains host groups | Contains plays and tasks |
| Identifies where tasks run | Describes what tasks should happen |

---

# 6. Five-Server AWS Lab Setup

For the practical lab, five AWS EC2 instances are used.

The objective is to configure one Ansible control node and four managed nodes.

### 6.1 Lab Infrastructure

| Instance | Role | Operating system |
|---|---|---|
| Main Server | Ansible Control Node | Amazon Linux or Ubuntu |
| Dev Server 1 | Managed Node | Linux |
| Dev Server 2 | Managed Node | Linux |
| Test Server 1 | Managed Node | Linux |
| Test Server 2 | Managed Node | Linux |

### 6.2 Architecture

```text
                     AWS EC2 Environment

                       Main Server
                    Ansible Control Node
                            |
                      SSH Connections
                            |
             +--------------+--------------+
             |              |              |
         Dev Server 1   Dev Server 2   Test Server 1
                                           |
                                       Test Server 2
```

### 6.3 AWS Instance Preparation

Launch five EC2 instances.

For each instance:

1. Select an appropriate Linux AMI.
2. Select the required instance type.
3. Configure networking and security groups.
4. Select or create an SSH key pair.
5. Launch the instance.
6. Note its private IP address.

### 6.4 Security Group Requirements

The control node must communicate with managed nodes over SSH.

| Instance | Inbound rule | Source |
|---|---|---|
| Main Server | SSH TCP 22 | Trusted administrator IP |
| Managed Nodes | SSH TCP 22 | Main Server security group |

Use private IP addresses when instances communicate inside the same VPC.

### 6.5 Important AWS Networking Concepts

- A security group controls inbound and outbound traffic.
- Private IP addresses are used for communication within private networks.
- Public IP addresses can be used for external access when networking permits.
- SSH requires TCP port 22 to be reachable.
- Route tables and network ACLs must also permit communication.

---

# 7. Hostname Configuration

A hostname identifies a machine on a network.

For the lab, meaningful hostnames help distinguish the control node, development servers and testing servers.

### 7.1 Switch to Root

```bash
sudo -i
```

This opens a root shell.

### 7.2 Configure the Main Server Hostname

```bash
hostnamectl set-hostname ansible-main
```

### 7.3 Configure Development Server Hostnames

On Development Server 1:

```bash
hostnamectl set-hostname dev1
```

On Development Server 2:

```bash
hostnamectl set-hostname dev2
```

### 7.4 Configure Testing Server Hostnames

On Testing Server 1:

```bash
hostnamectl set-hostname test1
```

On Testing Server 2:

```bash
hostnamectl set-hostname test2
```

### 7.5 Verify the Hostname

```bash
hostname
```

Or:

```bash
hostnamectl
```

### 7.6 Why Configure Hostnames?

Meaningful hostnames make it easier to identify servers.

For example, instead of remembering multiple IP addresses, administrators can recognize machines by names such as:

- ansible-main
- dev1
- dev2
- test1
- test2

---

# 8. SSH Key Authentication

SSH key-based authentication allows the control node to connect to managed nodes without repeatedly entering account passwords.

### 8.1 Generate an SSH Key Pair

Execute on the control node:

```bash
ssh-keygen -t ed25519
```

Press Enter to accept the default location.

The command generates two files:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

| File | Purpose |
|---|---|
| id_ed25519 | Private key |
| id_ed25519.pub | Public key |

The private key must remain protected on the control node.

### 8.2 Prepare the Root SSH Directory

On each managed node:

```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
```

### 8.3 Copy the Public Key

If initial root authentication is available:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@<MANAGED_NODE_PRIVATE_IP>
```

Repeat for all managed nodes.

This adds the public key to:

```text
/root/.ssh/authorized_keys
```

### 8.4 Configure Permissions

On managed nodes:

```bash
chmod 600 /root/.ssh/authorized_keys
chown -R root:root /root/.ssh
```

### 8.5 Test SSH Connection

```bash
ssh root@<MANAGED_NODE_PRIVATE_IP>
```

If the connection succeeds using the configured key, SSH key authentication is working.

### 8.6 SSH Configuration for Root

On Ubuntu, edit:

```bash
nano /etc/ssh/sshd_config
```

On Amazon Linux, edit the same configuration file.

Check:

```text
PermitRootLogin yes
PubkeyAuthentication yes
```

Restart the SSH service.

Ubuntu:

```bash
systemctl restart ssh
```

Amazon Linux:

```bash
systemctl restart sshd
```

**Security note:** Root SSH access is used here for the requested lab. In production, a dedicated automation account with controlled privilege escalation is generally preferred.

---

# 9. Ansible Installation

Ansible must be installed on the main/control node.

The installation process differs depending on the Linux distribution.

## 9.1 Ubuntu 22.04, 24.04 and 26.04

### Step 1: Update the system

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Install dependencies

```bash
sudo apt install -y software-properties-common \
    python3 \
    python3-pip \
    python3-venv \
    openssh-client \
    git
```

### Step 3: Add the Ansible PPA

```bash
sudo add-apt-repository --yes --update ppa:ansible/ansible
```

### Step 4: Install Ansible

```bash
sudo apt install -y ansible
```

### Step 5: Verify installation

```bash
ansible --version
python3 --version
```

The same general procedure applies to all three Ubuntu versions, provided the Ansible PPA supports the selected release.

## 9.2 Amazon Linux 2023

Amazon Linux 2023 uses DNF.

The older Amazon Linux Extras method is not available on Amazon Linux 2023.

### Step 1: Update the system

```bash
sudo dnf update -y
```

### Step 2: Install Python and required packages

```bash
sudo dnf install -y python3.13 python3.13-pip \
    git openssh-clients
```

Check Python:

```bash
python3.13 --version
```

If the packages are unavailable:

```bash
dnf search python3
```

Select an available supported Python version and adjust the commands accordingly.

### Step 3: Create a virtual environment

```bash
python3.13 -m venv ~/ansible-venv
```

### Step 4: Activate the environment

```bash
source ~/ansible-venv/bin/activate
```

### Step 5: Upgrade pip

```bash
python -m pip install --upgrade pip
```

### Step 6: Install Ansible

```bash
python -m pip install ansible
```

### Step 7: Verify installation

```bash
ansible --version
python --version
```

Whenever a new terminal session is opened:

```bash
source ~/ansible-venv/bin/activate
```

### 9.3 Installation Comparison

| Item | Ubuntu | Amazon Linux 2023 |
|---|---|---|
| Package manager | APT | DNF |
| Python | python3 | Versioned Python executable |
| Ansible installation | APT and PPA | pip in virtual environment |
| SSH client package | openssh-client | openssh-clients |
| Extras repository | Not applicable | Not available |

---

# 10. Inventory Configuration

Ansible uses an inventory file to identify managed servers.

The default inventory location is:

```text
/etc/ansible/hosts
```

A custom inventory file can also be created and supplied using the `-i` option.

### 10.1 Create the Inventory Directory

For a custom lab:

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
```

### 10.2 Create the Inventory File

```bash
nano inventory.ini
```

### 10.3 Configure Development and Testing Groups

```ini
[dev]
dev1 ansible_host=10.0.1.10 ansible_user=root
dev2 ansible_host=10.0.1.11 ansible_user=root

[test]
test1 ansible_host=10.0.1.20 ansible_user=root
test2 ansible_host=10.0.1.21 ansible_user=root

[all:vars]
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Replace the sample IP addresses with the actual private IP addresses of the EC2 instances.

### 10.4 Understanding Inventory Groups

**[dev]**

Contains development servers.

**[test]**

Contains testing servers.

**ansible_host**

Specifies the actual IP address of a managed server.

**ansible_user=root**

Specifies the SSH user.

**ansible_ssh_private_key_file**

Specifies the private key used for authentication.

### 10.5 Verify Inventory

List all managed nodes:

```bash
ansible all -i inventory.ini --list-hosts
```

List development nodes:

```bash
ansible dev -i inventory.ini --list-hosts
```

List testing nodes:

```bash
ansible test -i inventory.ini --list-hosts
```

---

# 11. Ad Hoc Commands

Ansible ad hoc commands are used to execute individual tasks without creating a playbook.

They are useful for quick operations and initial testing.

### 11.1 Basic Syntax

```bash
ansible <host-pattern> -i <inventory-file> -m <module> -a "<arguments>"
```

Where:

- `ansible`: Executes the Ansible command.
- `host-pattern`: Identifies the target hosts.
- `-i`: Specifies the inventory.
- `-m`: Specifies the module.
- `-a`: Provides module arguments.

### 11.2 Install Git on All Servers

Original command style:

```bash
ansible all -i inventory.ini -a "yum install git -y"
```

Module-based approach:

```bash
ansible all -i inventory.ini -m ansible.builtin.package -a "name=git state=present"
```

### 11.3 Install Maven on Development Servers

```bash
ansible dev -i inventory.ini -m ansible.builtin.package -a "name=maven state=present"
```

### 11.4 Create a File on All Servers

```bash
ansible all -i inventory.ini -m ansible.builtin.file -a "path=/tmp/file1 state=touch"
```

### 11.5 List Files

```bash
ansible all -i inventory.ini -m ansible.builtin.command -a "ls /tmp"
```

### 11.6 Install Apache

```bash
ansible all -i inventory.ini -m ansible.builtin.package -a "name=httpd state=present"
```

**Note:** The package name differs across distributions. On Ubuntu, Apache is normally packaged as `apache2`; on Amazon Linux, it is `httpd`.

For a mixed Ubuntu and Amazon Linux inventory, target the appropriate groups separately.

Ubuntu:

```bash
ansible ubuntu -i inventory.ini -m ansible.builtin.package -a "name=apache2 state=present"
```

Amazon Linux:

```bash
ansible amazon -i inventory.ini -m ansible.builtin.package -a "name=httpd state=present"
```

### 11.7 Start Apache on Amazon Linux

```bash
ansible amazon -i inventory.ini -m ansible.builtin.service -a "name=httpd state=started enabled=true"
```

For Ubuntu:

```bash
ansible ubuntu -i inventory.ini -m ansible.builtin.service -a "name=apache2 state=started enabled=true"
```

### 11.8 Create a User

```bash
ansible all -i inventory.ini -m ansible.builtin.user -a "name=raham state=present"
```

### 11.9 Check User Information

```bash
ansible all -i inventory.ini -m ansible.builtin.command -a "cat /etc/passwd"
```

Because the lab uses root SSH access, the commands execute with root permissions.

**Important:** The original commands using `yum`, `httpd` and `systemctl` reflect the Amazon Linux environment. The module-based commands are more portable, but package names can still differ between operating systems.

---

# 12. Host Selection and Inventory Patterns

Ansible supports host patterns to select individual servers or groups of servers.

Host patterns allow administrators to control where automation tasks execute.

### 12.1 Select All Hosts

```bash
ansible all -i inventory.ini --list-hosts
```

Selects all hosts in the inventory.

### 12.2 Select Development Hosts

```bash
ansible dev -i inventory.ini --list-hosts
```

Selects only development servers.

### 12.3 Select Testing Hosts

```bash
ansible test -i inventory.ini --list-hosts
```

Selects only testing servers.

### 12.4 Select the First Host

```bash
ansible "test[0]" -i inventory.ini --list-hosts
```

Selects the first host in the `test` group.

Host indices are zero-based.

### 12.5 Select the Last Host

```bash
ansible "all[-1]" -i inventory.ini --list-hosts
```

Selects the last host in the inventory ordering.

### 12.6 Important Host Patterns

| Pattern | Meaning |
|---|---|
| all | All hosts |
| dev | Hosts in the dev group |
| test | Hosts in the test group |
| test[0] | First host in the test group |
| all[-1] | Last host in inventory ordering |

These are inventory host-selection patterns, not IP address range definitions.

---

# 13. Bash History Timestamps

Bash maintains a history of previously executed commands.

By default, the history output may not display the date and time at which commands were executed.

The `HISTTIMEFORMAT` environment variable can be used to display timestamps.

### 13.1 Enable Timestamps

```bash
export HISTTIMEFORMAT="%F %T "
```

### 13.2 Display Command History

```bash
history
```

Example:

```text
1  2026-11-15 10:20:15 sudo apt update
2  2026-11-15 10:25:30 ansible --version
3  2026-11-15 10:30:10 ssh root@10.0.1.10
```

The exact dates and times depend on the command execution history.

### 13.3 Understand the Format

| Format | Meaning |
|---|---|
| `%F` | Date in YYYY-MM-DD format |
| `%T` | Time in HH:MM:SS format |

### 13.4 Make Timestamp Configuration Permanent

To keep timestamps enabled in future Bash sessions, add the setting to `~/.bashrc`.

```bash
echo 'export HISTTIMEFORMAT="%F %T "' >> ~/.bashrc
source ~/.bashrc
```

This is a Bash shell feature and is not specific to Ansible.

---

# 14. Troubleshooting

| Problem | Possible reason | Solution |
|---|---|---|
| Ansible command not found | Ansible is not installed or virtual environment is inactive | Verify installation and activate the environment |
| Permission denied (publickey) | Incorrect key or SSH configuration | Check private key, public key and root SSH access |
| UNREACHABLE | Network or SSH connectivity problem | Check private IP, security groups and routes |
| Connection refused | SSH service is not running | Check and restart SSH |
| Python interpreter not found | Python is missing on managed node | Install a supported Python version |
| Root login rejected | Root login policy blocks access | Inspect effective SSH configuration |
| PPA installation error | Repository does not support selected Ubuntu release | Verify repository support |
| Package not found | Package name differs by distribution | Use the correct package name for the operating system |

### Useful Troubleshooting Commands

Check SSH status on Ubuntu:

```bash
sudo systemctl status ssh
```

Check SSH status on Amazon Linux:

```bash
sudo systemctl status sshd
```

Validate SSH configuration:

```bash
sudo sshd -t
```

Check effective SSH settings:

```bash
sudo sshd -T | grep -E 'permitrootlogin|pubkeyauthentication'
```

Test SSH with debugging:

```bash
ssh -vvv root@<MANAGED_NODE_PRIVATE_IP>
```

Check Ansible inventory:

```bash
ansible-inventory -i inventory.ini --list
```

---

# 15. Important Learning Points

1. Ansible is an automation and configuration management tool.
2. Ansible was created by Michael DeHaan.
3. Ansible follows a control-node and managed-node architecture.
4. SSH is the normal connection method for Linux managed nodes.
5. Ansible is agentless for its standard SSH-based Linux management.
6. Inventory files define managed hosts and groups.
7. Playbooks are YAML files containing automation tasks.
8. Modules perform operations on managed nodes.
9. Python is required for the Ansible runtime and most standard Linux modules.
10. SSH key authentication supports automated remote connections.
11. Ad hoc commands are useful for executing individual tasks.
12. Host patterns allow selective execution against inventory groups and hosts.
13. Ubuntu uses APT, while Amazon Linux 2023 uses DNF.
14. Package names may differ between Linux distributions.
15. Root SSH access is possible for a lab, but a restricted automation account is preferable in production.

---

# 16. Interview Questions

### 1. What is Ansible?

Ansible is an open-source automation and configuration management tool used to manage servers, deploy applications and automate repetitive IT tasks.

### 2. What is configuration management?

Configuration management is the process of maintaining and controlling the desired configuration of systems, including packages, users, services and files.

### 3. What is a control node?

A control node is the machine where Ansible is installed and automation commands are executed.

### 4. What are managed nodes?

Managed nodes are the remote machines controlled by Ansible.

### 5. Does Ansible require an agent on managed Linux nodes?

No. Standard SSH-based Linux management is agentless, although managed nodes normally need Python for most standard modules.

### 6. Why does Ansible use SSH?

SSH provides secure remote communication and allows Ansible to execute tasks on managed nodes.

### 7. What is an inventory file?

An inventory file contains information about managed servers, including their IP addresses, groups and connection variables.

### 8. What is a playbook?

A playbook is a YAML file that defines plays and tasks for automating operations on managed nodes.

### 9. What is an ad hoc command?

An ad hoc command executes an individual task without requiring a playbook.

### 10. What is the purpose of Ansible modules?

Modules perform specific operations such as package installation, file management, user creation and service management.

### 11. What is the difference between inventory and playbook?

Inventory identifies the target servers, whereas playbooks define the tasks to execute on those servers.

### 12. Why is Python required?

Ansible uses Python on the control node, and most standard modules require a compatible Python interpreter on managed Linux nodes.

### 13. What is SSH key-based authentication?

It is an authentication method that uses a public/private key pair to establish secure SSH connections.

### 14. What is the purpose of `ansible all --list-hosts`?

It displays the hosts selected by the `all` inventory pattern.

### 15. What is the difference between Ubuntu and Amazon Linux package management?

Ubuntu uses APT, while Amazon Linux 2023 uses DNF.

---

# 17. Practical Lab Completion Sequence

The complete practical workflow can be summarized as follows:

```text
Introduction to Ansible
          |
Configuration Management
          |
Ansible Architecture
          |
Control and Managed Nodes
          |
SSH Communication
          |
AWS EC2 Lab Preparation
          |
Hostname Configuration
          |
Root SSH Configuration
          |
SSH Key Authentication
          |
Ansible Installation
          |
Inventory Configuration
          |
Ansible Ad Hoc Commands
          |
Host Selection Patterns
          |
Bash History Timestamps
          |
Connectivity Verification
```

## Final Practical Verification

Before considering the basic lab setup complete, execute:

```bash
# Check Ansible version
ansible --version

# List all managed hosts
ansible all -i inventory.ini --list-hosts

# Test development servers
ansible dev -i inventory.ini -m ansible.builtin.ping

# Test testing servers
ansible test -i inventory.ini -m ansible.builtin.ping

# Test all managed servers
ansible all -i inventory.ini -m ansible.builtin.ping

# Check remote hostnames
ansible all -i inventory.ini -m ansible.builtin.command -a "hostname"

# Check remote users
ansible all -i inventory.ini -m ansible.builtin.command -a "whoami"
```

A successful response from every managed node confirms that the basic Ansible connection and execution workflow is functioning.

---

## Security Considerations

This learning lab uses root SSH access as requested.

For production environments, use a dedicated automation account with SSH key authentication and controlled privilege escalation. Restrict network access and protect private keys. Avoid exposing root SSH access to public networks.

## Technical References

- Ansible documentation: https://docs.ansible.com/
- Ansible installation guide: https://docs.ansible.com/ansible/latest/installation_guide/
- Ansible inventory guide: https://docs.ansible.com/ansible/latest/inventory_guide/
- Ansible SSH connection guide: https://docs.ansible.com/ansible/latest/inventory_guide/connection_details.html
- Ubuntu documentation: https://documentation.ubuntu.com/
- Amazon Linux 2023 documentation: https://docs.aws.amazon.com/linux/al2023/ug/
