# Troubleshooting SSH Connection Timeouts & AWS Security Groups

This guide documents a real-world troubleshooting case: solving `Connection timed out during banner exchange` when attempting to connect Ansible to an AWS EC2 instance.

---

## 1. The Error Encountered

```text
db1 | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: Connection timed out during banner exchange\r\nConnection to 44.194.255.219 port 22 timed out",
    "unreachable": true
}
```

---

## 2. Key Diagnostic Rule: Timeout vs Authentication Failure

| Error Message | Meaning | Where the Problem Is |
| :--- | :--- | :--- |
| `Permission denied (publickey)` | The server received your packet, but rejected your user or SSH `.pem` key. | SSH user, key path, or `chmod 600` permissions. |
| `Connection timed out` or `Connection timed out during banner exchange` | The network packet never reached port 22 or was silently dropped by a firewall before the OS could respond. | AWS Security Group, VPC Route Table, or Internet Gateway. |

---

## 3. The Root Cause Discovered

When checking the running instance using the AWS CLI:
```bash
aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=ansible-db-server" "Name=instance-state-name,Values=running" \
  --profile ansible \
  --query "Reservations[].Instances[][InstanceId,PublicIpAddress,SubnetId,SecurityGroups[].GroupId,SecurityGroups[].GroupName]" \
  --output table
```

**The Output Revealed:**
```text
| i-01e205735fda06ca5
| 44.194.255.219
| subnet-09a651c0cd1846c74
| sg-04f77fd5da9470eb4
| default
```

- The instance was attached to the **`default`** AWS security group (`sg-04f77fd5da9470eb4`).
- AWS's `default` security group **blocks all incoming public traffic** by default.
- Our custom security group `ansible-db-sg` (`sg-07ee0ee90e149eb6b`) with port 22 allowed had never been attached to the instance!

---

## 4. The Fix

### 4.1 Live Fix: Attaching the Security Group via AWS CLI
```bash
aws ec2 modify-instance-attribute \
  --instance-id i-01e205735fda06ca5 \
  --groups sg-07ee0ee90e149eb6b \
  --profile ansible
```
This instantly updated the instance's network interface to use `ansible-db-sg`.

---

### 4.2 Playbook Fix in `provision_db.yml`
In `amazon.aws.ec2_instance`, the `security_groups` attribute must be at the top level of the task, not nested under `network:`:

**Incorrect:**
```yaml
    - name: Launch database EC2 instance
      amazon.aws.ec2_instance:
        ...
        network:
          assign_public_ip: true
          security_groups:              # <-- Ignored under network
            - "{{ db_security_group.group_id }}"
```

**Correct:**
```yaml
    - name: Launch database EC2 instance
      amazon.aws.ec2_instance:
        ...
        security_groups:
          - "{{ db_security_group.group_id }}"
        network:
          assign_public_ip: true
```

---

## 5. Successful Verification

Once `ansible-db-sg` was attached, re-running the ad-hoc ping succeeded immediately:

```bash
ansible dbservers -m ping
```

**Result:**
```text
The authenticity of host '44.194.255.219 (44.194.255.219)' can't be established.
ED25519 key fingerprint is SHA256:j7vcX3156Pvh4+/qeNYeE8WsN472oDYktuGS1yPWB1Q.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
db1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

> **Note on Host Authenticity**:  
> The first time you connect to any new server via SSH, OpenSSH prompts you to confirm its fingerprint. Typing `yes` saves the key into `~/.ssh/known_hosts`.  
> *(To bypass this prompt automatically in Ansible automation, set `host_key_checking = False` in `ansible.cfg`)*.
