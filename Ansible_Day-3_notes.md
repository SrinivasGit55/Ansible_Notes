# Ansible Day 3 – Shell vs Command vs Raw, Variables, Loops and Troubleshooting

## 1. Introduction

Ansible provides different ways to execute commands, manage packages, create users and groups, and inspect managed nodes.

This learning session covers:

- Executing Linux commands using Shell, Command and Raw modules.
- Installing and removing software packages.
- Defining variables inside Playbooks and passing them at runtime.
- Using loops to manage multiple packages, users and groups.
- Troubleshooting SSH connections and checking managed-node availability.
- Collecting CPU and memory information.
- Understanding how changing an AWS EC2 instance type can provide additional computing resources.

The objective is to understand how Ansible automates repetitive Linux administration tasks across multiple managed nodes.

---

## 2. Executing Linux Commands Using Ansible

Ansible provides three modules for executing commands on managed nodes:

1. `ansible.builtin.shell`
2. `ansible.builtin.command`
3. `ansible.builtin.raw`

Although all three can execute commands, they work differently.

### 2.1 Shell Module

The Shell module executes commands through a shell on the managed node.

It is useful when a command requires shell features such as:

- Pipes (`|`)
- Redirection (`>`, `>>`)
- Wildcards (`*`)
- Command chaining (`&&`)
- Environment-variable expansion
- Shell operators

Example:

```yaml
---
- name: Execute commands using Shell
  hosts: all
  become: true

  tasks:
    - name: Install Git
      ansible.builtin.shell: yum install git -y

    - name: Display disk information
      ansible.builtin.shell: df -h | grep -v tmpfs
```

**Scenario:** Suppose I manage several Linux servers and need to filter disk information using `grep`. The pipe operator requires shell processing, so the Shell module is appropriate.

For a straightforward command that does not need shell features, the Command module is generally preferable.

### 2.2 Command Module

The Command module executes a command directly without passing it through a shell. It is suitable for ordinary Linux commands that do not require shell-specific features.

Example:

```yaml
---
- name: Execute commands using Command
  hosts: all
  become: true

  tasks:
    - name: Display current date
      ansible.builtin.command: date

    - name: Display hostname
      ansible.builtin.command: hostname
```

The Command module does not interpret shell operators such as pipes, redirection, wildcards or command chaining.

For example, this is not a shell pipeline:

```yaml
- name: Incorrect use of Command for a pipe
  ansible.builtin.command: df -h | grep tmpfs
```

Use Shell when shell interpretation is required.

### 2.3 Raw Module

The Raw module executes a command directly through the configured remote connection without using Ansible's normal module subsystem.

It is particularly useful when a managed node does not yet have Python installed. Most normal Ansible modules require a usable Python interpreter on the managed node. Raw can help install Python or prepare a machine before using those modules.

Example:

```yaml
---
- name: Prepare a managed node
  hosts: all
  become: true

  tasks:
    - name: Install Python
      ansible.builtin.raw: yum install python3 -y
```

**Scenario:** Imagine that I launch a new Linux server in AWS. The server is accessible through SSH, but it does not have the Python environment required by normal Ansible modules. Raw can execute an initial installation command. Once Python is available, I can use normal Ansible modules.

Raw is a special-purpose option, not a routine replacement for Shell or Command.

### 2.4 Comparison: Shell vs Command vs Raw

| Feature | Shell | Command | Raw |
|---|---|---|---|
| Executes commands | Yes | Yes | Yes |
| Uses a shell | Yes | No | Executes through the remote connection |
| Supports shell operators | Yes | No | Shell syntax may be available |
| Uses normal Ansible module subsystem | Yes | Yes | No |
| Useful for initial Python installation | Not normally | Not normally | Yes |
| Suitable for simple commands | Not usually | Yes | No |
| Suitable for shell pipelines | Yes | No | Possible, but not the normal choice |

**My understanding:**

