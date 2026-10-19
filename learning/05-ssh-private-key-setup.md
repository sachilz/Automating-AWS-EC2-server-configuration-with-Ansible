# SSH Private Key Setup & Permissions [IMPORTANT]

When connecting to remote Linux servers (such as AWS EC2 instances) using SSH or Ansible, managing your SSH private key (`.pem`) and setting the correct file permissions is **mandatory**.

> [!IMPORTANT]
> **SSH Security Rule**: OpenSSH will **refuse to connect** if your private key permissions are too loose. If others can read your key, SSH rejects it with:  
> `Permissions 0644 for 'key.pem' are too open. This private key will be ignored. Load key 'key.pem': Permission denied (publickey).`

---

## 1. Commands Overview

| Step | Command | Purpose |
| :--- | :--- | :--- |
| **1** | `mkdir -p ~/.ssh` | Ensures the user's hidden SSH directory exists. |
| **2** | `cp /mnt/c/Users/.../Downloads/ansible-lab-key.pem ~/.ssh/` | Copies the `.pem` key from Windows Downloads into Linux/WSL. |
| **3** | `chmod 600 ~/.ssh/ansible-lab-key.pem` | **(CRITICAL)** Restricts permissions so only your user can read the key. |
| **4** | `ls -l ~/.ssh/ansible-lab-key.pem` | Verifies the file permissions and attributes. |
| **5** | `test -f ~/.ssh/... && echo "..."` | Automated check confirming the file exists. |

---

## 2. Step-by-Step Command Breakdown

### 2.1 Step 1: `mkdir -p ~/.ssh`
- **`~`**: Shortcut for your user's Linux home directory (`/home/sachintha`).
- **`.ssh`**: Standard hidden directory where SSH looks for keys and configuration.
- **`-p` flag**: Creates the folder if it doesn't exist, and does nothing (no error) if it already does.

---

### 2.2 Step 2: Copy Key from Windows to WSL
```bash
cp /mnt/c/Users/sachintha/Downloads/ansible-lab-key.pem ~/.ssh/
```
- **Why copy it into Linux/WSL?**
  - Files stored on Windows drives (`/mnt/c/...`) use Windows NTFS permissions, which do not properly enforce Linux POSIX permissions.
  - Moving the key into native Linux storage (`~/.ssh/`) allows Linux `chmod` permissions to work correctly.

---

### 2.3 Step 3: `chmod 600 ~/.ssh/ansible-lab-key.pem` (CRITICAL)

This command sets strict POSIX file permissions:

```text
Permission Code: 600
  ├── 6 (Owner)  : Read (4) + Write (2) = 6
  ├── 0 (Group)  : No permissions = 0
  └── 0 (Others) : No permissions = 0
```

- **Output mode**: `-rw-------`
- **Result**: You are the only person on the system who can open and read this key. This satisfies SSH's strict security requirements.

---

### 2.4 Step 4: `ls -l ~/.ssh/ansible-lab-key.pem`

- **`-l` flag**: Long listing format.
- **Terminal Output**:
  ```text
  -rw------- 1 sachintha sachintha 1674 Oct 19 18:44 /home/sachintha/.ssh/ansible-lab-key.pem
  ```
- **What this verifies**:
  - `-rw-------`: Exact `600` permission mode confirmed.
  - `sachintha`: Correct user owner and group owner.
  - `1674`: File size in bytes (confirms key content is intact).

---

### 2.5 Step 5: Validate Key Existence
```bash
test -f ~/.ssh/ansible-lab-key.pem && echo "Private key copied successfully"
```
- **`test -f <path>`**: Shell test command that checks if the specified path exists and is a regular file.
- **`&&` (AND operator)**: Only executes the next command if `test -f` succeeded (exit code 0).
- **`echo "..."`**: Prints a confirmation message if the test succeeded.

---

## 3. How to Use This Key in Ansible

Once your key is stored at `~/.ssh/ansible-lab-key.pem`, you can configure Ansible to use it in two clean ways:

### Method A: In `ansible.cfg` (Recommended for the whole project)
Add `private_key_file` under `[defaults]`:
```ini
[defaults]
inventory = ./inventory/hosts.ini
private_key_file = ~/.ssh/ansible-lab-key.pem
```

### Method B: In `inventory/hosts.ini` (Host or Group specific)
```ini
[webservers]
web1 ansible_host=54.x.x.x ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/ansible-lab-key.pem
```
