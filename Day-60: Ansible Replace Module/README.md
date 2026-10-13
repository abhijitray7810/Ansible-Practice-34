# Ansible Text Replacement Playbook

## Overview
This Ansible playbook performs text replacement operations across three app servers in the Stratos DC environment. It uses the `replace` module to update specific strings in finance-related text files.

## Directory Structure
```
/home/thor/ansible/
├── inventory
├── playbook.yml
└── README.md 
```

## Prerequisites
- Ansible installed on the jump host
- SSH access to all app servers (stapp01, stapp02, stapp03)
- Inventory file already configured at `/home/thor/ansible/inventory`
- Sudo/root privileges on target servers

## Playbook Details

### Tasks Performed

#### App Server 1 (stapp01)
- **File**: `/opt/finance/blog.txt`
- **Action**: Replace `xFusionCorp` → `Nautilus`

#### App Server 2 (stapp02)
- **File**: `/opt/finance/story.txt`
- **Action**: Replace `Nautilus` → `KodeKloud`

#### App Server 3 (stapp03)
- **File**: `/opt/finance/media.txt`
- **Action**: Replace `KodeKloud` → `xFusionCorp Industries`

## Usage

### Execute the Playbook
From the `/home/thor/ansible` directory on the jump host, run:

```bash
ansible-playbook -i inventory playbook.yml
```

### Check Syntax (Optional)
Before running, you can verify the playbook syntax:

```bash
ansible-playbook --syntax-check -i inventory playbook.yml
```

### Dry Run (Optional)
To see what changes would be made without actually applying them:

```bash
ansible-playbook -i inventory playbook.yml --check
```

### Verbose Output (Optional)
For detailed execution information:

```bash
ansible-playbook -i inventory playbook.yml -v
```

## Expected Output
When executed successfully, you should see output similar to:

```
PLAY [Update blog.txt on app server 1] *************************************

TASK [Gathering Facts] *****************************************************
ok: [stapp01]

TASK [Replace xFusionCorp with Nautilus in blog.txt] ***********************
changed: [stapp01]

PLAY [Update story.txt on app server 2] ************************************

TASK [Gathering Facts] *****************************************************
ok: [stapp02]

TASK [Replace Nautilus with KodeKloud in story.txt] ************************
changed: [stapp02]

PLAY [Update media.txt on app server 3] ************************************

TASK [Gathering Facts] *****************************************************
ok: [stapp03]

TASK [Replace KodeKloud with xFusionCorp Industries in media.txt] **********
changed: [stapp03]

PLAY RECAP *****************************************************************
stapp01                    : ok=2    changed=1    unreachable=0    failed=0
stapp02                    : ok=2    changed=1    unreachable=0    failed=0
stapp03                    : ok=2    changed=1    unreachable=0    failed=0
```

## Verification

### Manual Verification
After running the playbook, you can verify the changes by connecting to each server and checking the files:

```bash
# On stapp01
ssh stapp01
cat /opt/finance/blog.txt

# On stapp02
ssh stapp02
cat /opt/finance/story.txt

# On stapp03
ssh stapp03
cat /opt/finance/media.txt
```

### Using Ansible Ad-hoc Commands
```bash
# Check blog.txt on stapp01
ansible stapp01 -i inventory -m shell -a "cat /opt/finance/blog.txt" -b

# Check story.txt on stapp02
ansible stapp02 -i inventory -m shell -a "cat /opt/finance/story.txt" -b

# Check media.txt on stapp03
ansible stapp03 -i inventory -m shell -a "cat /opt/finance/media.txt" -b
```

## Troubleshooting

### Common Issues

**Permission Denied Errors**
- Ensure the playbook has `become: yes` enabled
- Verify SSH keys are properly configured
- Check sudo privileges on target servers

**File Not Found Errors**
- Verify the files exist at the specified paths
- Check file permissions on target servers

**Connection Issues**
- Test connectivity: `ansible all -i inventory -m ping`
- Verify inventory file configuration
- Check SSH access to all app servers

**No Changes Made**
- The replace module is idempotent - if the string doesn't exist, no changes are made
- Verify the source strings exist in the respective files

## Module Information

The playbook uses the `ansible.builtin.replace` module with the following parameters:
- **path**: The file to modify
- **regexp**: Regular expression pattern to search for
- **replace**: The replacement string

## Notes
- The playbook is idempotent - running it multiple times will only make changes if the original strings are present
- All operations require elevated privileges (`become: yes`)
- The playbook runs sequentially across the three app servers

## Author
DevOps Team - Nautilus Development

## Last Updated
October 12, 20
