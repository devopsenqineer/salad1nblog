---
layout: post
title: introduction into ansible
description: introduction for ansible
summary: ansible
tags: ansible
minute: 3
lang: en
---

# Ansible - Introduction

Ansible is an open source tool for the automation of the configuration and management of computer systems. Here are some easy points to understand:

- **Automation**: Ansible can automate repetitive tasks, such as the installation of software on a computer.

- **Configuration management**: Ansible can manage and change the settings on a computer.

- **Simplicity**: Ansible uses the simple YAML language.

- **No additional software required**: Ansible has no need for additional software on the computers it manages. 
Ansible uses existing technologies such as OpenSSH and Python.

### Ansible's approach:

To automate tasks on computers or servers, Ansible works in a series of steps:

- **Connect to nodes**: Ansible connects to nodes (called hosts). Nodes are the target endpoints. They can be servers, network devices, or any computer you want to manage with Ansible.

- **Transfer modules**: Ansible pushes small programs, called modules, to these nodes. These modules are used to perform automation tasks in Ansible.

- **Running modules**: Ansible executes the modules and removes them when the execution is complete. These programs are written as resource models for the desired state of the system.

- **Agentless automation**: Ansible is agentless and can therefore manage nodes without the need to install any software on them. Ansible uses the SSH protocol to connect to servers and perform tasks.


### YAML

YAML (**Yet Another Markup Language**) is the language used for Ansible. 

I'm going to show you some basic Ansible Tasks. 
In the future I will delve deeper into Ansible Tasks, Roles and Playbooks in general. 
And I will teach you how to write your own Windows based Ansible modules.


This Task installs the IIS. 

```yaml
- name: Install IIS (Web-Server only)
  ansible.windows.win_feature:
    name: Web-Server
    state: present
```

Documentation: https://docs.ansible.com/ansible/latest/collections/ansible/windows/win_feature_module.html