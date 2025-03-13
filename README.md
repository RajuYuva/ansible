
---

# Ansible Playbook Examples and Best Practices

This document provides examples of various Ansible concepts, including playbook structure, variable definitions, modules, and role-based organization. Each section contains code snippets and notes to help you understand how to use these features effectively.

---

## Table of Contents

- [Playbook for Installing and Running Nginx](#playbook-for-installing-and-running-nginx)
- [Defining Variables](#defining-variables)
  - [Inline Variables](#inline-variables)
  - [Variables from Files](#variables-from-files)
  - [Prompting for Variables](#prompting-for-variables)
  - [Inventory Variables](#inventory-variables)
  - [Passing Variables via Arguments](#passing-variables-via-arguments)
  - [Lists and Dictionaries](#lists-and-dictionaries)
- [Using Ansible Modules](#using-ansible-modules)
- [Roles and Directory Structure](#roles-and-directory-structure)
- [Sample Role Invocation](#sample-role-invocation)
- [Additional Notes](#additional-notes)

---

## Playbook for Installing and Running Nginx

```yaml
- name: install and run nginx
  hosts: localhost  # Alternatively, use an inventory file (e.g., inventory.ini with groups [web] [db])
  become: yes       # Enables privilege escalation (sudo)
  
  tasks:
    - name: install nginx
      ansible.builtin.package:
        name: nginx
        state: latest

    - name: run nginx
      ansible.builtin.service:
        name: nginx
        state: started  # Options: started, restarted, or reloaded
        enabled: yes
```

**Notes:**
- This playbook installs the latest version of **nginx** and ensures that the service is running.
- The `become: yes` parameter allows the tasks to run with elevated privileges.

---

## Defining Variables

### Inline Variables

```yaml
---
vars:  # Variable declaration within the playbook
  Course: "Terraform"
  Trainer: "Siva"
  Duration: "15hrs"
```

**Note:** Inline variables are defined directly in the playbook file.

### Variables from Files

```yaml
vars_files:
  - vars.yml
```

**Note:** Using external files for variables helps in managing configurations separately and makes playbooks cleaner.

### Prompting for Variables

```yaml
vars_prompt:
  - name: USERNAME
    prompt: Please enter your username
    private: false
  - name: PASSWORD
    prompt: Please enter your password
    private: true
```

**Note:** `vars_prompt` allows you to request variable values at runtime. Mark sensitive inputs (like passwords) as `private` so they are not echoed.

### Inventory Variables

```
[localhost:vars]
Course="DevOps with AWS"
Trainer="Sivakumar"
Duration="120hrs"
WISHES="Morning from inventory"
```

**Note:** Variables defined in an inventory file can be host- or group-specific and will override playbook variables based on Ansible’s precedence rules.

### Passing Variables via Arguments

```yaml
- name: variables from arguments
  hosts: localhost
  tasks:
    - name: get from args
      ansible.builtin.debug:
        msg: "Hello {{ PERSON }}, Good {{ WISHES }}"
```

**Note:** You can pass variables from the command line (using `-e`) to modify playbook behavior dynamically.

### Lists and Dictionaries

```yaml
vars:
  Course: "DevOps with AWS"  # String
  Tools:  # List of tools
    - Linux
    - Shell
    - Ansible
  Experience:  # Dictionary (map) with key-value pairs
    DevOps: 3
    AWS: 2
    Docker: 1
  RealProject: true  # Boolean
```

**Note:** Ansible supports different data types including strings, lists, dictionaries, and booleans for versatile variable handling.

---

## Using Ansible Modules

Below are examples of commonly used Ansible modules with sample tasks:

- **ansible.builtin.package:** Manage packages (e.g., installing nginx).
- **ansible.builtin.service:** Manage services (e.g., starting, restarting).
- **ansible.builtin.debug:** Print messages for debugging.
  ```yaml
  - name: print a message
    ansible.builtin.debug:
      msg: "Hi,"
  ```
- **ansible.builtin.command:** Run commands without a shell.
- **ansible.builtin.shell:** Run commands through a shell (useful for redirections).
- **ansible.builtin.dnf:** Manage packages using DNF.
- **ansible.builtin.user:** Manage user accounts.
  ```yaml
  - name: create expense user
    ansible.builtin.user:
      name: expense
  ```
- **ansible.builtin.file:** Manage files and directories.
  ```yaml
  - name: create app directory
    ansible.builtin.file:
      path: /app
      state: directory
  ```
- **ansible.builtin.get_url:** Download files.
  ```yaml
  - name: download backend code
    ansible.builtin.get_url:
      url: https://expense-builds.s3.us-east-1.amazonaws.com/expense-backend-v2.zip
      dest: /tmp/backend.zip
  ```
- **ansible.builtin.unarchive:** Extract archives.
  ```yaml
  - name: unzip backend code
    ansible.builtin.unarchive:
      src: /tmp/backend.zip
      dest: /app
      remote_src: yes
  ```
- **ansible.builtin.copy:** Copy files to remote hosts.
  ```yaml
  - name: copy backend service
    ansible.builtin.copy:
      src: backend.service
      dest: /etc/systemd/system/backend.service
  ```
- **ansible.builtin.pip:** Install Python packages.
  ```yaml
  - name: install python mysql dependencies
    ansible.builtin.pip:
      name:
        - PyMySQL  # For MySQL connectivity
        - cryptography
      executable: pip3.9  # Adjust pip version as needed
  ```
- **ansible.builtin.systemd_service:** Reload the systemd daemon.
  ```yaml
  - name: daemon reload
    ansible.builtin.systemd_service:
      daemon_reload: true
  ```
- **ansible.builtin.service:** Manage services such as backend and nginx.
  ```yaml
  - name: start and enable backend service
    ansible.builtin.service:
      name: backend
      state: started
      enabled: yes
  - name: restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
  ```

---

## Roles and Directory Structure

Roles help in organizing and reusing your Ansible code by separating tasks, variables, templates, and handlers into a standardized directory structure.

```
roles/
├── backend
│   ├── tasks/
│   │   └── main.yaml
│   ├── templates/
│   │   └── backend.service.j2
│   └── vars/
│       └── main.yaml
├── common
│   └── tasks/
│       └── app-pre-req.yaml
├── db
│   ├── tasks/
│   │   └── main.yaml
│   └── vars/
│       └── main.yaml
└── frontend
    ├── handlers/
    │   └── main.yaml
    ├── tasks/
    │   └── main.yaml
    ├── templates/
    │   └── expense.conf.j2
    └── vars/
        └── main.yaml
```

**Note:** This structure supports the DRY (Don't Repeat Yourself) principle by reusing code across different playbooks and environments.

---

## Sample Role Invocation

To use a role, create a playbook file that references the role. For example, in `backend.yaml`:

```yaml
- name: configure backend server
  hosts: backend
  become: yes
  vars_files:
    - credentials.yaml
  roles:
    - backend
```

**Note:** The playbook calls the `backend` role which encapsulates all tasks, variables, and templates required to configure a backend server. Ensure that all referenced files (e.g., `credentials.yaml`) and the inventory file are correctly set up.

---

## Additional Notes

- **Inventory Files:** Use inventory files (e.g., `inventory.ini`) to manage hosts and groups. Variables defined here can be critical for host-specific configurations.
- **Configuration File:** `ansible.cfg` can be used to set global configurations for your Ansible environment.
- **Command-Line Variables:** Run playbooks with the command `ansible-playbook <playbook>.yaml` and pass extra variables using the `-e` flag when needed.
- **Variable Precedence:** Remember that variable precedence is key in Ansible; variables defined in the inventory file can override those defined in the playbook.
- **Security Practices:** Always secure sensitive data using Ansible Vault or environment variables to protect passwords and other sensitive information.

---

