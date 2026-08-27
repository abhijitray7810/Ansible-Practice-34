# Ansible Conditional Playbook - File Distribution 

## Overview
This playbook demonstrates the use of Ansible conditional statements (`when`) to distribute different files to different application servers based on their hostnames. It's designed for the Stratos DC environment with three app servers.

## Prerequisites
- Ansible installed on jump host
- Inventory file present at `/home/thor/ansible/inventory`
- Source files located in `/usr/src/itadmin/` directory:
  - `blog.txt`
  - `story.txt`
  - `media.txt`
- SSH access to all app servers configured
- Sudo privileges on target servers

## Directory Structure
```
/home/thor/ansible/
├── inventory
├── playbook.yml
└── README.md

/usr/src/itadmin/
├── blog.txt
├── story.txt
└── media.txt
```

## Playbook Details

### File Distribution Matrix

| App Server | Hostname | Source File | Destination | Owner:Group | Permissions |
|------------|----------|-------------|-------------|-------------|-------------|
| App Server 1 | stapp01 | blog.txt | /opt/itadmin/blog.txt | tony:tony | 0777 |
| App Server 2 | stapp02 | story.txt | /opt/itadmin/story.txt | steve:steve | 0777 |
| App Server 3 | stapp03 | media.txt | /opt/itadmin/media.txt | banner:banner | 0777 |

### Key Features
- **Conditional Execution**: Uses `when` clause with `ansible_nodename` variable
- **Fact Gathering**: Automatically collects system facts to determine hostname
- **Privilege Escalation**: Uses `become: yes` for sudo access
- **Idempotent**: Safe to run multiple times

## Usage

### Basic Execution
```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```

### Dry Run (Check Mode)
```bash
ansible-playbook -i inventory playbook.yml --check
```

### Verbose Output
```bash
ansible-playbook -i inventory playbook.yml -v
ansible-playbook -i inventory playbook.yml -vv   # More verbose
ansible-playbook -i inventory playbook.yml -vvv  # Very verbose
```

### Limit to Specific Host
```bash
ansible-playbook -i inventory playbook.yml --limit stapp01
```

## How Conditionals Work

The playbook uses the `ansible_nodename` fact to determine which tasks to execute on each server:

```yaml
when: ansible_nodename == "stapp01" or ansible_nodename == "stapp01.stratos.xfusioncorp.com"
```

This ensures:
- Each server only receives its designated file
- Tasks skip automatically if the condition doesn't match
- Works with both short hostnames and FQDNs

## Verification

### Check Files Were Copied
```bash
# On App Server 1
ansible stapp01 -i inventory -m shell -a "ls -la /opt/itadmin/blog.txt"

# On App Server 2
ansible stapp02 -i inventory -m shell -a "ls -la /opt/itadmin/story.txt"

# On App Server 3
ansible stapp03 -i inventory -m shell -a "ls -la /opt/itadmin/media.txt"
```

### Verify All at Once
```bash
ansible all -i inventory -m shell -a "ls -la /opt/itadmin/"
```

## Expected Output
When running the playbook successfully, you should see:
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
stapp01    : ok=2    changed=1    unreachable=0    failed=0    skipped=2
stapp02    : ok=2    changed=1    unreachable=0    failed=0    skipped=2
stapp03    : ok=2    changed=1    unreachable=0    failed=0    skipped=2
```

## Troubleshooting

### Issue: Permission Denied
**Solution**: Ensure you have sudo access configured and the playbook uses `become: yes`

### Issue: Source File Not Found
**Solution**: Verify files exist in `/usr/src/itadmin/` on jump host:
```bash
ls -la /usr/src/itadmin/
```

### Issue: Destination Directory Doesn't Exist
**Solution**: Add a task to create the directory first:
```yaml
- name: Ensure destination directory exists
  file:
    path: /opt/itadmin
    state: directory
    mode: '0755'
```

### Issue: Wrong Hostname
**Solution**: Check the actual hostname with:
```bash
ansible all -i inventory -m setup -a "filter=ansible_nodename"
```

## Learning Points

This playbook demonstrates:
1. **Conditional Execution**: Using `when` clause for task-level conditionals
2. **Fact Variables**: Leveraging `ansible_nodename` from gathered facts
3. **File Management**: Copy module with ownership and permissions
4. **Best Practices**: Running plays against all hosts with targeted execution

## Additional Resources
- [Ansible Conditionals Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_conditionals.html)
- [Ansible Copy Module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)
- [Ansible Facts](https://docs.ansible.com/ansible/latest/user_guide/playbooks_vars_facts.html)

## Author
Nautilus DevOps Team

## Last Updated
December 2025