- Command is my first choice for ordinary commands.
- Shell is used when shell functionality is required.
- Raw is mainly useful for special situations such as preparing a managed node that does not yet have Python.

### 2.5 A Playbook Demonstrating All Three Modules

The following example combines the three modules to demonstrate their differences:

```yaml
---
- name: Understand Ansible command execution modules
  hosts: all
  become: true

  tasks:
    - name: Execute a command using Shell
      ansible.builtin.shell: date

    - name: Execute a command using Command
      ansible.builtin.command: hostname

    - name: Execute a command using Raw
      ansible.builtin.raw: uptime
```

Execute the Playbook:

```bash
ansible-playbook modules.yml
```

This example is for understanding module behavior. In actual automation, I should use dedicated Ansible modules whenever they can perform the required task.

---

## 3. Installing and Removing Linux Packages

Ansible can install or remove packages from managed nodes.

For example, on a suitable Red Hat-based Linux system:

```bash
yum install git -y
```

Package removal:

```bash
yum remove git -y
```

A wildcard can match multiple package names:

```bash
yum remove 'git*' -y
```

The wildcard can match more than one package, so I should inspect the packages that match before removing them.

### 3.1 Removing Packages Through an Ansible Module

Instead of executing Linux package-manager commands directly, Ansible provides package-management modules.

Example:

```yaml
---
- name: Remove Git
  hosts: all
  become: true

  tasks:
    - name: Remove Git package
      ansible.builtin.package:
        name: git
        state: absent
```

The `state` parameter describes the required package state:

| State | Meaning |
|---|---|
| `present` | Ensure the package is installed |
| `absent` | Ensure the package is removed |
| `latest` | Ensure the latest available package version is installed |

For a DNF-based system, I can also use:

```yaml
- name: Install Git
  ansible.builtin.dnf:
    name: git
    state: present
```

The `ansible.builtin.package` module is useful when a Playbook needs to work across different Linux distributions because it selects the appropriate package manager on the managed node.

### 3.2 Understanding the `sed` Command

The session also demonstrates changing text inside a file using `sed`.

General syntax:

```bash
sed -i 's/old_value/new_value/g' filename
```

Explanation:

- `sed`: Stream editor used to process text.
- `-i`: Modify the file directly.
- `s`: Substitute text.
- `old_value`: Text to find.
- `new_value`: Replacement text.
- `g`: Replace all matching occurrences on each line.
- `filename`: File to modify.

Example:

```bash
sed -i 's/install/remove/g' playbook.yml
```

This replaces every occurrence of `install` with `remove` in the specified file.

**Practical consideration:** Using `sed` to replace text in a Playbook can unintentionally modify task names, comments, variables or other configuration values. I should inspect the replacement and keep a backup when necessary.

---

## 4. Variables in Ansible

Variables allow me to store values and reuse them in different tasks.

Without variables, I may need to repeat package names, usernames, directories or other configuration values throughout a Playbook.

For example, if I need to install Git, Maven and Docker, I can define their names once and reference them in the tasks.

### 4.1 Variables Defined Inside a Playbook

The session refers to these as **static variables**. More precisely, these are variables defined using the `vars` keyword at the play level.

Example:

```yaml
---
- name: Install packages using variables
  hosts: all
  become: true

  vars:
    package_one: git
    package_two: maven
    package_three: docker

  tasks:
    - name: Install Git
      ansible.builtin.package:
        name: "{{ package_one }}"
        state: present

    - name: Install Maven
      ansible.builtin.package:
        name: "{{ package_two }}"
        state: present

    - name: Install Docker
      ansible.builtin.package:
        name: "{{ package_three }}"
        state: present
```

**Understanding the syntax:**

- `vars`: Defines variables for the Play.
- `package_one`, `package_two`, `package_three`: Variable names.
- `git`, `maven`, `docker`: Variable values.
- `{{ package_one }}`: Jinja2 expression used to reference a variable.

Meaningful variable names make larger Playbooks easier to understand and maintain.

