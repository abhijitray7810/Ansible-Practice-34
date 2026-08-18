# Ansible File Creation on Stratos DC App Servers 
 
## Overview 

This project contains Ansible automation scripts for the Nautilus DevOps team to create and manage files on remote app servers in the Stratos Data Center. The playbook creates a file with specific permissions and ownership on multiple servers simultaneously.

## Project Structure

```
~/playbook/
├── inventory          # Ansible inventory file with app server details
├── playbook.yml       # Ansible playbook for file creation
└── README.md          # This documentation file
```

## Requirements

- Ansible installed on the jump host
- SSH access to all app servers using the `ansible` user
- Sudo privileges on app servers
- Python installed on target servers

## Task Description

The playbook performs the following operations on all app servers:

1. Creates users if they don't exist on each server
2. Creates a blank file `/opt/code.txt` on all app servers
3. Sets file permissions to `0777` (rwxrwxrwx - full access)
4. Sets ownership based on the server:
   - **App Server 1 (stapp01)**: Owner = `tony`
   - **App Server 2 (stapp02)**: Owner = `steve`
   - **App Server 3 (stapp03)**: Owner = `banner`

## Quick Start

### Step 1: Create the Inventory File

```bash 
cat > ~/playbook/inventory << 'EOF'
[appservers]
stapp01 ansible_host=172.16.238.10 ansible_user=ansible ansible_password=ansible file_owner=tony
stapp02 ansible_host=172.16.238.11 ansible_user=ansible ansible_password=ansible file_owner=steve
stapp03 ansible_host=172.16.238.12 ansible_user=ansible ansible_password=ansible file_owner=banner

[appservers:vars]
ansible_become=yes
ansible_become_method=sudo
EOF
```

### Step 2: Create the Playbook

```bash
cat > ~/playbook/playbook.yml << 'EOF'
---
- name: Create code.txt file on all app servers
  hosts: appservers
  become: yes
  tasks:
    - name: Ensure user exists on each server
      user:
        name: "{{ file_owner }}"
        state: present

    - name: Create blank file /opt/code.txt with appropriate ownership and permissions
      file:
        path: /opt/code.txt
        state: touch
        mode: '0777'
        owner: "{{ file_owner }}"
        group: "{{ file_owner }}"
        modification_time: preserve
        access_time: preserve
EOF
```

### Step 3: Execute the Playbook

```bash
cd ~/playbook
ansible-playbook -i inventory playbook.yml
```

## Inventory File Details

### Server Configuration

```ini
[appservers]
stapp01 ansible_host=172.16.238.10 ansible_user=ansible ansible_password=ansible file_owner=tony
stapp02 ansible_host=172.16.238.11 ansible_user=ansible ansible_password=ansible file_owner=steve
stapp03 ansible_host=172.16.238.12 ansible_user=ansible ansible_password=ansible file_owner=banner
```

**Parameters:**
- `ansible_host`: IP address of the target server
- `ansible_user`: User account for SSH connection (ansible)
- `ansible_password`: Password for authentication
- `file_owner`: Custom variable defining the file owner for each server

### Group Variables

```ini
[appservers:vars]
ansible_become=yes
ansible_become_method=sudo
```

These settings enable privilege escalation using sudo on all app servers.

## Playbook Breakdown

### Task 1: Ensure User Exists

```yaml
- name: Ensure user exists on each server
  user:
    name: "{{ file_owner }}"
    state: present
```

**Purpose:** Creates the user account (tony/steve/banner) on each server if it doesn't already exist. This prevents "user not found" errors during file ownership assignment.

### Task 2: Create File with Permissions

```yaml
- name: Create blank file /opt/code.txt with appropriate ownership and permissions
  file:
    path: /opt/code.txt
    state: touch
    mode: '0777'
    owner: "{{ file_owner }}"
    group: "{{ file_owner }}"
    modification_time: preserve
    access_time: preserve
```

