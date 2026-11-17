# Ansible Day 2 – Modules, Playbooks and Tags

## Learning Objectives

This session builds on the Ansible architecture and practical setup from Day 1. The focus is on using modules to perform individual operations, combining tasks in reusable Playbooks, and selecting tasks with Tags.

Topics covered:

1. Ansible modules and key-value arguments
2. Package management with DNF/Yum and APT
3. Service, User and Copy modules
4. Playbooks and YAML basics
5. Gathering Facts and understanding task results
6. Task order and failure handling
7. Ansible Tags
8. Practical examples and interview questions

---

## 1. Quick Recap: Ansible and Ad-Hoc Commands

Ansible automates server administration activities such as software installation, configuration, service management and application deployment.

Ansible commonly communicates with Linux managed nodes over SSH. The control node sends tasks to the target hosts defined in the inventory.

### What are Ad-Hoc Commands?

Ad-Hoc commands are used to perform quick, individual tasks without creating a Playbook.

Example:

```bash
ansible all -a "yum install git -y"
```

Command breakdown:

| Part | Meaning |
|---|---|
| `ansible` | Ansible command-line tool |
| `all` | All hosts in the inventory |
| `-a` | Arguments passed to the module |
| `yum install git -y` | Command to execute |

Ad-Hoc commands are useful for quick operations. For a sequence of reusable tasks, a Playbook is more suitable.

**Important:** Ad-Hoc commands can use Ansible modules; they are not limited to executing Linux commands.

---

## 2. Ansible Modules

### 2.1 What is a Module?

An Ansible module is a reusable unit that performs a specific operation on a managed node.

| Requirement | Module |
|---|---|
| Install or remove software | Package management module |
| Start or stop a service | Service module |
| Create or remove a user | User module |
| Copy a file | Copy module |

Modules provide a structured way to manage systems instead of writing separate shell commands for every operation.

### 2.2 Understanding Key-Value Pairs

Ansible module arguments commonly use key-value pairs.

Think of an Aadhaar registration form:

| Key | Value |
|---|---|
| Name | Example User |
| Course | DevOps |
| Fee | 8000 |

The key identifies the parameter, and the value supplies the information for that parameter.

Ansible uses the same pattern:

```bash
ansible all -m ansible.builtin.package -a "name=git state=present"
```

- `name` is a key and `git` is its value.
- `state` is a key and `present` is its value.

### 2.3 Why Use Modules?

**Reusability:** Modules can be included in Playbooks and reused for similar operations.

**Idempotency:** An idempotent task maintains the requested state without making unnecessary changes when it is run repeatedly. For example, ensuring that Git is present does not reinstall it if it is already installed.

---

## 3. Package Management Modules

Package management modules install, remove and update software on managed nodes. The module and package name depend on the Linux distribution.

| Linux distribution | Package manager/module |
|---|---|
| Amazon Linux / RHEL family | DNF or Yum |
| Ubuntu / Debian | APT |

### 3.1 Common Package States

| State | Purpose |
|---|---|
| `present` | Ensure the package is installed |
| `absent` | Ensure the package is removed |
| `latest` | Ensure the latest available package version is installed |

### 3.2 Amazon Linux Examples

Install Git:

```bash
ansible all -m ansible.builtin.dnf -a "name=git state=present" --become
```

Install Apache:

```bash
ansible all -m ansible.builtin.dnf -a "name=httpd state=present" --become
```

Remove Apache:

```bash
ansible all -m ansible.builtin.dnf -a "name=httpd state=absent" --become
```

Update Apache to the latest available version:

```bash
ansible all -m ansible.builtin.dnf -a "name=httpd state=latest" --become
```

The Yum module remains available for compatibility, but DNF is the preferred explicit module for modern DNF-based systems such as Amazon Linux 2023.

### 3.3 Ubuntu Examples

Install Git:

```bash
ansible all -m ansible.builtin.apt -a "name=git state=present update_cache=yes" --become
```

Install Apache:

```bash
ansible all -m ansible.builtin.apt -a "name=apache2 state=present" --become
```

Remove Apache:

```bash
ansible all -m ansible.builtin.apt -a "name=apache2 state=absent" --become
```

Update Apache to the latest available version:

```bash
ansible all -m ansible.builtin.apt -a "name=apache2 state=latest" --become
```

### 3.4 Distribution Differences

Apache has different package names:

- Amazon Linux: `httpd`
- Ubuntu: `apache2`

The package management modules also differ:

- Amazon Linux: `ansible.builtin.dnf`
- Ubuntu: `ansible.builtin.apt`