### 4.2 Runtime Variables Using `--extra-vars`

The session calls these **dynamic variables**. More accurately, these are variables supplied at runtime through the Ansible command line.

They allow me to change values without editing the Playbook.

Example:

```yaml
---
- name: Install a package provided at runtime
  hosts: all
  become: true

  tasks:
    - name: Install the requested package
      ansible.builtin.package:
        name: "{{ package_name }}"
        state: present
```

Execute:

```bash
ansible-playbook install.yml --extra-vars "package_name=tree"
```

Another execution:

```bash
ansible-playbook install.yml --extra-vars "package_name=git"
```

The same Playbook can install different packages depending on the value supplied at execution time.

### 4.3 Passing Multiple Variables

I can supply multiple variables in a single command:

```bash
ansible-playbook install.yml --extra-vars "package_one=tree package_two=git"
```

The Playbook can use both variables:

```yaml
---
- name: Install multiple runtime packages
  hosts: all
  become: true

  tasks:
    - name: Install first package
      ansible.builtin.package:
        name: "{{ package_one }}"
        state: present

    - name: Install second package
      ansible.builtin.package:
        name: "{{ package_two }}"
        state: present
```

### 4.4 Understanding Variable Terminology

| Session terminology | More accurate Ansible explanation |
|---|---|
| Static variable | Variable defined in the Playbook using `vars` |
| Dynamic variable | Variable supplied at runtime using `--extra-vars` |
| Variable reference | Jinja2 expression such as `{{ package_name }}` |

These are useful beginner-level descriptions, but Ansible supports several additional variable sources:

- Inventory variables.
- Group variables.
- Host variables.
- Role variables.
- Variables loaded from external files.
- Registered task results.
- Facts collected from managed nodes.

**Variable precedence:** Ansible has variable precedence rules. When the same variable is defined in multiple places, Ansible determines which value takes priority. Extra variables supplied using `--extra-vars` have the highest variable precedence.

For example, if the Playbook defines:

```yaml
vars:
  package_name: git
```

But I execute:

```bash
ansible-playbook install.yml --extra-vars "package_name=tree"
```

The runtime value `tree` takes precedence over the Playbook value `git`.

---

## 5. Loops in Ansible

Loops help execute the same task repeatedly with different values.

Without loops, I may need to write separate tasks for every package, user or group. Using loops, I can reuse one task and provide multiple values.

### 5.1 Installing Multiple Packages Using `with_items`

The session demonstrates the older `with_items` syntax:

```yaml
---
- name: Install multiple packages
  hosts: all
  become: true

  tasks:
    - name: Install required packages
      ansible.builtin.package:
        name: "{{ item }}"
        state: present
      with_items:
        - git
        - maven
        - docker
        - httpd
        - tree
```

**Understanding `item`:** Ansible processes each value in the list and makes the current value available through the variable `item`.

1. First iteration: `item = git`
2. Second iteration: `item = maven`
3. Third iteration: `item = docker`
4. Fourth iteration: `item = httpd`
5. Fifth iteration: `item = tree`

The same task is executed for each value.

### 5.2 Modern Loop Syntax

`with_items` remains supported, but the modern recommended syntax is `loop`.

Equivalent example:

```yaml
---
- name: Install multiple packages
  hosts: all
  become: true

  tasks:
    - name: Install required packages
      ansible.builtin.package:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - maven
        - docker
        - httpd
        - tree
```

For package installation, I can simplify this further by passing the complete list directly to the package module:

```yaml
---
- name: Install multiple packages
  hosts: all
  become: true

  tasks:
    - name: Install required packages
      ansible.builtin.package:
        name:
          - git
          - maven
          - docker
          - httpd
          - tree
        state: present
```

This is generally more efficient than processing each package through a separate loop iteration.

### 5.3 Creating Multiple Users

Ansible provides the User module to create and manage Linux user accounts.

Example:

