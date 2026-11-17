# Ansible Day 4 – Conditions, Handlers, Vault, Roles and Additional Modules

## 1. Introduction

Ansible provides several features that help automate infrastructure configuration and application deployment across different servers. In real-world environments, every server may not use the same operating system, package manager, configuration or application requirements.

For example, an organization may maintain Ubuntu servers for some applications and Red Hat-based servers for other applications. Ansible allows us to manage these different environments using conditions, handlers, variables, modules, roles and reusable automation content.

In this section, I explore how these features help create flexible, reusable and manageable automation workflows.

---

## 2. Homogeneous and Heterogeneous Environments

Infrastructure environments can be classified based on whether the systems have similar or different configurations.

### Homogeneous Environment

A homogeneous environment contains servers with similar operating systems, configurations and software requirements.

For example:
- Five Ubuntu servers running the same application.
- Multiple Amazon Linux servers configured with the same packages.
- A group of web servers using the same application configuration.

In this type of environment, we can often execute the same Ansible tasks across all managed nodes because their requirements are similar.

### Heterogeneous Environment

A heterogeneous environment contains servers with different operating systems, configurations or software requirements.

For example:
- Ubuntu servers and Red Hat servers in the same infrastructure.
- Different Linux distributions running the same application.
- Development, testing and production servers with different configurations.

In this situation, a single task may not work on every server because package names and package managers can differ. Ansible conditions help us execute the appropriate task based on the managed node's operating system or other facts.

**Real-world scenario:** An organization may run an application on Ubuntu and Red Hat servers. Instead of maintaining completely separate Playbooks, we can use Ansible facts and conditions to select the appropriate installation task for each operating system.

---

## 3. Ansible Conditions (`when`)

Ansible conditions allow us to execute tasks only when a specified condition is satisfied. They are useful when different servers require different actions or when a task should run only after checking a particular value.

The `when` keyword is commonly used to control task execution based on variables, facts, operating system information and other conditions.

For example, if a Playbook manages both Ubuntu and Red Hat servers, we can check the operating system family and install the appropriate package.

### Understanding OS Family Facts

Ansible gathers information about managed nodes through its facts-gathering mechanism. These facts contain details such as:

- Operating system name and version.
- Operating system family.
- IP addresses and network interfaces.
- Memory and processor information.
- Hostname and other system details.

The `ansible_facts['os_family']` fact identifies the operating system family.

Common values include:

| Operating System | OS Family |
|---|---|
| Ubuntu | Debian |
| Debian | Debian |
| Red Hat Enterprise Linux | RedHat |
| CentOS | RedHat |
| Amazon Linux | RedHat |

We can use this information in a `when` condition to select the correct task.

### Example: Install Git Based on OS Family

**Ubuntu/Debian – APT:**

```yaml
- name: Install Git on Ubuntu or Debian
  ansible.builtin.apt:
    name: git
    state: present
    update_cache: yes
  when: ansible_facts['os_family'] == "Debian"
```

**Red Hat-based systems – DNF:**

```yaml
- name: Install Git on Red Hat-based systems
  ansible.builtin.dnf:
    name: git
    state: present
  when: ansible_facts['os_family'] == "RedHat"
```

In this example, Ansible checks the operating system family before executing the task. Ubuntu and Debian systems execute the APT task, while Red Hat-based systems execute the DNF task.

### Alternative: Generic Package Module

Ansible also provides the `ansible.builtin.package` module. It can use the appropriate package manager on the managed node without requiring us to write separate APT and DNF tasks for a package available under the same name.

```yaml
- name: Install Git
  ansible.builtin.package:
    name: git
    state: present
```

This approach is useful when the package name is consistent across supported operating systems. When package names or installation options differ, operating-system-specific tasks and conditions provide more control.

### Example: Install Apache

Apache package names differ between operating systems.

**Ubuntu/Debian – APT:**

```yaml
- name: Install Apache on Ubuntu
  ansible.builtin.apt:
    name: apache2
    state: present
    update_cache: yes
  when: ansible_facts['os_family'] == "Debian"
```

**Red Hat-based systems – DNF:**

```yaml
- name: Install Apache on Red Hat
  ansible.builtin.dnf:
    name: httpd
    state: present
  when: ansible_facts['os_family'] == "RedHat"
```