The `--become` option requests privilege escalation so that a task can run with the permissions needed to manage system packages.

### 3.5 Understanding Ansible Results

| Status | Meaning |
|---|---|
| `ok` | Task completed; no change was required |
| `changed` | Task completed and made a change |
| `failed` | Task failed |
| `unreachable` | Ansible could not connect to the host |
| `skipped` | Task was skipped |

For example, if Git is already installed, the task will normally report `ok`. If Ansible installs Git, it will normally report `changed`.

### Interview Questions

**Q: What is the difference between `present`, `absent` and `latest`?**

- `present`: Ensure the package is installed.
- `absent`: Ensure the package is removed.
- `latest`: Ensure the latest available package version is installed.

**Q: How do you install Git using Ansible?**

Amazon Linux:

```bash
ansible all -m ansible.builtin.dnf -a "name=git state=present" --become
```

Ubuntu:

```bash
ansible all -m ansible.builtin.apt -a "name=git state=present" --become
```

---

## 4. Service Module

A package and a service are different:

- A **package** is software installed on the server.
- A **service** is a background process managed by the operating system.

Installing Apache places its software on the server. Starting Apache makes its web server service run.

### 4.1 Common Mistake: Using the Package Module to Start a Service

This command is incorrect:

```bash
ansible all -m ansible.builtin.yum -a "name=httpd state=started"
```

The package module does not start services. `started` is a service state, not a package state.

### 4.2 Start Apache

Amazon Linux:

```bash
ansible all -m ansible.builtin.service -a "name=httpd state=started" --become
```

Ubuntu:

```bash
ansible all -m ansible.builtin.service -a "name=apache2 state=started" --become
```

The Service module is the same, but the service name differs.

### 4.3 Check Service Status

Amazon Linux:

```bash
ansible all -a "systemctl status httpd"
```

Ubuntu:

```bash
ansible all -a "systemctl status apache2"
```

### 4.4 Stop a Service

Amazon Linux:

```bash
ansible all -m ansible.builtin.service -a "name=httpd state=stopped" --become
```

Ubuntu:

```bash
ansible all -m ansible.builtin.service -a "name=apache2 state=stopped" --become
```

### 4.5 Common Service States

| State | Purpose |
|---|---|
| `started` | Ensure the service is running |
| `stopped` | Ensure the service is stopped |
| `restarted` | Restart the service |
| `reloaded` | Reload service configuration where supported |

To start a service and enable it at boot:

```bash
ansible all -m ansible.builtin.service -a "name=httpd state=started enabled=true" --become
```

- `state=started` ensures the service is currently running.
- `enabled=true` configures it to start automatically during system boot.

### Interview Questions

**Q: Which module starts or stops a service?**

The Service module.

**Q: Can the Yum module start Apache?**

No. Package modules manage software installation and removal. The Service module manages services.

**Q: What is the difference between `started` and `enabled`?**

`started` controls the current running state; `enabled` controls whether the service starts during boot.

---

## 5. User Module

The User module manages Linux user accounts on managed nodes.

Create a user:

```bash
ansible all -m ansible.builtin.user -a "name=example_user state=present" --become
```

Remove a user:

```bash
ansible all -m ansible.builtin.user -a "name=example_user state=absent" --become
```

| State | Purpose |
|---|---|
| `present` | Ensure the user exists |
| `absent` | Ensure the user is removed |

**Common mistake:** Do not use the Service module to create a user. Select the module according to the operation: User for accounts, Service for services.

### Interview Questions

**Q: Which module creates a Linux user?**

The User module.

**Q: Can we create a user with the Service module?**

No. The Service module manages services, not user accounts.

---

## 6. Copy Module

The Copy module transfers files from the Ansible control node to managed nodes.

### 6.1 Practical Scenario: application.yml

An application may use a configuration file such as `application.yml` to store properties such as:

- Application name
- Web server address and port
- Application server address and port
- Database server address and port
- Load balancer address

Example:

```yaml
application:
  name: ABC

webserver:
  url: 192.168.100.1
  port: 80

appserver:
  url: 192.168.100.2
  port: 8080

dbserver:
  url: 192.168.100.3
  port: 5432

loadbalancer:
  url: www.abc.com
```

This is an example configuration structure. The exact properties depend on the application.

### 6.2 Common Port Numbers

| Component | Common port |
|---|---:|
| HTTP – Apache / Nginx | 80 |
| Tomcat (default) | 8080 |
| MySQL | 3306 |
| PostgreSQL | 5432 |
| ArangoDB | 8529 |

