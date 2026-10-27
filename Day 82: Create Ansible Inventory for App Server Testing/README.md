# Ansible Inventory Configuration for App Server 2

## Overview 
This README documents the Ansible inventory file configuration for App Server 2 (stapp02) in the Stratos DC environment, created for the Nautilus DevOps team's playbook testing.

## Inventory File Location
```
/home/thor/playbook/inventory
```

## Server Information
- **Server Name:** stapp02
- **Server Type:** App Server 2 in Stratos DC
- **IP Address:** 172.16.238.11  
- **Username:** steve
- **Password:** Am3ric@ 

## Inventory Configuration
The inventory file is configured in INI format with the following content:

```ini
stapp02 ansible_host=172.16.238.11 ansible_connection=ssh ansible_user=steve ansible_ssh_pass=Am3ric@
```

## Variables Explained
| Variable | Value | Description |
|----------|--------|-------------|
| `ansible_host` | 172.16.238.11 | The actual IP address of the target server |
| `ansible_connection` | ssh | Specifies SSH as the connection method |
| `ansible_user` | steve | The username for SSH connection |
| `ansible_ssh_pass` | Am3ric@ | The password for SSH authentication |

## Usage Instructions

### 1. Navigate to the playbook directory
```bash
cd /home/thor/playbook/
```

### 2. Create the inventory file
```bash
vi inventory
```

### 3. Copy the inventory configuration
```ini
stapp02 ansible_host=172.16.238.11 ansible_connection=ssh ansible_user=steve ansible_ssh_pass=Am3ric@
```

### 4. Save and exit the file

### 5. Test the connection
```bash
ansible -i inventory stapp02 -m ping
```

### 6. Run your playbook
```bash
ansible-playbook -i inventory playbook.yml
```

## Validation Requirements Met
- ✅ **INI format inventory file** created at `/home/thor/playbook/inventory`
- ✅ **App Server 2 included** with hostname `stapp02`
- ✅ **All necessary variables** configured for proper Ansible functionality
- ✅ **Hostname corresponds** to server name as per wiki (stapp02)
- ✅ **No extra arguments required** when running the playbook

## Troubleshooting
If connection fails:
1. Verify the inventory file syntax
2. Check network connectivity to 172.16.238.11
3. Ensure SSH service is running on the target server
4. Verify credentials are correct

## Notes
- This inventory is specifically for App Server 2 (stapp02)
- The inventory file follows Ansible's INI format standards
- All connection parameters are embedded in the inventory for seamless playbook execution
