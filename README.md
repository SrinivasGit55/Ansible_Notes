# Ansible Notes

A structured collection of my Ansible learning notes, practical examples, commands and troubleshooting points.

## Learning Progress

### Introduction, Architecture and Practical Setup

[Ansible Day 1 Notes Entry](1.Introduction-to-Ansible.md)

Covered topics:

- Introduction to Ansible and configuration management
- Ansible architecture: control node and managed nodes
- SSH communication and key-based authentication
- AWS EC2 lab setup with one control node and four managed nodes
- Hostname configuration and Ansible installation
- Inventory files, host groups and host patterns
- Ad-Hoc commands and basic module usage
- Connectivity checks, troubleshooting and Bash history timestamps

### Day 2 – Modules, Playbooks and Tags

[Read Day 2 Notes](Ansible_Day-2_notes.md)

Covered topics:

- Ansible modules and key-value arguments
- Package management with DNF/Yum and APT
- Service, User and Copy modules
- Practical configuration-file distribution using `application.yml`
- Playbooks and YAML structure
- Gathering Facts and understanding task results
- Task execution order and failure handling
- Tags: selecting, executing and skipping tasks
- Multiple package installation and important commands

### Day 3 – Command Execution, Variables, Loops and Troubleshooting

[Read Day 3 Notes](Ansible_Day-3_notes.md)

Covered topics:

- Differences between Shell, Command and Raw modules
- Installing and removing packages with Ansible
- Using `sed` to modify file content
- Playbook variables and runtime variables with `--extra-vars`
- Variable precedence and Jinja2 variable references
- Loops for package installation and user/group management
- SSH `known_hosts`, inventory checks and Ansible Ping
- Gathering CPU and memory facts from managed nodes
- AWS EC2 instance resizing and vertical scaling
- Ansible Vault and local connection mode
- Verified terminology, practical distinctions and current approaches

### Day 4 – Conditions, Handlers, Vault, Roles and Additional Modules

[Read Day 4 Notes](Ansible_Day-4_notes.md)

Covered topics:

- Homogeneous and heterogeneous environments
- Conditions using `when` and operating system facts
- Package installation for Ubuntu/Debian using APT and Red Hat-based systems using DNF
- Generic package module and distribution-specific package names
- Handlers, `notify` and service restart behavior
- Ansible Vault commands and protecting environment credentials
- YAML troubleshooting, YAML Lint and syntax checking
- Check Mode (dry run) and its limitations
- Python package management using the PIP module and virtual environments
- Setup and Debug modules
- Ansible Roles and reusable role directory structure
- Ansible Galaxy, roles and collections

## Learning Approach

These notes are organized by learning day and focus on understanding Ansible concepts through practical scenarios, commands, execution flow and troubleshooting. The examples are intended to build a foundation for automating server configuration and application deployment.