Notice that Ubuntu uses the package name `apache2`, whereas Red Hat-based systems use `httpd`. This is why checking only the package manager is not always sufficient; we must also understand the package names used by each distribution.

---

## 4. Ansible Handlers and Notify

Handlers are special tasks that run when another task notifies them. They are commonly used for actions that should happen only when a configuration or resource has actually changed.

For example, when we update an Apache configuration file, the web server may need to be restarted or reloaded to apply the new configuration. Instead of restarting Apache every time the Playbook runs, we can notify a handler. Ansible executes that handler when the notifying task reports a change.

The `notify` keyword connects a normal task to a handler by its name.

### Example: Install Apache and Start the Service

The installation and service-starting tasks should be handled separately. Installing a package does not automatically mean that a handler should be used to start the service.

**Ubuntu/Debian – APT:**

```yaml
- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present
    update_cache: yes

- name: Ensure Apache is running
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: yes
```

**Red Hat-based systems – DNF:**

```yaml
- name: Install Apache
  ansible.builtin.dnf:
    name: httpd
    state: present

- name: Ensure Apache is running
  ansible.builtin.service:
    name: httpd
    state: started
    enabled: yes
```

The `state: started` option ensures that the service is running, while `enabled: yes` configures it to start automatically during system boot.

### Example: Restart Apache When Configuration Changes

A handler is more useful when a configuration file changes and the service needs to reload or restart.

**Ubuntu/Debian:**

```yaml
- name: Update Apache configuration
  ansible.builtin.copy:
    src: apache2.conf
    dest: /etc/apache2/apache2.conf
  notify: Restart Apache

handlers:
  - name: Restart Apache
    ansible.builtin.service:
      name: apache2
      state: restarted
```

**Red Hat-based systems:**

```yaml
- name: Update Apache configuration
  ansible.builtin.copy:
    src: httpd.conf
    dest: /etc/httpd/conf/httpd.conf
  notify: Restart Apache

handlers:
  - name: Restart Apache
    ansible.builtin.service:
      name: httpd
      state: restarted
```

### How Handlers Work

1. Ansible executes the normal tasks in the Playbook.
2. A task checks or modifies a resource.
3. If the task reports a change, it notifies the associated handler.
4. Ansible runs the notified handler at the appropriate point, normally after the tasks in the play have completed.
5. If the configuration has not changed, the handler is not triggered.

**Important:** A handler is not simply another task that runs after every task. It runs when it has been notified by a task that reports a change. This helps avoid unnecessary service restarts.

**Real-world scenario:** When deploying an application, Ansible may update Nginx or Apache configuration files. A handler can restart or reload the service only when the configuration has changed, reducing unnecessary service interruptions.

---

## 5. Ansible Vault

Ansible Vault is a feature used to encrypt sensitive information in Ansible files. Infrastructure automation often requires credentials and secrets such as database passwords, API tokens, SSH-related secrets and application credentials.

Storing these values directly in plain-text Playbooks or variable files can expose sensitive information. Ansible Vault helps protect this data by encrypting files or individual variables so that the contents are not readable without the required Vault password or secret.

Vault is particularly useful when automation files are stored in Git repositories or shared among team members. The encrypted file can be maintained with the automation code without exposing its protected contents.

### Common Ansible Vault Commands

**Create a new encrypted file:**

```bash
ansible-vault create credentials.yml
```

This creates a new file and opens it for editing. The contents are encrypted when saved.

**Encrypt an existing file:**

```bash
ansible-vault encrypt credentials.yml
```

This encrypts an existing plain-text file.

**Edit an encrypted file:**

```bash
ansible-vault edit credentials.yml
```

This opens the encrypted file for editing after authentication.

**Change the Vault password:**

```bash
ansible-vault rekey credentials.yml
```

This allows us to replace the existing Vault password with a new one.

**View an encrypted file:**

```bash
ansible-vault view credentials.yml
```

This displays the decrypted contents in the terminal after authentication.

**Decrypt a file:**

```bash
ansible-vault decrypt credentials.yml
```

This converts the encrypted file back into plain text. The resulting file should be protected carefully because its contents are no longer encrypted.

### Using Vault with an Ansible Playbook

When a Playbook requires encrypted variables, we can provide the Vault password during execution.

