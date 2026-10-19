# Ansible Learning Notes & Documentation

Welcome to your Ansible learning folder. This directory tracks core concepts, commands, and workflow guides as you build and configure remote servers.

---

## Table of Contents

- [01. Initial Setup Commands & Project Structure](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/01-initial-setup-commands.md)
  - Explanation of `mkdir -p inventory playbooks`
  - Purpose of `ansible.cfg`
  - Purpose of `inventory/hosts.ini`
  - Purpose of `playbooks/site.yml`
  - Inspecting inventory with `ansible-inventory --list` and `--graph`
  - Template examples for each file
- [02. Ansible Configuration (`ansible.cfg`)](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/02-ansible-cfg-configuration.md)
  - Breakdown of key settings (`inventory`, `host_key_checking`, `retry_files_enabled`, `interpreter_python`)
  - Order of precedence for `ansible.cfg`
  - Nano editor quick save shortcuts
- [03. AWS CLI Configuration & Profile Management](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/03-aws-cli-profile-setup.md)
  - Configuring named AWS profiles (`aws configure --profile ansible`)
  - Securing credentials permissions (`chmod 600 ~/.aws/credentials`)
  - Session environment switching (`export AWS_PROFILE=ansible`)
  - Identity verification with STS (`aws sts get-caller-identity`)
  - Profile listing (`aws configure list-profiles`)
  - How Ansible integrates with AWS profiles
- [04. Python AWS SDK: `boto3` & `botocore`](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/04-aws-python-sdk-boto3.md)
  - Why Ansible requires `boto3` to communicate with AWS APIs
  - Difference between `boto3` and `botocore`
  - Installing via `sudo apt install -y python3-boto3 python3-botocore`
  - Verifying installation with `python3 -c "import boto3, botocore..."`
- [05. SSH Private Key Setup & Permissions [IMPORTANT]](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/05-ssh-private-key-setup.md)
  - Copying keys from Windows to native Linux/WSL (`~/.ssh/`)
  - Hardening key permissions with `chmod 600` (why OpenSSH enforces this)
  - Inspecting file modes with `ls -l` and validating with `test -f`
  - Connecting the key to Ansible (`ansible.cfg` & `hosts.ini`)
- [06. Ansible Ad-Hoc Commands: Testing Connectivity (`ping`)](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/06-ansible-ad-hoc-ping.md)
  - Running `ansible webservers -m ping`
  - Ansible `ping` vs standard ICMP ping
  - Understanding JSON response (`pong`) and `changed: false`
  - Inventory host variables (`ansible_host`, `ansible_user`, `ansible_python_interpreter`)
  - Troubleshooting connection & publickey errors
- [07. Running Ansible Playbooks & Syntax Validation](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/07-running-ansible-playbooks.md)
  - Validating syntax with `--syntax-check`
  - Running playbooks (`ansible-playbook playbooks/site.yml`)
  - Step-by-step breakdown of Apache web server deployment (`apt`, `copy`, `service`)
  - Reading the terminal output and `PLAY RECAP` (`ok`, `changed`, `failed`)
  - Understanding Ansible **Idempotency**
  - Useful flags (`--check`, `--diff`, `-l`)