**Parameters:**
- `path`: Target file location (`/opt/code.txt`)
- `state: touch`: Creates a blank file (like the `touch` command)
- `mode: '0777'`: Sets full permissions (read/write/execute for owner, group, and others)
- `owner/group`: Uses the `file_owner` variable from inventory
- `modification_time/access_time: preserve`: Prevents timestamp updates on subsequent runs (idempotency)

## Usage Examples

### Basic Execution

```bash
ansible-playbook -i inventory playbook.yml
```

### Dry Run (Check Mode)

Test what changes will be made without actually applying them:

```bash
ansible-playbook -i inventory playbook.yml --check
```

### Verbose Output

Get detailed execution information:

```bash
ansible-playbook -i inventory playbook.yml -v
# or for even more detail
ansible-playbook -i inventory playbook.yml -vvv
```

### Run on Specific Server

Execute only on stapp01:

```bash
ansible-playbook -i inventory playbook.yml --limit stapp01
```

## Verification Commands

### Check Connectivity

Test connection to all app servers:

```bash
ansible -i inventory appservers -m ping
```

Expected output:
```
stapp01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Verify File Creation

Check if the file exists with correct permissions:

```bash
ansible -i inventory appservers -m shell -a "ls -l /opt/code.txt" --become
```

Expected output:
```
stapp01 | CHANGED | rc=0 >>
-rwxrwxrwx 1 tony tony 0 Dec 06 10:30 /opt/code.txt

stapp02 | CHANGED | rc=0 >>
-rwxrwxrwx 1 steve steve 0 Dec 06 10:30 /opt/code.txt

stapp03 | CHANGED | rc=0 >>
-rwxrwxrwx 1 banner banner 0 Dec 06 10:30 /opt/code.txt
```

### Verify User Creation

Check if users were created:

```bash
ansible -i inventory appservers -m shell -a "id {{ file_owner }}" --become
```

## Expected Output

Successful playbook execution will display:

```
PLAY [Create code.txt file on all app servers] ******************************************

TASK [Gathering Facts] ******************************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Ensure user exists on each server] ************************************************
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

TASK [Create blank file /opt/code.txt with appropriate ownership and permissions] *******
changed: [stapp01]
changed: [stapp02]
changed: [stapp03]

PLAY RECAP ******************************************************************************
stapp01                    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp02                    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
stapp03                    : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Troubleshooting

### Issue 1: "Failed to look up user"

**Error Message:**
```
fatal: [stapp01]: FAILED! => {"msg": "chown failed: failed to look up user tony"}
```

**Solution:** This is why we include the "Ensure user exists" task. If you still see this error, verify the user creation task is running:

```bash
ansible -i inventory appservers -m shell -a "getent passwd tony steve banner" --become
```

### Issue 2: Permission Denied

**Error Message:**
```
fatal: [stapp01]: FAILED! => {"msg": "Permission denied"}
```

**Solution:** Ensure the ansible user has sudo privileges:

```bash
# Test sudo access
ansible -i inventory appservers -m shell -a "sudo whoami"
```

### Issue 3: Connection Timeout

**Error Message:**
```
fatal: [stapp01]: UNREACHABLE!
```

**Solutions:**
1. Verify server IP addresses are correct
2. Check network connectivity:
   ```bash
   ping 172.16.238.10
   ```
3. Test SSH access manually:
   ```bash
   ssh ansible@172.16.238.10
   ```

### Issue 4: Authentication Failure

**Error Message:**
```
fatal: [stapp01]: FAILED! => {"msg": "Authentication or permission failure"}
```

**Solution:** Verify credentials in inventory file or use SSH keys instead:

```bash
# Generate SSH key
ssh-keygen -t rsa -b 4096

# Copy to servers
ssh-copy-id ansible@172.16.238.10
ssh-copy-id ansible@172.16.238.11
ssh-copy-id ansible@172.16.238.12

# Update inventory to remove passwords
```