```bash
ansible-playbook deploy.yml --ask-vault-pass
```

Ansible prompts for the Vault password and uses it to decrypt the required information during execution.

### Real-World Scenario: QA and Production Credentials

Consider an organization that maintains separate QA and production environments.

- QA servers use QA database credentials.
- Production servers use production database credentials.
- Both environments are managed through Ansible automation.

We can store the credentials in separate encrypted variable files, such as `qa-secrets.yml` and `production-secrets.yml`.

This protects the credentials from being exposed as plain text in the repository. However, encryption alone does not decide who is allowed to deploy to production.

Production access should also be controlled through appropriate permissions, separate inventories, CI/CD approval processes, restricted access to Vault passwords and deployment authorization.

**Important:** Ansible Vault provides encryption for sensitive data. It is not a complete production access-control or deployment-approval system.

---

## 6. YAML Troubleshooting and YAML Lint

Ansible Playbooks are commonly written in YAML. YAML uses indentation and a structured format to represent tasks, variables, lists and other configuration data.

Because YAML is indentation-sensitive, incorrect spacing or formatting can cause errors when Ansible reads a Playbook. Even a small indentation mistake can change the structure of a task or make the file invalid.

For this reason, checking YAML syntax and formatting is an important part of writing and troubleshooting Ansible automation.

### Common YAML Mistakes

- Incorrect indentation.
- Missing colons.
- Incorrect list formatting.
- Incorrect quotation marks.
- Using invalid comment syntax.
- Placing task parameters at the wrong indentation level.

### YAML Lint

YAML Lint is a tool that checks YAML files for syntax and formatting problems. It helps identify common mistakes before the files are used in an automation workflow.

It is useful when maintaining larger Playbooks because it can identify formatting problems that may be difficult to notice manually.

However, YAML Lint does not replace Ansible-specific validation. A YAML file may be syntactically valid but still contain an invalid Ansible module or incorrect task configuration.

### Ansible Syntax Check

We can use Ansible's syntax-check option to validate the Playbook structure before executing it.

```bash
ansible-playbook playbook.yml --syntax-check
```

This checks the Playbook syntax without performing the deployment tasks.

### YAML Comments

YAML comments use the `#` symbol. They are useful for adding explanations to configuration files without affecting execution.

Example:

```yaml
# Install Apache web server
- name: Install Apache
  ansible.builtin.apt:
    name: apache2
    state: present
```

For multiple comment lines, use `#` at the beginning of each line.

```yaml
# This Playbook installs Apache.
# It also ensures that the service is running.
# The service is enabled during system startup.
```

C-style block comments such as `/* ... */` are not valid YAML comment syntax.

---

## 7. Ansible Check Mode (Dry Run)

Ansible Check Mode is used to preview the changes that a Playbook would make without applying supported changes to managed systems.

It is useful when we want to review an automation task before executing it against important infrastructure. For example, before changing a production configuration, we can run the Playbook in Check Mode to inspect the expected changes.

### Command

```bash
ansible-playbook playbook.yml --check
```

This runs the Playbook in Check Mode.

### Why Check Mode Is Useful

- Helps review expected configuration changes.
- Reduces the risk of applying unintended changes.
- Allows us to inspect supported tasks before actual execution.
- Is useful when testing changes to infrastructure automation.

### Check Mode vs Syntax Check

These two options serve different purposes.

| Option | Purpose |
|---|---|
| `--syntax-check` | Checks whether the Playbook syntax is valid. |
| `--check` | Attempts to simulate supported task changes without applying them. |

**Important:** Check Mode is not a guarantee that every task will be simulated accurately. Some modules do not fully support it, and tasks that depend on previous changes may behave differently during a simulation. It should be treated as a preview, not as a complete replacement for testing.

---

## 8. Ansible PIP Module

The Ansible PIP module is used to install and manage Python packages on managed nodes. Python applications often depend on external libraries to provide additional functionality.

For example:
- NumPy is used for numerical calculations and array operations.
- Pandas is used for data analysis and data manipulation.
- Other Python libraries may be required by application code or automation scripts.

Instead of manually connecting to every server and running PIP commands, Ansible can manage Python package installation across multiple managed nodes.

