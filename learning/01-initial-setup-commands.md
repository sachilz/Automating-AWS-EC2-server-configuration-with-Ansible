# Initial Project Setup Commands & Ansible Directory Structure

This document explains the commands used to bootstrap an Ansible project repository and details what each file/folder is used for.

---

## 1. Shell Commands Breakdown

### 1.1 `mkdir -p inventory playbooks`

- **Command**: `mkdir` stands for **Make Directory**.
- **Flag `-p` (`--parents`)**:
  - Creates parent directories as needed without failing if they already exist.
  - Suppresses error messages if the directory is already present.
  - Allows creating multiple directories in a single command (`inventory` and `playbooks`).
- **Purpose**: Creates the two primary directories needed for organizing an Ansible automation workspace.

---

### 1.2 `touch ansible.cfg`

- **Command**: `touch` updates the access and modification timestamp of a file. If the file does not exist, it creates a new empty file.
- **Purpose**: Creates the local Ansible configuration file in the project root.

---

### 1.3 `touch inventory/hosts.ini`

- **Command**: Creates an empty file named `hosts.ini` inside the `inventory/` folder.
- **Purpose**: Creates the default inventory file where target servers, IP addresses, and group definitions will be stored.

---

### 1.4 `touch playbooks/site.yml`

- **Command**: Creates an empty file named `site.yml` inside the `playbooks/` folder.
- **Purpose**: Creates the master playbook file (often called `site.yml` or `main.yml`), which orchestrates configuration tasks across target systems.

---

### 1.5 `ansible-inventory --list`

- **Command**: `ansible-inventory` with the `--list` flag.
- **Purpose**: Inspects and validates your inventory file, rendering all detected groups, hosts, and host variables in JSON format.
- **How it works**:
  - Reads the inventory path defined in `ansible.cfg` (`./inventory/hosts.ini`) or provided via `-i`.
  - Verifies syntax and parses all groups and nested hosts.
  - Useful for debugging inventory parsing errors and viewing the final variable resolution.
- **Example Output** (based on `hosts.ini` containing `[webservers] web1` and `[dbservers] db1`):
  ```json
  {
      "_meta": {
          "hostvars": {}
      },
      "all": {
          "children": [
              "ungrouped",
              "webservers",
              "dbservers"
          ]
      },
      "dbservers": {
          "hosts": [
              "db1"
          ]
      },
      "webservers": {
          "hosts": [
              "web1"
          ]
      }
  }
  ```

> **Pro Tip**: To view the inventory as an easy-to-read visual tree instead of JSON, use:
> ```bash
> ansible-inventory --graph
> ```
> Output:
> ```text
> @all:
>   |--@dbservers:
>   |  |--db1
>   |--@ungrouped:
>   |--@webservers:
>   |  |--web1
> ```

---

## 2. Ansible Architecture & Component Overview

| File / Folder | Role in Ansible | Description & Best Practices |
| :--- | :--- | :--- |
| **`ansible.cfg`** | Configuration File | Overrides system-wide Ansible settings (`/etc/ansible/ansible.cfg`). Commonly specifies defaults such as default inventory location, SSH settings, and privilege escalation (`become`). |
| **`inventory/`** | Inventory Directory | Contains target machine configurations. Can hold static files (`hosts.ini`, `hosts.yml`) or dynamic inventory scripts. |
| **`inventory/hosts.ini`** | Host Inventory File | INI-formatted file listing target machines, grouped logically (e.g. `[webservers]`, `[databases]`) along with connection variables like `ansible_host`, `ansible_user`, etc. |
| **`playbooks/`** | Playbooks Directory | Holds Ansible playbooks written in YAML. |
| **`playbooks/site.yml`** | Main Entry Playbook | The top-level playbook executed to apply configurations to the infrastructure (e.g., `ansible-playbook -i inventory/hosts.ini playbooks/site.yml`). |

---

## 3. Quick Reference: Recommended Basic Templates

### Basic `ansible.cfg` Example
```ini
[defaults]
inventory = ./inventory/hosts.ini
remote_user = ubuntu
host_key_checking = False
retry_files_enabled = False

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False
```

### Basic `inventory/hosts.ini` Example
```ini
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu

[dbservers]
db1 ansible_host=192.168.1.20 ansible_user=ubuntu

[all_servers:children]
webservers
dbservers
```

### Basic `playbooks/site.yml` Example
```yaml
---
- name: Test Connection to Remote Servers
  hosts: all
  gather_facts: true

  tasks:
    - name: Ping all hosts
      ansible.builtin.ping:
```