```yaml
---
- name: Create multiple Linux users
  hosts: all
  become: true

  tasks:
    - name: Create users
      ansible.builtin.user:
        name: "{{ item }}"
        state: present
      loop:
        - rahul
        - shiva
        - ramesh
        - mahesh
        - ravi
```

**Scenario:** Suppose a company provisions several Linux servers for different development teams. Instead of manually creating every user on every server, I can maintain a list of usernames and use Ansible to create those accounts consistently across the managed nodes.

### 5.4 Creating Multiple Groups

Ansible also provides the Group module.

Example:

```yaml
---
- name: Create multiple Linux groups
  hosts: all
  become: true

  tasks:
    - name: Create groups
      ansible.builtin.group:
        name: "{{ item }}"
        state: present
      loop:
        - developers
        - testers
        - administrators
```

This creates the specified groups on the managed nodes.

### 5.5 Removing Users and Groups

The same modules can remove users and groups by changing their state.

Remove users:

```yaml
- name: Remove users
  ansible.builtin.user:
    name: "{{ item }}"
    state: absent
  loop:
    - rahul
    - shiva
```

Remove groups:

```yaml
- name: Remove groups
  ansible.builtin.group:
    name: "{{ item }}"
    state: absent
  loop:
    - developers
    - testers
```

**Important consideration:** Removing a user account and removing that user's home directory are separate considerations. If the home directory also needs to be deleted, the User module provides a `remove: true` option when used with `state: absent`.

---

## 6. Troubleshooting Ansible Managed Nodes

Ansible normally communicates with Linux managed nodes through SSH.

For successful automation, I need to check:

- Whether the managed node is included in the inventory.
- Whether the control node can reach the managed node.
- Whether SSH authentication works.
- Whether the remote user has the required permissions.
- Whether the managed node has a usable Python interpreter for normal Ansible modules.

### 6.1 Understanding `known_hosts`

The SSH client maintains a file named:

```bash
~/.ssh/known_hosts
```

Inspect it using:

```bash
cat ~/.ssh/known_hosts
```

This file stores SSH host keys for remote machines that the client has previously trusted.

**Important distinction:** `known_hosts` does not contain a live list of connected managed nodes. It does not prove that a server is currently online or reachable. It helps SSH verify that a remote server presents the expected host identity.

If a server is rebuilt and its SSH host key changes, an SSH host-key verification error may occur. I should verify the server's identity before removing or replacing an old host-key entry.

### 6.2 Listing Inventory Hosts

Command:

```bash
ansible all --list-hosts
```

This displays the hosts selected from the Ansible inventory.

Example:

```text
hosts (2):
  192.168.1.10
  192.168.1.11
```

This command confirms which inventory hosts match the selection. It does not test whether those machines are currently reachable. An inventory can contain a server that is powered off or has an incorrect IP address.

### 6.3 Testing Ansible Connectivity Using Ping

Command:

```bash
ansible all -m ansible.builtin.ping
```

A successful result looks similar to:

```text
192.168.1.10 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**Important terminology:** Ansible Ping is not the same as the Linux `ping` command.

- Linux `ping` commonly uses ICMP to test network reachability.
- Ansible Ping verifies that Ansible can connect to the managed node and execute a Python-based module successfully.

A successful Ansible Ping indicates that SSH connectivity, authentication and the required remote Python environment are working for this test.

If it fails, I should investigate the actual error instead of assuming that the server is offline.

---

## 7. Collecting CPU and Memory Information

Ansible can collect system information from managed nodes using the Setup module. This information is called **Ansible facts**.

Facts are automatically collected when a Playbook uses the default fact-gathering behavior. They can also be collected manually using an ad-hoc command.

### 7.1 Collecting System Facts

Command:

```bash
ansible all -m ansible.builtin.setup
```

This gathers information such as:

- Operating system.
- Distribution and version.
- Hostname.
- Network interfaces.
- IP addresses.
- CPU information.
- Memory information.
- Architecture.
- Python information.

### 7.2 Filtering CPU Information

The session demonstrates:

```bash
ansible all -m ansible.builtin.setup | grep -i CPUs
```

This searches the output for text containing `CPUs`. However, Ansible provides specific facts that are more useful than relying on a text search.

For example:

```bash
ansible all -m ansible.builtin.setup -a 'filter=ansible_processor_vcpus'
```

Other useful CPU facts include:

- `ansible_processor_count`
- `ansible_processor_cores`
- `ansible_processor_vcpus`

These facts describe different aspects of the processor configuration. They should not automatically be treated as interchangeable.

### 7.3 Checking Memory Information

The session demonstrates:

```bash
ansible all -m ansible.builtin.shell -a "free -h"
```

The Linux `free -h` command displays memory usage in human-readable units.

Example:

```text
               total        used        free