The `ansible.builtin.pip` module can install, upgrade or remove Python packages. It can also install packages into a specified Python virtual environment.

### Install PIP Prerequisites

Before using the PIP module, the managed node needs a suitable Python environment and PIP installation.

**Ubuntu/Debian – APT:**

```bash
sudo apt update
sudo apt install python3-pip python3-venv -y
```

**Red Hat-based systems – DNF:**

```bash
sudo dnf install python3-pip -y
```

### Example: Install NumPy

```yaml
- name: Install NumPy
  ansible.builtin.pip:
    name: numpy
    state: present
```

### Example: Install Pandas

```yaml
- name: Install Pandas
  ansible.builtin.pip:
    name: pandas
    state: present
```

### Installing Packages in a Virtual Environment

Modern Linux distributions may restrict installing Python packages directly into the system-managed Python environment. This behavior is related to the externally managed environment mechanism described by PEP 668.

A virtual environment provides an isolated Python environment for an application and helps prevent conflicts with operating-system-managed Python packages.

```yaml
- name: Install NumPy in application virtual environment
  ansible.builtin.pip:
    name: numpy
    state: present
    virtualenv: /opt/myapp/venv
```

This installs NumPy into the specified virtual environment rather than directly into the system Python environment.

**Real-world scenario:** If an application requires specific Python libraries, Ansible can create or manage its environment and install the required packages consistently across multiple application servers.

---

## 9. Ansible Setup Module

The Setup module is used to gather facts from managed nodes. These facts provide information about the systems that Ansible controls.

When Ansible executes a Playbook, fact gathering is normally enabled by default unless it is disabled. The gathered information can then be used in conditions, variables and configuration decisions.

For example, an automation task may need to know the operating system family before installing a package. The Setup module provides the operating system information required for that decision.

### Command

```bash
ansible all -m ansible.builtin.setup
```

This gathers and displays facts from all managed hosts in the inventory.

### Examples of Information Gathered

- Operating system and distribution.
- OS family and version.
- Hostname.
- Network interfaces and IP addresses.
- Memory information.
- Processor details.
- Architecture.

### Using Facts in a Playbook

```yaml
- name: Display operating system family
  ansible.builtin.debug:
    var: ansible_facts['os_family']
```

This displays the operating system family gathered by Ansible.

**Real-world scenario:** When managing a mixed environment containing Ubuntu and Red Hat servers, Ansible can gather facts and use them to select the appropriate package installation or configuration tasks.

---

## 10. Ansible Debug Module

The Debug module is used to display messages, variable values and gathered facts during Playbook execution.

It is especially useful when troubleshooting automation because it allows us to inspect the values Ansible is using. When a condition does not behave as expected, the Debug module can help identify whether a variable contains the expected value.

### Example: Display a Message

```yaml
- name: Display a message
  ansible.builtin.debug:
    msg: "Ansible Playbook execution started"
```

### Example: Display a Variable

```yaml
- name: Display OS family
  ansible.builtin.debug:
    var: ansible_facts['os_family']
```

### Example: Display Multiple Values

```yaml
- name: Display server information
  ansible.builtin.debug:
    msg:
      - "Hostname: {{ inventory_hostname }}"
      - "OS Family: {{ ansible_facts['os_family'] }}"
```

The Debug module does not change the managed system configuration. It only displays information during execution, making it useful for testing conditions, checking variables and understanding Playbook behavior.

---

## 11. Ansible Roles

Ansible Roles provide a structured way to organize automation tasks into reusable components. As a Playbook becomes larger, placing every task in a single file can make it difficult to understand, maintain and reuse.

Roles help divide automation into logical units based on responsibilities. For example, we can create separate roles for installing Maven, creating users, managing files and configuring services.

Each role can contain its own tasks, handlers, variables, templates and other supporting files. This makes automation easier to maintain because related tasks are grouped together instead of being mixed into one large Playbook.

### Why Roles Are Useful

- Organize large Playbooks into smaller components.
- Reuse automation across multiple Playbooks.
- Separate responsibilities between different infrastructure components.
- Make automation easier to read and maintain.
- Support collaboration when multiple team members work on different parts of the infrastructure.

### Example Role Directory Structure