### Issue 5: File Already Exists

The playbook is **idempotent** - running it multiple times is safe. The `preserve` parameters ensure timestamps aren't modified if the file already exists.

## Security Considerations

### Password Storage

**Current Setup:** Passwords are stored in plain text in the inventory file.

**Better Approach:** Use Ansible Vault to encrypt sensitive data:

```bash
# Create encrypted inventory
ansible-vault create inventory_encrypted

# Edit encrypted inventory
ansible-vault edit inventory_encrypted

# Run playbook with vault
ansible-playbook -i inventory_encrypted playbook.yml --ask-vault-pass
```

### File Permissions (0777)

**Warning:** `0777` permissions grant full access to everyone. This is suitable for testing but should be reconsidered for production:

- **0644**: Owner can read/write, others can only read
- **0755**: Owner can read/write/execute, others can read/execute
- **0700**: Only owner has full access

Update the playbook's `mode` parameter as needed for your security requirements.

## Advanced Usage

### Using Host Variables

For more complex scenarios, create host-specific variable files:

```bash
mkdir -p ~/playbook/host_vars

cat > ~/playbook/host_vars/stapp01.yml << 'EOF'
file_owner: tony
file_permissions: '0777'
EOF
```

### Using Group Variables

For settings shared across all servers:

```bash
mkdir -p ~/playbook/group_vars

cat > ~/playbook/group_vars/appservers.yml << 'EOF'
file_path: /opt/code.txt
ansible_become: yes
EOF
```

### Creating Multiple Files

Extend the playbook to create multiple files:

```yaml
- name: Create multiple files
  file:
    path: "{{ item }}"
    state: touch
    mode: '0777'
    owner: "{{ file_owner }}"
    group: "{{ file_owner }}"
  loop:
    - /opt/code.txt
    - /opt/data.txt
    - /opt/config.txt
```

## Best Practices

1. **Always test with --check first**: Preview changes before applying
2. **Use version control**: Store playbooks in Git
3. **Document changes**: Add comments explaining complex logic
4. **Use descriptive task names**: Make output easy to understand
5. **Implement error handling**: Use `ignore_errors` or `block/rescue` when appropriate
6. **Keep playbooks simple**: Break complex tasks into separate playbooks
7. **Use roles**: For larger projects, organize tasks into reusable roles

## Related Commands

### List All Hosts

```bash
ansible -i inventory appservers --list-hosts
```

### Gather Facts

```bash
ansible -i inventory appservers -m setup
```

### Execute Ad-Hoc Commands

```bash
# Check disk space
ansible -i inventory appservers -m shell -a "df -h /opt" --become

# View file contents
ansible -i inventory appservers -m shell -a "cat /opt/code.txt" --become

# Remove the file
ansible -i inventory appservers -m file -a "path=/opt/code.txt state=absent" --become
```

## Validation

The DevOps team will validate this playbook using:

```bash
ansible-playbook -i inventory playbook.yml
```

Ensure your playbook:
- ✅ Runs without additional arguments
- ✅ Creates `/opt/code.txt` on all servers
- ✅ Sets permissions to `0777`
- ✅ Sets correct ownership (tony/steve/banner)
- ✅ Completes without errors

## Additional Resources

- [Ansible File Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
- [Ansible User Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html)
- [Ansible Inventory Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Ansible Vault Guide](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

## Support

For issues or questions:
1. Check the troubleshooting section above
2. Review Ansible logs: `~/.ansible.log`
3. Contact the Nautilus DevOps team
4. Submit feedback via the project repository

## Changelog

### Version 1.0 (Current)
- Initial release
- Creates `/opt/code.txt` with 0777 permissions
- Implements per-server ownership (tony/steve/banner)
- Includes user creation task to prevent lookup failures

## Author

**Nautilus DevOps Team**  
Stratos Data Center Operations

## License

Internal use only - Stratos DC Operations
