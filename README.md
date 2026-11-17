# Ansible Notes

A structured collection of my Ansible learning notes, practical examples, commands and troubleshooting points.

## Learning Progress

### Day 1 – Introduction, Architecture and Practical Setup

[Read Day 1 Notes](Introduction-to-Ansible.md)

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

## Additional Reference

- [Ansible Day 1 Notes Entry](Ansible_Day-1_notes.md) — short Day 1 entry pointing to the detailed introduction and practical setup notes.

## Learning Approach

These notes are organized by learning day and focus on understanding Ansible concepts through practical scenarios, commands, execution flow and troubleshooting. The examples are intended to build a foundation for automating server configuration and application deployment.
