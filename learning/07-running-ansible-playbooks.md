# Running Ansible Playbooks & Syntax Validation

This guide explains how to validate and execute Ansible playbooks, how each task in `playbooks/site.yml` works, and how to read the execution output and `PLAY RECAP`.

---

## 1. The Core Commands

### 1.1 Check Playbook Syntax (Dry Check)
```bash
ansible-playbook playbooks/site.yml --syntax-check
```
- **Purpose**: Verifies that your YAML syntax and Ansible module options are well-formed without executing anything on the remote servers.
- **Expected Success Output**:
  ```text
  playbook: playbooks/site.yml
  ```
- **Why do it first**: Catches indentation errors, missing quotes, or typo-ridden module names before connecting to target servers.

---

### 1.2 Execute the Playbook
```bash
ansible-playbook playbooks/site.yml
```
- **Purpose**: Connects via SSH to target servers defined in `hosts: webservers` and applies each task in sequence.

---

## 2. Breakdown of `playbooks/site.yml`

```yaml
---
- name: Configure Apache Web Server
  hosts: webservers
  become: true
```

| Parameter | Meaning |
| :--- | :--- |
| `name` | Human-readable title describing what this play does. |
| `hosts: webservers` | Targets the `webservers` group defined in `inventory/hosts.ini`. |
| `become: true` | Runs tasks with elevated privileges (**sudo / root**). Required for installing software and modifying `/var/www/`. |

---

### Task-by-Task Explanation

#### Task 1: Update Apt Cache
```yaml
- name: Update apt package cache
  ansible.builtin.apt:
    update_cache: true
    cache_valid_time: 3600
```
- Equivalent to running `sudo apt update`.
- `cache_valid_time: 3600`: If the cache was updated within the last hour (3600 seconds), Ansible skips updating it again to save time.

#### Task 2: Install Apache
```yaml
- name: Install Apache HTTP Server
  ansible.builtin.apt:
    name: apache2
    state: present
```
- Equivalent to `sudo apt install -y apache2`.
- `state: present`: Ensures the package is installed. If already installed, Ansible takes no action.

#### Task 3: Create Custom Webpage
```yaml
- name: Create a custom homepage
  ansible.builtin.copy:
    dest: /var/www/html/index.html
    content: |
      <!DOCTYPE html>
      ...
    owner: root
    group: root
    mode: "0644"
```
- Writes the inline HTML content directly to `/var/www/html/index.html` on the remote server.
- `mode: "0644"` ensures public read access so the web server can serve the file.

#### Task 4: Start and Enable Apache Service
```yaml
- name: Ensure Apache is running and enabled
  ansible.builtin.service:
    name: apache2
    state: started
    enabled: true
```
- `state: started`: Equivalent to `systemctl start apache2`.
- `enabled: true`: Equivalent to `systemctl enable apache2` (automatically restarts Apache on system boot).

---

## 3. Comparing First Run vs Subsequent Runs (Idempotency in Action)

### First Run (Initial Configuration):
When you run the playbook on a fresh server, Ansible modifies the system to match the desired state:
```text
TASK [Gathering Facts] ****************************************************
ok: [web1]
TASK [Update apt package cache] *******************************************
changed: [web1]
TASK [Install Apache HTTP Server] *****************************************
changed: [web1]
TASK [Create a custom homepage] *******************************************
changed: [web1]
TASK [Ensure Apache is running and enabled] *******************************
ok: [web1]

PLAY RECAP ****************************************************************
web1                       : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---

### Subsequent Run (From Your Terminal Session):
When you re-run `ansible-playbook playbooks/site.yml`, all tasks report `ok`:
```text
TASK [Gathering Facts] ****************************************************
ok: [web1]
TASK [Update apt package cache] *******************************************
ok: [web1]
TASK [Install Apache HTTP Server] *****************************************
ok: [web1]
TASK [Create a custom homepage] *******************************************
ok: [web1]
TASK [Ensure Apache is running and enabled] *******************************
ok: [web1]

PLAY RECAP ****************************************************************
web1                       : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---

## 4. Understanding Every `PLAY RECAP` Value

The exact number of successful tasks may differ depending on how many tasks are in your playbook. Here is what every terminal metric means:

| Result / Metric | Terminal Value | Meaning & Explanation |
| :--- | :---: | :--- |
| **`ok`** | `5` | **Five tasks completed successfully**. Every task that either ran cleanly or found the machine already in the desired state increments this counter. |
| **`changed`** | `0` | **No configuration changes were needed**. The target server already had Apache installed, the homepage file matched exactly, and the service was already running. |
| **`unreachable`** | `0` | **Ansible connected successfully**. Zero target machines failed SSH handshake or credential authentication. |
| **`failed`** | `0` | **No tasks failed**. Zero tasks exited with an error code. |
| **`skipped`** | `0` | No tasks were bypassed (e.g., via `when:` conditionals). |
| **`rescued`** | `0` | No tasks required error-recovery rescue blocks (`block / rescue`). |
| **`ignored`** | `0` | No failed tasks were ignored (`ignore_errors: true`). |

---

## 5. Key Takeaway: Idempotency

> [!TIP]
> **Idempotency** is one of Ansible's greatest superpowers:  
> Running the same playbook multiple times produces the exact same outcome without causing unintended side effects.
>
> - **1st run**: Makes needed changes (`changed=3`).
> - **2nd+ runs**: Verifies state and leaves the system alone (`changed=0, ok=5`).

---

## 6. Helpful Playbook CLI Flags

| Flag | Command Example | What it does |
| :--- | :--- | :--- |
| **Syntax Check** | `ansible-playbook site.yml --syntax-check` | Tests YAML syntax without connecting. |
| **Check Mode (Dry Run)** | `ansible-playbook site.yml --check` | Simulates execution without making changes. |
| **Diff Mode** | `ansible-playbook site.yml --diff` | Shows file diffs before making changes. |
| **Limit Host** | `ansible-playbook site.yml -l web1` | Runs only on a specific server. |
