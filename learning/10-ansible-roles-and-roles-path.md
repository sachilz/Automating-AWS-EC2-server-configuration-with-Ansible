# Organizing Playbooks with Ansible Roles & `roles_path`

As playbooks grow in size and complexity, keeping all tasks in a single `site.yml` file becomes messy. **Ansible Roles** allow you to group tasks, files, templates, and variables into clean, reusable components.

---

## 1. Why Use Ansible Roles?

- **Modularity**: Keeps individual components (e.g. `webserver` and `database`) isolated in their own folders.
- **Reusability**: You can share and reuse roles across different projects and environments.
- **Clean Playbooks**: Your main entry playbook (`site.yml`) becomes simple and readable:
  ```yaml
  - name: Configure Apache Web Server
    hosts: webservers
    become: true
    roles:
      - webserver

  - name: Configure MariaDB Database Server
    hosts: dbservers
    become: true
    roles:
      - database
  ```

---

## 2. Standard Role Directory Structure

```text
roles/
├── webserver/
│   └── tasks/
│       └── main.yml        # All Apache installation & deployment tasks
└── database/
    └── tasks/
        └── main.yml        # All MariaDB installation & service tasks
```

---

## 3. The "Role Not Found" Error & Why `roles_path` is Required

### The Problem:
When you run:
```bash
ansible-playbook playbooks/site.yml
```
Ansible looks for roles **relative to where the playbook is located**:
- Looks in: `./playbooks/roles/`
- Looks in: `~/.ansible/roles`
- Looks in: `/etc/ansible/roles`

Because our `roles/` folder is at the **project root** (`./roles`), Ansible cannot find `webserver` and fails with:
```text
ERROR! the role 'webserver' was not found in /mnt/.../playbooks/roles:...
```

### The Solution: `roles_path` in `ansible.cfg`
Add `roles_path = ./roles` inside [ansible.cfg](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/ansible.cfg):

```ini
[defaults]
inventory = ./inventory/hosts.ini
roles_path = ./roles
host_key_checking = True
retry_files_enabled = False
interpreter_python = auto_silent
```

This tells Ansible: *"Whenever a playbook asks for a role, look inside the `./roles` directory at the project root."*

---

## 4. Verification & Execution

### Step 1: Dry Syntax Validation
```bash
ansible-playbook playbooks/site.yml --syntax-check
```
**Expected Output:**
```text
playbook: playbooks/site.yml
```

### Step 2: Execute the Modular Playbook
```bash
ansible-playbook playbooks/site.yml
```
Ansible will execute the `webserver` role on `web1` and the `database` role on `db1`.