Mem:           3.8Gi       1.2Gi       1.5Gi
Swap:             0B          0B          0B
```

The actual values depend on the managed node.

Ansible can also collect memory information through facts:

```bash
ansible all -m ansible.builtin.setup -a 'filter=ansible_memory_mb'
```

**Scenario:** Suppose I have several application servers and one of them is experiencing high memory utilization. I can use Ansible to collect information from multiple managed nodes and compare their CPU and memory configurations.

This helps identify whether a machine has fewer resources than the workload requires.

For actual performance monitoring, I would also need metrics such as CPU utilization, memory pressure and application-level performance. CPU and memory capacity alone do not tell me how heavily a server is being used.

---

## 8. Increasing AWS EC2 Computing Capacity

An EC2 instance provides computing resources such as virtual CPUs, memory, network capacity and storage connectivity.

An instance type determines the hardware resources available to an EC2 instance. A small instance type may be sufficient for basic testing, but a larger workload may require additional CPU and memory.

### 8.1 Understanding the Resize Scenario

The session demonstrates changing an EC2 instance from:

```text
t2.micro → t2.medium
```

The objective is to provide additional computing resources.

This is an example of **vertical scaling**.

- Vertical scaling: Increase or decrease resources on an existing server.
- Horizontal scaling: Add more servers to distribute the workload.

### 8.2 Changing the EC2 Instance Type

For a compatible EBS-backed EC2 instance, the general procedure is:

1. Open the AWS Management Console.
2. Navigate to EC2 → Instances.
3. Select the required instance.
4. Stop the instance.
5. Open Actions → Instance settings → Change instance type.
6. Select the required instance type.
7. Apply the change.
8. Start the instance again.

**Why must the instance be stopped?**

AWS requires the instance to be stopped before changing its instance type through the normal procedure for a compatible EBS-backed instance.

Stopping the instance causes downtime while the change is made.

### 8.3 Verify the New Computing Resources

After the instance starts:

1. Connect to the Ansible control node.
2. Check whether the managed node is reachable.
3. Collect system facts again.
4. Compare the CPU and memory information.

Commands:

```bash
ansible all -m ansible.builtin.ping
```

```bash
ansible all -m ansible.builtin.setup
```

The collected facts can help confirm that the managed operating system recognizes the resources available after resizing.

### 8.4 Important AWS Considerations

Before changing an EC2 instance type, I should consider:

- **Compatibility:** The existing AMI and instance configuration must support the target instance type.
- **Downtime:** The instance must be stopped for the normal EBS-backed resize procedure.
- **Billing:** The new instance type has a different pricing rate.
- **Public IP:** A public IPv4 address assigned automatically can change when the instance is stopped and started.
- **Elastic IP:** An Elastic IP can help retain a stable public IPv4 address.
- **Storage:** Changing the instance type is not the same as increasing EBS storage capacity.

**Real-world DevOps scenario:** Suppose an application server initially runs on a small EC2 instance. As application traffic increases, the server may need more CPU and memory. I can evaluate the workload, select a compatible instance type, plan the downtime, resize the instance and verify the resources afterward.

For production systems, I should plan the change carefully. Depending on the application architecture, a maintenance window or traffic migration to another server may be necessary.

---

## 9. Additional Concepts

### 9.1 Ansible Vault

Ansible Vault is used to encrypt sensitive information.

Examples include:

- Database passwords.
- Application credentials.
- API tokens.
- Sensitive variables.

Instead of storing sensitive values in plain text, I can encrypt files or individual variables using Ansible Vault. This helps protect confidential information stored alongside automation code.

### 9.2 Local Connection

Ansible can also execute tasks on the control node itself instead of connecting to a separate managed node over SSH.

Example:

```yaml
---
- name: Execute tasks locally
  hosts: localhost
  connection: local

  tasks:
    - name: Display hostname
      ansible.builtin.command: hostname