```text
roles/
├── maven/
│   └── tasks/
│       └── main.yml
│
├── user/
│   └── tasks/
│       └── main.yml
│
└── file/
    └── tasks/
        └── main.yml
```

Each role has its own directory. The `tasks/main.yml` file is the default entry point for that role's tasks.

A more complete role can also contain directories such as:

| Directory | Purpose |
|---|---|
| `tasks/` | Contains the tasks executed by the role. |
| `handlers/` | Contains handlers notified by tasks. |
| `defaults/` | Contains default variables that can be overridden easily. |
| `vars/` | Contains role variables with higher precedence than defaults. |
| `files/` | Contains static files that can be copied to managed nodes. |
| `templates/` | Contains Jinja2 templates used to generate configuration files. |
| `meta/` | Contains role metadata and dependencies. |

### Example 1: Maven Role

The Maven role is responsible for installing Maven on managed nodes.

**Ubuntu/Debian – APT:**

```yaml
# roles/maven/tasks/main.yml

- name: Install Maven on Ubuntu
  ansible.builtin.apt:
    name: maven
    state: present
    update_cache: yes
  when: ansible_facts['os_family'] == "Debian"
```

**Red Hat-based systems – DNF:**

```yaml
# roles/maven/tasks/main.yml

- name: Install Maven on Red Hat
  ansible.builtin.dnf:
    name: maven
    state: present
  when: ansible_facts['os_family'] == "RedHat"
```

This role checks the operating system family and selects the appropriate package manager.

### Example 2: User Role

A user role can manage user accounts across multiple managed nodes.

```yaml
# roles/user/tasks/main.yml

- name: Create DevOps user
  ansible.builtin.user:
    name: devops
    state: present
    create_home: yes
```

This task ensures that a user named `devops` exists on the managed nodes. If the user already exists, Ansible maintains the desired state rather than creating a duplicate account.

### Example 3: File Role

A file role can manage files and directories on managed nodes.

```yaml
# roles/file/tasks/main.yml

- name: Create application file
  ansible.builtin.file:
    path: /opt/myapp/application.txt
    state: touch
    owner: root
    group: root
    mode: '0644'
```

This task ensures that the specified file exists. The `state: touch` option creates the file if it does not exist and updates its timestamp when the task is applied.

### Main Playbook Using Roles

After creating the roles, we can call them from a main Playbook.

```yaml
# master.yml

- name: Configure application servers
  hosts: all
  become: yes

  roles:
    - maven
    - user
    - file
```

Instead of writing all the tasks directly in `master.yml`, we call the roles by their names. Ansible locates each role in the configured roles path and executes its tasks.

### Real-World Scenario

Consider an organization that manages several application servers. Every server needs Maven, a common service account and a specific directory structure.

We can create three separate roles:

- `maven`: installs Maven.
- `user`: creates the required user account.
- `file`: creates application files and directories.

The same roles can then be reused in different Playbooks and environments. If the Maven installation procedure changes, we can update the Maven role without modifying unrelated user or file-management tasks.

---

## 12. Ansible Galaxy

Ansible Galaxy is a platform for discovering and sharing Ansible roles and collections. It provides reusable automation content created by the Ansible community and other publishers.

Instead of developing every automation task from the beginning, we can search for existing content that performs common infrastructure-management tasks. For example, a role may already be available for installing Java, configuring a web server or managing a particular service.

Ansible Galaxy helps reduce repeated work and encourages reuse across automation projects. However, community content should be reviewed before using it in important environments because its quality, maintenance status, supported versions and security practices can vary.

### Roles and Collections

Ansible Galaxy supports both roles and collections.

**Roles:** Roles organize related automation tasks into reusable components. They can include tasks, handlers, variables, templates and files.

**Collections:** Collections are broader packages of Ansible content. A collection can include roles, modules, plugins, playbooks and other reusable resources.

For example, a collection may provide additional modules or plugins that are not included in the standard Ansible built-in content.

### Search for Roles

```bash
ansible-galaxy role search java
```

This searches for roles related to Java.

### Search by Author

```bash
ansible-galaxy role search --author geerlingguy
```

This searches for roles published by the specified author.

### Install a Role

```bash
ansible-galaxy role install geerlingguy.java
```

This downloads and installs the specified role into the configured Ansible roles path.

### Install a Collection

```bash
ansible-galaxy collection install community.general
```