Ports can be changed according to the application and server configuration.

### 6.3 Why Distribute Configuration Files?

Suppose an application moves to a new database server and its IP address changes. The application configuration must be updated on the relevant servers.

Instead of manually editing the file on every server:

1. Update the file on the Ansible control node.
2. Copy the updated file to managed nodes.
3. Verify that the file is present at the destination.
4. Restart or reload the application if its configuration requires it.

### 6.4 Copy Command

```bash
ansible all -m ansible.builtin.copy -a "src=application.yml dest=/tmp/"
```

| Parameter | Meaning |
|---|---|
| `src` | Source file on the control node |
| `dest` | Destination path on managed nodes |

To copy the file into the root user's directory:

```bash
ansible all -m ansible.builtin.copy -a "src=application.yml dest=/root/" --become
```

Verify the copied file:

```bash
ansible all -a "ls /tmp/"
```

### 6.5 Real-World Examples

**Distributing scripts:** Keep a setup script on the control node, copy it to multiple managed nodes, and then execute it using an appropriate Ansible task.

**Updating application configuration:** Update a configuration file centrally and distribute the same version to the required application servers.

### Interview Questions

**Q: What does the Copy module do?**

It copies files from the Ansible control node to managed nodes.

**Q: What is the difference between `src` and `dest`?**

`src` identifies the source file on the control node; `dest` specifies the destination path on managed nodes.

**Q: What is application.yml?**

It is a configuration file used by some applications to store settings such as application properties, server addresses, ports and database connection details.

---

## 7. Ansible Playbooks

### 7.1 What is a Playbook?

An Ansible Playbook is a YAML file containing one or more plays. A play defines the target hosts and the tasks to execute on them. Tasks normally use Ansible modules to perform specific operations.

### 7.2 Why Use Playbooks?

Consider a requirement to:

1. Install Apache.
2. Start the Apache service.
3. Create a Linux user.

These operations need different modules. A Playbook organizes them in one reusable file and executes them in the specified order.

Advantages:

- Multiple tasks can be organized in one file.
- Automation can be reused.
- The same configuration can be applied consistently to multiple managed nodes.
- Task execution order is clearly defined.

---

## 8. YAML Basics

Ansible Playbooks are written in YAML, a human-readable data serialization language.

The historical expansion of YAML was "Yet Another Markup Language". Its current recursive expansion is "YAML Ain't Markup Language".

YAML is also used in DevOps tools such as Docker Compose and Kubernetes resource definitions.

### 8.1 Important YAML Characteristics

**Human-readable:** YAML uses a readable structure for representing data.

**Indentation-based:** Indentation defines the relationship between elements. Incorrect indentation can make a file invalid or change its structure.

**Key-value structure:**

```yaml
name: httpd
state: present
```

**List format:** A hyphen (`-`) identifies an item in a list.

```yaml
tasks:
  - name: Install Apache
  - name: Start Apache
```

**Document markers:**

- `---` marks the beginning of a YAML document.
- `...` marks the end of a YAML document.

These markers are optional in many ordinary Playbook files.

### 8.2 YAML File Extensions

Both extensions are commonly used:

- `.yml`
- `.yaml`

Examples: `playbook.yml`, `install_apache.yaml`.

