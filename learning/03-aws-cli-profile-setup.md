# AWS CLI Configuration & Profile Management

When using Ansible to manage AWS resources (like EC2 instances, dynamic inventories, or security groups), you need properly configured AWS credentials. This document details the commands used to configure, switch, and verify named AWS profiles.

---

## 1. Commands Overview

| Command | Purpose |
| :--- | :--- |
| `aws configure --profile <name>` | Sets up or updates a named AWS profile with credentials and region. |
| `chmod 600 ~/.aws/credentials` | Secures the credentials file with read/write permissions for only your user. |
| `export AWS_PROFILE=<name>` | Sets the active profile for the current terminal session. |
| `aws sts get-caller-identity --profile <name>` | Checks IAM identity directly for a specific profile without changing environment variables. |
| `aws sts get-caller-identity` | Validates active credentials using whichever profile is currently exported/default. |
| `aws configure list-profiles` | Lists all AWS profiles configured on your machine. |

---

## 2. Command Details & Step-by-Step Breakdown

### 2.1 `aws configure --profile ansible`

Creates a distinct profile named `ansible` inside `~/.aws/credentials` and `~/.aws/config`, keeping it separate from your `default` or other project profiles (e.g., `terraform`).

**Interactive Prompts:**
```text
AWS Access Key ID [None]: <YOUR_ACCESS_KEY_ID>
AWS Secret Access Key [None]: <YOUR_SECRET_ACCESS_KEY>
Default region name [None]: us-east-1
Default output format [None]: json
```

> [!SECURITY]
> **Important**: Never commit credentials to Git or public repositories. Store them only in `~/.aws/credentials` or use AWS IAM roles and environment variables.

---

### 2.2 Securing Your AWS Credentials File: `chmod 600 ~/.aws/credentials`

If AWS CLI warns:
`[WARNING]: The file '/home/user/.aws/credentials' is accessible by other users. Consider running 'chmod 600 /home/user/.aws/credentials' to restrict access to only your user.`

Run:
```bash
chmod 600 ~/.aws/credentials
```

**Permission breakdown (`600`):**
- **First digit (`6`)**: Owner (you) gets Read (`4`) + Write (`2`) = `6`.
- **Second digit (`0`)**: Group gets `0` (no read, write, or execute).
- **Third digit (`0`)**: Others (all other users on the system) get `0` (no permissions at all).
- **Result**: `rw-------` — strictly private to your user account, resolving the security warning.

---

### 2.3 Option A: Checking Identity with `--profile` Flag
```bash
aws sts get-caller-identity --profile ansible
```
- **How it works**: Specifies the exact profile directly in the command call.
- **When to use**: Ideal for quick one-off checks or scripts where you do not want to alter the shell environment for subsequent commands.

**Example Output:**
```json
{
    "UserId": "AIDAUSDSIYAL45QGGO6Y4",
    "Account": "313772261399",
    "Arn": "arn:aws:iam::313772261399:user/ansible-devops-user"
}
```

---

### 2.4 Option B: Setting the Session Profile via Environment Variable
```bash
export AWS_PROFILE=ansible
aws sts get-caller-identity
```
- **How it works**: Sets the environment variable `AWS_PROFILE` in your current terminal session.
- **When to use**: Ideal for interactive terminal sessions or running Ansible playbooks and dynamic inventory plugins so you don't need to specify `--profile` repeatedly.

---

### 2.5 Comparing Both Approaches

| Approach | Command Example | Best Used For |
| :--- | :--- | :--- |
| **Direct Flag (`--profile`)** | `aws sts get-caller-identity --profile ansible` | One-off commands, non-interactive bash scripts, isolated verification. |
| **Session Variable (`export`)** | `export AWS_PROFILE=ansible` | Interactive sessions, Ansible CLI commands, `ansible-playbook` runs. |

---

### 2.6 `aws configure list-profiles`

- **Purpose**: Lists all profile names found in `~/.aws/credentials` and `~/.aws/config`.
- **Example Output**:
  ```text
  default
  terraform-devops
  ansible
  ```

---

## 3. How Ansible Uses AWS Profiles

When you write playbooks or configure dynamic inventory for AWS EC2, you can refer to this profile in two ways:

1. **Via Environment Variable** (already active when `export AWS_PROFILE=ansible` is run):
   Ansible and `boto3` automatically pick up the credentials from the environment.

2. **In Ansible Inventory / Playbook Variables**:
   ```yaml
   # Example in dynamic inventory or playbook tasks
   ansible.builtin.include_vars: ...
   # Or using plugin configuration:
   plugin: amazon.aws.aws_ec2
   boto_profile: ansible
   regions:
     - us-east-1
   ```