This installs the `community.general` collection, which provides additional modules, plugins and other Ansible content.

### Important Considerations

Before using a role or collection in an organization, we should review:

- Whether it is actively maintained.
- Which Ansible and Python versions it supports.
- Whether it supports the target operating systems.
- Its dependencies and required permissions.
- Its source code and security implications.

**Real-world scenario:** If an automation project needs to configure a service, we can search Ansible Galaxy for existing reusable content. After reviewing the available role or collection, we can install it and integrate it into our Playbook instead of building every task ourselves.

---

## 13. Important Command Reference

| Purpose | Command |
|---|---|
| Display Ansible facts | `ansible all -m ansible.builtin.setup` |
| Check Playbook syntax | `ansible-playbook playbook.yml --syntax-check` |
| Run in Check Mode | `ansible-playbook playbook.yml --check` |
| Create Vault file | `ansible-vault create credentials.yml` |
| Encrypt file | `ansible-vault encrypt credentials.yml` |
| Edit Vault file | `ansible-vault edit credentials.yml` |
| Change Vault password | `ansible-vault rekey credentials.yml` |
| View Vault file | `ansible-vault view credentials.yml` |
| Decrypt Vault file | `ansible-vault decrypt credentials.yml` |
| Run Playbook with Vault password prompt | `ansible-playbook deploy.yml --ask-vault-pass` |
| Search Galaxy roles | `ansible-galaxy role search java` |
| Search roles by author | `ansible-galaxy role search --author geerlingguy` |
| Install a role | `ansible-galaxy role install geerlingguy.java` |
| Install a collection | `ansible-galaxy collection install community.general` |

---

## 14. Practical Understanding and Interview Points

**1. What is a homogeneous environment?**

An environment containing systems with similar operating systems, configurations and software requirements. The same automation tasks can often be applied across these systems.

**2. What is a heterogeneous environment?**

An environment containing systems with different operating systems or configurations. Conditions and facts help Ansible select the appropriate tasks for each system.

**3. What is the purpose of the `when` keyword?**

It controls whether a task should execute based on a specified condition, such as an operating system family or variable value.

**4. What is the purpose of an Ansible handler?**

A handler executes when notified by a task that reports a change. It is commonly used for restarting or reloading services after configuration updates.

**5. What is Ansible Vault?**

Ansible Vault encrypts sensitive information stored in Ansible files, helping protect credentials and secrets from exposure.

**6. Does Ansible Vault control production deployment permissions?**

No. Vault protects sensitive data through encryption. Deployment authorization and production access require separate access controls and approval mechanisms.

**7. What is Ansible Check Mode?**

It attempts to preview supported changes without applying them to managed systems. It is useful for reviewing automation before execution.

**8. What is the purpose of the Setup module?**

It gathers system facts from managed nodes, such as operating system details, network information and hardware information.

**9. What is the purpose of the Debug module?**

It displays messages, variables and fact values during Playbook execution. It helps troubleshoot automation and understand task behavior.

**10. Why are Ansible Roles useful?**

Roles organize related automation tasks into reusable components, making larger Playbooks easier to maintain and share.

**11. What is Ansible Galaxy?**

Ansible Galaxy is a platform for discovering and sharing reusable Ansible roles and collections. It helps teams reuse existing automation content instead of developing everything from scratch.

**12. What is the difference between a role and a collection?**

A role organizes related automation tasks into a reusable structure. A collection is a broader package that can contain roles, modules, plugins, playbooks and other Ansible content.

---

## 15. Overall Understanding

Ansible provides several features that make infrastructure automation flexible and reusable.

- Conditions allow tasks to execute according to system facts and variables.
- Handlers perform actions when notified by tasks that report changes.
- Ansible Vault protects sensitive information through encryption.
- YAML validation and syntax checking help identify configuration mistakes.
- Check Mode provides a preview of supported changes before execution.
- The PIP module manages Python package installation.
- Setup gathers facts from managed nodes.
- Debug displays information that helps troubleshoot Playbooks.
- Roles organize automation into reusable components.
- Ansible Galaxy provides a platform for discovering and sharing roles and collections.

Together, these features help manage different operating systems, protect sensitive information, reduce repeated work and organize automation for larger infrastructure environments.