A YAML linter can help identify formatting and syntax problems: [yamllint.com](https://www.yamllint.com/).

---

## 9. First Playbook: Install Apache, Start Service and Create a User

### Requirement

Perform three operations on all managed nodes:

1. Install Apache.
2. Start the Apache service.
3. Create a Linux user.

Create a file named `playbook.yml`.

Example for Amazon Linux:

```yaml
---
- name: Configure Apache and user
  hosts: all
  become: true

  tasks:
    - name: Install Apache
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Start Apache service
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: true

    - name: Create a Linux user
      ansible.builtin.user:
        name: example_user
        state: present
...
```

### 9.1 Playbook Breakdown

| Element | Purpose |
|---|---|
| `---` | Starts the YAML document |
| `- name:` | Describes the play |
| `hosts: all` | Targets all hosts in the inventory |
| `become: true` | Enables privilege escalation for the play |
| `tasks:` | Defines the tasks |
| Task `name` | Describes an individual task |
| `ansible.builtin.dnf` | Package management module |
| `ansible.builtin.service` | Service management module |
| `ansible.builtin.user` | User management module |
| `state: present` | Ensures the requested resource exists |

### 9.2 Task Execution Order

Ansible executes tasks in the order in which they are written:

1. Install Apache.
2. Start and enable Apache.
3. Create the Linux user.

### 9.3 Execute the Playbook

```bash
ansible-playbook playbook.yml
```

The Playbook targets the hosts defined by `hosts: all` and executes the tasks in order.

---

## 10. Understanding Playbook Output and Gathering Facts

Ansible reports results for each task. Common statuses include:

- `ok`
- `changed`
- `failed`
- `skipped`
- `unreachable`

At the end of execution, Ansible displays a summary called **PLAY RECAP**.

Example:

```text
ok=4 changed=3 unreachable=0 failed=0 skipped=0
```

The exact values depend on the Playbook and its execution.

### 10.1 What is Gathering Facts?

Gathering Facts collects information about managed nodes before normal tasks run. Examples include:

- Operating system
- IP addresses
- Hostname
- Memory
- Processor information

Fact gathering is enabled by default in a normal Playbook unless disabled.

### 10.2 Why Can ok Be Greater Than the Number of User-Defined Tasks?

Suppose a Playbook has three user-defined tasks. Ansible may also execute the automatic fact-gathering step.

A recap such as:

```text
ok=4 changed=3
```

can represent three user-defined tasks that made changes and one successful fact-gathering step that made no changes.

The `ok` counter is not simply the number of tasks that changed something. The `changed` counter represents tasks that reported a change.

---

## 11. Uninstall Playbook: Why Task Order Matters

Suppose Apache must be removed from a server.

The intended sequence is:

1. Stop the Apache service.
2. Remove the Apache package.
3. Remove the Linux user.

### 11.1 Incorrect Task Order

```yaml
tasks:
  - name: Remove Apache package
    ansible.builtin.dnf:
      name: httpd
      state: absent

  - name: Stop Apache service
    ansible.builtin.service:
      name: httpd
      state: stopped
```

This order is problematic. Removing the package first may also remove the service definition. The next task then tries to stop a service that may no longer exist.

Think of it like trying to celebrate the birthday of a person who is no longer alive. The person must exist before the birthday can be celebrated!

### 11.2 Correct Task Order

```yaml
---
- name: Remove Apache configuration
  hosts: all
  become: true

  tasks:
    - name: Stop Apache service
      ansible.builtin.service:
        name: httpd
        state: stopped

    - name: Remove Apache package
      ansible.builtin.dnf:
        name: httpd
        state: absent

    - name: Remove Linux user
      ansible.builtin.user:
        name: example_user
        state: absent
...
```

Execution logic:

1. Stop the running service.
2. Remove the installed package.
3. Remove the user account.

This example assumes Apache is installed and its service exists. A reusable Playbook should also consider how to handle a service that is already absent.

**Learning point:** Task order matters. A Playbook should follow the correct operational sequence and account for dependencies between tasks.

---

## 12. Task Failure and ignore_errors

By default, when a task fails on a particular host, Ansible stops executing the remaining tasks for that host. Other hosts may continue.

For example:

1. Task 1 installs Apache.
2. Task 2 fails.
3. Task 3 does not normally execute on that affected host.

### 12.1 Using ignore_errors

If an ordinary task failure is expected and the next tasks should continue, we can use:

```yaml
- name: Run a task that may fail
  ansible.builtin.command: /some/command
  ignore_errors: true
```

This tells Ansible to continue after an ordinary task failure.

However, `ignore_errors` does not handle every kind of failure. Syntax errors and connection failures require different handling.

---

## 13. Ansible Tags

### 13.1 What are Tags?

Tags select specific tasks for execution or exclude specific tasks when running a Playbook.

**Practical scenario:** Imagine a Playbook containing ten tasks. If only one operation is required, running the entire Playbook may perform unnecessary work. Tags help select the task or tasks needed.

### 13.2 Playbook with Tags

Example for Amazon Linux:

```yaml
---
- name: Configure software and users
  hosts: all
  become: true

  tasks:
    - name: Install Docker
      ansible.builtin.dnf:
        name: docker
        state: present
      tags:
        - a

    - name: Start Docker service
      ansible.builtin.service:
        name: docker
        state: started
        enabled: true
      tags:
        - b

    - name: Install Maven
      ansible.builtin.dnf:
        name: maven
        state: present
      tags:
        - c

    - name: Create a Linux user
      ansible.builtin.user:
        name: example_user
        state: present
      tags:
        - d
...
```

| Task | Operation | Tag |
|---|---|---|
| Task 1 | Install Docker | `a` |
| Task 2 | Start Docker | `b` |
| Task 3 | Install Maven | `c` |
| Task 4 | Create a user | `d` |

Tags can be short labels or meaningful names such as `install_docker`, `start_docker`, `install_maven` and `create_user`.

### 13.3 Execute a Single Tag

To execute only the Maven installation task (tag `c`):

```bash
ansible-playbook playbook.yml --tags c
```

### 13.4 Execute Multiple Tags

To execute the Docker installation and service tasks (tags `a` and `b`):

```bash
ansible-playbook playbook.yml --tags a,b
```

Ansible selects tasks matching either tag.

### 13.5 Skip a Specific Tag

To skip Maven installation (tag `c`) and run the other eligible tasks:

```bash
ansible-playbook playbook.yml --skip-tags c
```

An alternative is to select the other tags explicitly:

```bash
ansible-playbook playbook.yml --tags a,b,d
```

When the requirement is simply to exclude a task, `--skip-tags` is easier to express.

### 13.6 Skip Multiple Tags

To skip the Docker service task (tag `b`) and Maven installation (tag `c`):

```bash
ansible-playbook playbook.yml --skip-tags b,c
```

### 13.7 Real-World Use Case

A deployment Playbook may contain tasks for installing dependencies, configuring an application, starting a service, updating configuration and validating the deployment.

Tags can select the relevant tasks without running every task.

**Important:** Tags control task selection. They do not automatically resolve dependencies between tasks. If a selected task depends on an earlier task, that dependency must still be considered.

### Interview Questions

**Q: What are Tags in Ansible?**

Tags select specific tasks for execution or exclude selected tasks from a Playbook run.

**Q: How do you execute one tag?**

```bash
ansible-playbook playbook.yml --tags tagname
```

**Q: How do you skip a tag?**

```bash
ansible-playbook playbook.yml --skip-tags tagname
```

**Q: How do you execute multiple tags?**

```bash
ansible-playbook playbook.yml --tags a,b
```

**Q: How do you skip multiple tags?**

```bash
ansible-playbook playbook.yml --skip-tags b,c
```

---

## 14. Installing Multiple Packages in One Task

Multiple packages can be supplied as a list to a package management module. A loop is not required just to install several packages.

Amazon Linux example:

```yaml
- name: Install multiple packages
  ansible.builtin.dnf:
    name:
      - git
      - tree
      - maven
    state: present
```

For Ubuntu, use `ansible.builtin.apt`.

A loop is another option for repeating tasks over a list of items. Choose a structure that is clear and maintainable.

---

## 15. Day 2 Quick Revision

| Topic | Purpose |
|---|---|
| Ansible Module | Performs a specific operation |
| Key-value pair | Defines module parameters and their values |
| DNF / Yum | Package management on supported Linux distributions |
| APT | Package management on Ubuntu and Debian |
| Service module | Starts, stops, restarts or reloads services |
| User module | Creates and removes Linux users |
| Copy module | Copies files from control node to managed nodes |
| Playbook | Organizes plays and tasks in a reusable YAML file |
| YAML | Human-readable data serialization language |
| Gathering Facts | Collects information from managed nodes |
| Tags | Selects tasks for execution or exclusion |
| `--tags` | Selects tasks using tags |
| `--skip-tags` | Excludes tasks using tags |

### Important Commands to Practice

Install a package on Amazon Linux:

```bash
ansible all -m ansible.builtin.dnf -a "name=git state=present" --become
```

Install a package on Ubuntu:

```bash
ansible all -m ansible.builtin.apt -a "name=git state=present" --become
```

Start Apache on Amazon Linux:

```bash
ansible all -m ansible.builtin.service -a "name=httpd state=started enabled=true" --become
```

Start Apache on Ubuntu:

```bash
ansible all -m ansible.builtin.service -a "name=apache2 state=started enabled=true" --become
```

Create a user:

```bash
ansible all -m ansible.builtin.user -a "name=example_user state=present" --become
```

Copy a file:

```bash
ansible all -m ansible.builtin.copy -a "src=application.yml dest=/tmp/"
```

Execute a Playbook:

```bash
ansible-playbook playbook.yml
```

Execute a specific tag:

```bash
ansible-playbook playbook.yml --tags a
```

Skip a specific tag:

```bash
ansible-playbook playbook.yml --skip-tags c
```

---

## Final Understanding

Ansible modules perform individual operations such as installing software, managing services, creating users and copying files.

Playbooks organize multiple tasks in a YAML file so that automation can be executed in a defined order and reused. The uninstall scenario demonstrates why task order matters. Tags make it possible to select particular tasks instead of executing every task in a Playbook.

These concepts provide the foundation for writing organized Ansible automation for server configuration and application deployment.