```

This is useful when I need Ansible to perform an operation on the machine where Ansible itself is running.

### 9.3 GitHub and GitLab

GitHub and GitLab are platforms used to host and manage source code repositories. Both support collaboration and software development workflows.

GitLab also provides integrated CI/CD capabilities.

In DevOps workflows, source code can be stored in a repository and automation pipelines can use that code to build, test and deploy applications.

### 9.4 Ansible Control Node and Managed Nodes

An Ansible environment commonly contains:

- **Control Node:** The machine where Ansible is installed and from which automation is initiated.
- **Managed Nodes:** The remote machines that Ansible manages.

Architecture:

```text
                 Ansible Control Node
                         |
                    SSH Connection
                         |
          +--------------+--------------+
          |              |              |
       Server 1       Server 2       Server 3
       Managed        Managed        Managed
        Node            Node           Node
```

The control node reads the inventory and executes tasks against the selected managed nodes. This architecture makes it possible to manage multiple servers centrally.

---

## 10. Important Commands – Quick Revision

| Requirement | Command |
|---|---|
| List inventory hosts | `ansible all --list-hosts` |
| Test Ansible connectivity | `ansible all -m ansible.builtin.ping` |
| Collect system facts | `ansible all -m ansible.builtin.setup` |
| Filter CPU virtual processor fact | `ansible all -m ansible.builtin.setup -a 'filter=ansible_processor_vcpus'` |
| Filter memory facts | `ansible all -m ansible.builtin.setup -a 'filter=ansible_memory_mb'` |
| Check memory using Linux | `ansible all -m ansible.builtin.shell -a "free -h"` |
| Display SSH known hosts | `cat ~/.ssh/known_hosts` |
| Execute a Playbook | `ansible-playbook playbook.yml` |
| Pass one runtime variable | `ansible-playbook playbook.yml --extra-vars "name=value"` |
| Pass multiple runtime variables | `ansible-playbook playbook.yml --extra-vars "a=value1 b=value2"` |

---

## 11. Key Learning Points

1. Ansible provides Shell, Command and Raw modules for different command-execution requirements.
2. Command is suitable for simple commands, Shell is useful when shell functionality is required, and Raw is useful for special cases such as initial Python installation.
3. Package-management modules provide a better way to maintain the desired package state than repeatedly executing installation commands.
4. Variables make Playbooks reusable and allow values to be supplied at runtime.
5. Ansible variable precedence determines which value is used when the same variable is defined in multiple places.
6. Loops help automate repetitive tasks involving packages, users and groups.
7. The `loop` keyword is the modern syntax, while `with_items` remains supported.
8. Ansible inventory identifies the machines targeted by automation.
9. SSH `known_hosts` stores trusted host keys; it is not a live connectivity list.
10. Ansible Ping verifies Ansible connectivity and remote Python execution rather than performing an ICMP network test.
11. The Setup module collects facts about managed nodes, including CPU, memory, operating system and network information.
12. Changing an EC2 instance type is an example of vertical scaling and requires planning for compatibility, downtime and cost.

The important connection between these topics is that Ansible is not only a tool for executing commands. It can also maintain the desired configuration of multiple servers, collect system information and support infrastructure administration through repeatable automation.

---

## 12. Verification Summary – What Changed and Why?

| Original topic or approach | Classification | Verification outcome |
|---|---|---|
| Shell module | Still Valid | Its purpose and behavior remain the same. |
| Command module | Still Valid | Preferred for simple commands without shell processing. |
| Raw module | Clarification Required | Mainly for special cases such as bootstrapping Python-less systems. |
| `yum install` and `yum remove` | Still Valid / Alternative Approach | Linux commands remain usable, but Ansible package modules are preferable for configuration management. |
| `ansible.builtin.yum` | Clarification Required | Current documentation redirects it to the DNF implementation. |
| `sed` replacement | Still Valid / Clarification Required | The command works, but broad replacements can damage Playbook content. |
| Static and dynamic variables | Clarification Required | Useful learning labels, but not the full official Ansible variable classification. |
| `--extra-vars` | Still Valid | Supported and has the highest variable precedence. |
| `with_items` | Alternative Approach | Still supported; `loop` is recommended for new Playbooks. |
| Installing packages with a loop | Alternative Approach | Passing a package list directly is generally more efficient. |
| Creating users and groups | Still Valid | User and Group modules remain appropriate. |
| SSH `known_hosts` | Clarification Required | Stores trusted host keys, not live connection status. |
| `ansible all --list-hosts` | Still Valid | Lists inventory matches, not reachable machines. |
| Ansible Ping | Clarification Required | Tests Ansible connectivity and Python-based module execution, not ICMP. |
| Setup module | Still Valid | Remains the standard way to collect Ansible facts. |
| `free -h` | Still Valid | Useful Linux memory-inspection command. |
| EC2 instance resizing | Still Valid | The workflow remains valid for compatible EBS-backed instances, subject to AWS requirements. |
| Ansible Vault | Still Valid | Remains a tool for protecting sensitive information. |
| Local connection | Still Valid | Useful for running tasks on the control node. |

### Important Practical Distinctions

**A. Ansible inventory vs SSH known_hosts**

- Inventory: Which machines should Ansible manage?
- `known_hosts`: Which SSH server identities has the client previously trusted?
- Ansible Ping: Can Ansible successfully connect and execute its test module?

These three serve different purposes.

**B. Package installation vs command execution**

Both of these may install a package:

```yaml
ansible.builtin.command: yum install git -y
```

```yaml
ansible.builtin.package:
  name: git
  state: present
