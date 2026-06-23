# Ansible Conditional Playbook - File Distribution 

## Overview
This playbook demonstrates the use of Ansible's `when` conditionals to distribute different files to specific servers based on their node names. It's designed for the Stratos DC infrastructure to train team members on Ansible conditional logic.

## Directory Structure
```
/home/thor/ansible/
├── inventory          # Inventory file with all Stratos DC app servers
├── playbook.yml       # Main playbook with conditional tasks
└── README.md          # This file
```

## Source Files Location
Source files are located in `/usr/src/devops/` on the jump host:
- `blog.txt`
- `story.txt`
- `media.txt`

## Playbook Tasks

### Task 1: App Server 1 (stapp01)
- **File**: `blog.txt`
- **Destination**: `/opt/devops/blog.txt`
- **Owner/Group**: `tony`
- **Permissions**: `0655`
- **Condition**: `ansible_nodename == "stapp01"`

### Task 2: App Server 2 (stapp02)
- **File**: `story.txt`
- **Destination**: `/opt/devops/story.txt`
- **Owner/Group**: `steve`
- **Permissions**: `0655`
- **Condition**: `ansible_nodename == "stapp02"`

### Task 3: App Server 3 (stapp03)
- **File**: `media.txt`
- **Destination**: `/opt/devops/media.txt`
- **Owner/Group**: `banner`
- **Permissions**: `0655`
- **Condition**: `ansible_nodename == "stapp03"`

## How It Works

The playbook uses Ansible's `when` conditional to determine which task runs on which server:

```yaml
when: ansible_nodename == "stapp01"
```

This conditional checks the `ansible_nodename` fact (automatically gathered by Ansible) and only executes the task on the matching server. This approach ensures:
- Each server receives only its designated file
- Tasks are skipped on non-matching servers
- Efficient execution across all hosts

## Execution

### Run the playbook:
```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```

### Check syntax before running:
```bash
ansible-playbook -i inventory playbook.yml --syntax-check
```

### Dry run (check mode):
```bash
ansible-playbook -i inventory playbook.yml --check
```

### Verbose output for troubleshooting:
```bash
ansible-playbook -i inventory playbook.yml -v
# or for more details: -vv, -vvv, -vvvv
```

## Expected Output

When executed successfully, you should see output similar to:

```
PLAY [Copy files to app servers using conditionals] ***************************

TASK [Gathering Facts] *********************************************************
ok: [stapp01]
ok: [stapp02]
ok: [stapp03]

TASK [Copy blog.txt to App Server 1] *******************************************
changed: [stapp01]
skipping: [stapp02]
skipping: [stapp03]

TASK [Copy story.txt to App Server 2] ******************************************
skipping: [stapp01]
changed: [stapp02]
skipping: [stapp03]

TASK [Copy media.txt to App Server 3] ******************************************
skipping: [stapp01]
skipping: [stapp02]
changed: [stapp03]

PLAY RECAP *********************************************************************
stapp01  : ok=2  changed=1  unreachable=0  failed=0  skipped=2
stapp02  : ok=2  changed=1  unreachable=0  failed=0  skipped=2
stapp03  : ok=2  changed=1  unreachable=0  failed=0  skipped=2
```

## Verification

### Verify file presence and permissions on each server:

**On App Server 1:**
```bash
ssh tony@stapp01
ls -la /opt/devops/blog.txt
cat /opt/devops/blog.txt
```

**On App Server 2:**
```bash
ssh steve@stapp02
ls -la /opt/devops/story.txt
cat /opt/devops/story.txt
```

**On App Server 3:**
```bash
ssh banner@stapp03
ls -la /opt/devops/media.txt
cat /opt/devops/media.txt
```

Expected permissions format: `-rw-r-xr-x` (0655)

## Key Learning Points

### 1. Conditional Execution
- Use `when` to control task execution based on facts
- `ansible_nodename` provides the hostname of the target server
- Multiple conditions can be combined with `and`, `or` operators

### 2. Ansible Facts
- Facts are automatically gathered at the start of each play
- Common facts: `ansible_nodename`, `ansible_hostname`, `ansible_fqdn`, `ansible_os_family`
- Disable fact gathering with `gather_facts: no` if not needed

### 3. File Management
- `copy` module handles file transfers from control node to targets
- Automatically creates parent directories if they don't exist
- Can set ownership, group, and permissions in a single task

### 4. Privilege Escalation
- `become: yes` enables privilege escalation (sudo)
- Required for changing file ownership and system-level operations
- Can be set at play or task level

## Troubleshooting

### Issue: Tasks skip on all servers
**Solution**: Check that `ansible_nodename` matches your server hostnames
```bash
ansible -i inventory all -m setup -a "filter=ansible_nodename"
```

### Issue: Permission denied errors
**Solution**: Ensure `become: yes` is enabled and sudo access is configured

### Issue: Source file not found
**Solution**: Verify source files exist in `/usr/src/devops/` on jump host
```bash
ls -la /usr/src/devops/
```

### Issue: Connection errors
**Solution**: Test connectivity to all hosts
```bash
ansible -i inventory all -m ping
```

## Additional Resources

- [Ansible Documentation - Conditionals](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)
- [Ansible Documentation - Copy Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
- [Ansible Documentation - Facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)

## Best Practices Applied

1. ✅ Use meaningful task names
2. ✅ Quote octal permissions (0655)
3. ✅ Use `hosts: all` for broad execution
4. ✅ Leverage built-in facts instead of hardcoding values
5. ✅ Include both short and FQDN hostnames in conditionals for flexibility
6. ✅ Use `become` for privilege escalation
7. ✅ Keep playbooks idempotent (safe to run multiple times)

## Author
Nautilus DevOps Team

## Last Updated
December 2025
