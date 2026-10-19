# Ansible Ad-Hoc Commands: Verifying Remote Services (`command` & `-b`)

This guide explains how to use Ansible ad-hoc commands to verify running services across your servers using privilege escalation (`-b`) and the `command` module (`-m command -a`).

---

## 1. The Commands Used

### 1.1 Check Apache Status on Web Servers
```bash
ansible webservers -b -m command -a "systemctl is-active apache2"
```

**Output:**
```text
web1 | CHANGED | rc=0 >>
active
```

---

### 1.2 Check MariaDB Status on Database Servers
```bash
ansible dbservers -b -m command -a "systemctl is-active mariadb"
```

**Output:**
```text
db1 | CHANGED | rc=0 >>
active
```

---

## 2. Command Anatomy Breakdown

| Flag / Argument | Long Form | Meaning & Purpose |
| :--- | :--- | :--- |
| **`webservers` / `dbservers`** | Target Host | The inventory group targeted by the command. |
| **`-b`** | `--become` | Runs the command with elevated privileges (**sudo / root**). Required for system services. |
| **`-m command`** | `--module-name` | Uses Ansible's `ansible.builtin.command` module to execute a binary on the remote server. |
| **`-a "..."`** | `--args` | The arguments passed to the module — in this case, the exact Linux command to run. |
| **`systemctl is-active <name>`** | Linux command | Queries `systemd` to check if a service is actively running. Returns string `active` and exit code `0` if running. |

---

## 3. Understanding the Terminal Output

```text
web1 | CHANGED | rc=0 >>
active
```

- **`rc=0`**: Return Code 0. In Linux, exit code `0` means the command completed successfully with no errors.
- **`active`**: The standard output (`stdout`) returned by `systemctl`. It confirms the service is up and running.
- **Why does it say `CHANGED`?**
  - Unlike declarative modules like `ansible.builtin.apt` or `service` (which know the exact system state), the generic `command` module cannot know whether an arbitrary command changed anything on disk.
  - By default, Ansible marks any `command` execution that succeeds as `CHANGED`.

---

## 4. Alternative: Checking Services Declaratively

While `systemctl is-active` works well for quick checks, Ansible also provides the `service_facts` module to inspect services declaratively without running raw shell commands:

```bash
ansible all -b -m ansible.builtin.service_facts
```

This returns a structured JSON dictionary of all services running on the machine.

---

## 5. Summary Cheat Sheet

| Task | Ad-Hoc Command |
| :--- | :--- |
| **Ping all hosts** | `ansible all -m ping` |
| **Check Apache status** | `ansible webservers -b -m command -a "systemctl is-active apache2"` |
| **Check MariaDB status** | `ansible dbservers -b -m command -a "systemctl is-active mariadb"` |
| **Check disk usage on all servers** | `ansible all -m command -a "df -h"` |
| **Check server uptime** | `ansible all -m command -a "uptime"` |