```

However, the package module understands the desired state. It can check whether the package is already installed and avoid unnecessary changes.

This is an important Ansible concept called **idempotency**: repeatedly applying the same desired configuration should not continually produce unnecessary changes.

**C. CPU and memory capacity vs utilization**

Collecting CPU and memory facts tells me about the machine's available configuration. It does not necessarily tell me how much CPU or memory the application is consuming at a particular moment.

For capacity, I can inspect system facts. For utilization and performance monitoring, I need monitoring tools and runtime metrics.

**D. Vertical scaling vs horizontal scaling**

- Vertical scaling: Increase resources on an existing server.
- Horizontal scaling: Add more servers to distribute the workload.

Changing an EC2 instance from `t2.micro` to `t2.medium` is vertical scaling. Adding more application instances behind a load balancer is horizontal scaling.

---

## 13. Official Technical References

- Ansible Shell module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/shell_module.html
- Ansible Command module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/command_module.html
- Ansible Raw module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/raw_module.html
- Ansible Package module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/package_module.html
- Ansible DNF module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/dnf_module.html
- Ansible Variables: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html
- Ansible Loops: https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_loops.html
- Ansible Connection Details: https://docs.ansible.com/projects/ansible/latest/inventory_guide/connection_details.html
- Ansible Ping module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/ping_module.html
- Ansible Setup module: https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/setup_module.html
- AWS EC2 Instance Type Changes: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html
- AWS EC2 Resize Compatibility: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/resize-limitations.html
