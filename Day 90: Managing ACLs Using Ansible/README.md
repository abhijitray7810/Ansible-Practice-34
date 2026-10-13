# Ansible ACL Configuration Playbook 

## Overview  
This playbook configures files with specific ACL (Access Control List) permissions across multiple app servers in the Stratos DC environment. Files are owned by root but have additional permissions granted to specific users and groups via ACLs.

## Playbook Execution Results

The playbook successfully completed all tasks across three app servers:

### App Server 1 (stapp01)
- ✅ Created `/opt/dba/` directory
- ✅ Created `blog.txt` file owned by root
- ✅ Set ACL: group `tony` has read (r) permission

### App Server 2 (stapp02)
- ✅ Created `/opt/dba/` directory
- ✅ Created `story.txt` file owned by root
- ✅ Set ACL: user `steve` has read+write (rw) permissions

### App Server 3 (stapp03)
- ✅ Created `/opt/dba/` directory
- ✅ Created `media.txt` file owned by root
- ✅ Set ACL: group `banner` has read+write (rw) permissions

## Execution Summary

```
Total Tasks: 12 (4 per server)
Changed: 6 (2 per server - file creation and ACL setting)
Failed: 0
Unreachable: 0
```

## Usage

To run the playbook:

```bash
ansible-playbook -i inventory playbook.yml
```

## File Structure

```
/home/thor/ansible/
├── inventory          # Inventory file with app server definitions
├── playbook.yml       # Main playbook file
└── README.md          # This documentation file
```

## Playbook Components

### Tasks Performed on Each Server

1. **Directory Creation**: Ensures `/opt/dba/` directory exists with proper permissions
2. **File Creation**: Creates empty text files owned by root:root
3. **ACL Configuration**: Sets specific ACL permissions for designated users/groups

### ACL Permissions Summary

| Server   | File       | Entity Type | Entity Name | Permissions |
|----------|------------|-------------|-------------|-------------|
| stapp01  | blog.txt   | group       | tony        | read (r)    |
| stapp02  | story.txt  | user        | steve       | read+write (rw) |
| stapp03  | media.txt  | group       | banner      | read+write (rw) |

## Verification

To verify the ACL permissions were set correctly, connect to each server and run:

```bash
# On stapp01
getfacl /opt/dba/blog.txt

# On stapp02
getfacl /opt/dba/story.txt

# On stapp03
getfacl /opt/dba/media.txt
```

## Notes

- All files are owned by `root:root`
- ACLs provide additional permissions without changing the base ownership
- The playbook uses `become: yes` to execute tasks with elevated privileges
- The `acl` module requires the `acl` package to be installed on target systems
- Idempotent: Running the playbook multiple times will not cause issues

## Requirements

- Ansible 2.9 or higher
- Python ACL library on target hosts
- SSH access to all app servers
- Sudo privileges on app servers

## Author

Nautilus DevOps Team - Stratos DC

## Date

December 10, 2025
