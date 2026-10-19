# Ansible Configuration (`ansible.cfg`)

This guide explains the project configuration file `ansible.cfg` and breaks down each key configuration setting.

---

## 1. Configuration Snippet

```ini
[defaults]
inventory = ./inventory/hosts.ini
host_key_checking = True
retry_files_enabled = False
interpreter_python = auto_silent
```

> **Tip for Nano Editor users in terminal:**
> - Save file: Press `Ctrl + O`, then hit `Enter`.
> - Exit editor: Press `Ctrl + X`.

---

## 2. Understanding Each Setting

| Setting | Value | Purpose | Why It Matters |
| :--- | :--- | :--- | :--- |
| `inventory` | `./inventory/hosts.ini` | Tells Ansible where your server inventory is located. | Avoids passing `-i ./inventory/hosts.ini` every time you run `ansible` or `ansible-playbook` commands. |
| `host_key_checking` | `True` | Keeps SSH host-key verification enabled for security. | Prevents Man-in-the-Middle (MITM) attacks by verifying target server keys against your `~/.ssh/known_hosts`. |
| `retry_files_enabled` | `False` | Prevents Ansible from creating `.retry` files after failed playbook runs. | Keeps your workspace clean. By default, older Ansible versions dump `.retry` files whenever a host fails. |
| `interpreter_python` | `auto_silent` | Lets Ansible automatically detect the Python interpreter on remote hosts without warning prompts. | Different Linux distributions install Python in different paths (`/usr/bin/python3`, `/usr/bin/python`, etc.). `auto_silent` auto-detects Python silently and suppresses warning banners. |

---

## 3. How Ansible Finds `ansible.cfg` (Order of Precedence)

Ansible looks for its configuration file in the following priority order (first match wins):

1. **`ANSIBLE_CONFIG`**: An environment variable if set (`export ANSIBLE_CONFIG=/path/to/ansible.cfg`).
2. **`./ansible.cfg`**: The `ansible.cfg` file located in the **current working directory** (where you run the command).
3. **`~/.ansible.cfg`**: User-specific configuration in your home directory.
4. **`/etc/ansible/ansible.cfg`**: System-wide global default file.

> Placing `ansible.cfg` in your project repository allows team members and CI/CD pipelines to share identical configuration without modifying their local system settings.
