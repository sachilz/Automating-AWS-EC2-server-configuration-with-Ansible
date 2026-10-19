# Automated Remote Server Configuration with Ansible & AWS

[![Ansible](https://img.shields.io/badge/Ansible-2.15+-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20Security%20Groups-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Python](https://img.shields.io/badge/Python-Boto3%20%7C%20Botocore-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)

An enterprise-ready Infrastructure-as-Code (IaC) and configuration management project. This repository demonstrates automated cloud provisioning on AWS, secure SSH authentication, and modular Ansible playbook execution using roles to deploy an Apache web server tier and a MariaDB database tier.

---

## Architecture Overview

![Ansible Remote Server Configuration on AWS EC2](Ansible%20AWS%20EC2%20Automation%20Architecture%20Diagram.png)

---

## Project Structure

```text
ansible-remote-server-configuration/
├── Ansible AWS EC2 Automation Architecture Diagram.png
├── ansible.cfg                    # Project-level Ansible configuration & roles path
├── inventory/
│   └── hosts.ini                  # Managed server definitions & host/group variables
├── playbooks/
│   ├── site.yml                   # Master orchestration playbook (calls roles)
│   └── provision_db.yml           # AWS EC2 & Security Group provisioning playbook
├── roles/
│   ├── webserver/                 # Modular Web Tier role
│   │   └── tasks/
│   │       └── main.yml           # Apache installation, webpage copy & service startup
│   └── database/                  # Modular Database Tier role
│       └── tasks/
│           └── main.yml           # MariaDB installation & service startup
├── learning/                      # Detailed technical guides & troubleshooting notes
│   ├── README.md                  # Learning Table of Contents & index
│   ├── 01-initial-setup-commands.md
│   ├── 02-ansible-cfg-configuration.md
│   ├── 03-aws-cli-profile-setup.md
│   ├── 04-aws-python-sdk-boto3.md
│   ├── 05-ssh-private-key-setup.md
│   ├── 06-ansible-ad-hoc-ping.md
│   ├── 07-running-ansible-playbooks.md
│   ├── 08-troubleshooting-ssh-timeouts-and-security-groups.md
│   ├── 09-ansible-ad-hoc-service-verification.md
│   └── 10-ansible-roles-and-roles-path.md
└── README.md                      # Project documentation (this file)
```

---

## Prerequisites & Initial Setup

### 1. Control Machine Requirements
- Linux or WSL (Windows Subsystem for Linux) with Ubuntu.
- Ansible installed:
  ```bash
  sudo apt update && sudo apt install -y ansible
  ```
- Python AWS SDK (`boto3` & `botocore`) for AWS module automation:
  ```bash
  sudo apt install -y python3-boto3 python3-botocore
  ```

### 2. AWS Credentials Configuration
Configure a named profile for Ansible:
```bash
aws configure --profile ansible
chmod 600 ~/.aws/credentials
export AWS_PROFILE=ansible
aws sts get-caller-identity
```

### 3. SSH Key Hardening
Place your AWS EC2 `.pem` private key in `~/.ssh/` with strict permissions:
```bash
mkdir -p ~/.ssh
chmod 600 ~/.ssh/ansible-lab-key.pem
```

---

## Configuration Details

### `ansible.cfg`
Defines project defaults so you do not need to pass manual CLI flags:
```ini
[defaults]
inventory = ./inventory/hosts.ini
roles_path = ./roles
host_key_checking = True
retry_files_enabled = False
interpreter_python = auto_silent
```

### `inventory/hosts.ini`
Maps servers into functional groups and declares connection parameters:
```ini
[webservers]
web1 ansible_host=54.146.224.105 ansible_user=ubuntu ansible_ssh_private_key_file=/home/sachintha/.ssh/ansible-lab-key.pem

[dbservers]
db1 ansible_host=44.194.255.219 ansible_user=ubuntu ansible_ssh_private_key_file=/home/sachintha/.ssh/ansible-lab-key.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3

[dbservers:vars]
ansible_python_interpreter=/usr/bin/python3
```

---

## Operational Workflows

### 1. Connectivity Testing (Ad-Hoc Commands)
Verify SSH connectivity and remote Python interpreter readiness across all tiers:
```bash
ansible webservers -m ping
ansible dbservers -m ping
```
*Expected output: `"ping": "pong"`.*

### 2. Playbook Syntax Check (Dry Run)
Validate playbook syntax before making changes:
```bash
ansible-playbook playbooks/site.yml --syntax-check
```

### 3. Master Playbook Execution
Run the full multi-tier deployment across Web and Database nodes:
```bash
ansible-playbook playbooks/site.yml
```

### 4. Service Verification
Confirm services are live and managed by `systemd`:
```bash
# Check Apache on Web tier
ansible webservers -b -m command -a "systemctl is-active apache2"

# Check MariaDB on Database tier
ansible dbservers -b -m command -a "systemctl is-active mariadb"
```
*Expected output: `active` with return code `rc=0`.*

---

## Key Concepts Demonstrated

- **Ansible Roles**: Clean modular separation of concerns (`roles/webserver`, `roles/database`).
- **Idempotency**: Running `site.yml` repeatedly leaves unchanged services untouched (`changed=0`, `ok=5`).
- **Privilege Escalation (`become: true` / `-b`)**: Automated `sudo` execution for package management and system services.
- **AWS Cloud Provisioning**: Automated creation of EC2 Security Groups and instances via `amazon.aws.ec2_instance`.
- **Infrastructure Security**: POSIX permission hardening (`600` for keys/credentials), discrete security group definitions, and SSH key verification.

---

## Detailed Step-by-Step Learning Notes

Comprehensive step-by-step documentation, command references, and troubleshooting guides are available in the [`learning/`](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/README.md) directory:

- [01. Initial Setup Commands & Project Structure](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/01-initial-setup-commands.md)
- [02. Ansible Configuration (`ansible.cfg`)](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/02-ansible-cfg-configuration.md)
- [03. AWS CLI Configuration & Profile Management](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/03-aws-cli-profile-setup.md)
- [04. Python AWS SDK: `boto3` & `botocore`](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/04-aws-python-sdk-boto3.md)
- [05. SSH Private Key Setup & Permissions [IMPORTANT]](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/05-ssh-private-key-setup.md)
- [06. Ansible Ad-Hoc Commands: Testing Connectivity (`ping`)](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/06-ansible-ad-hoc-ping.md)
- [07. Running Ansible Playbooks & Syntax Validation](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/07-running-ansible-playbooks.md)
- [08. Troubleshooting SSH Timeouts & AWS Security Groups](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/08-troubleshooting-ssh-timeouts-and-security-groups.md)
- [09. Verifying Remote Services with Ad-Hoc Commands](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/09-ansible-ad-hoc-service-verification.md)
- [10. Organizing Playbooks with Ansible Roles & `roles_path`](file:///c:/Users/sachintha/Desktop/ansible-remote-server-configuration/learning/10-ansible-roles-and-roles-path.md)