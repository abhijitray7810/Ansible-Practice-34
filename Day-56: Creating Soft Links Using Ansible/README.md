# Ansible Symbolic Links Playbook

## 📋 Overview
This Ansible playbook creates empty files with specific ownership and symbolic links on all app servers in the Nautilus infrastructure.

## 🎯 Objectives
- Create `/opt/itadmin/` directory on all app servers
- Create specific empty files with designated owners on each server
- Create symbolic links from `/opt/itadmin` to `/var/www/html`

## 📁 Project Structure
```
/home/thor/ansible/
├── README.md          # This file
├── playbook.yml       # Main Ansible playbook
├── inventory          # Inventory file with server details
├── command.md         # Command reference guide
└── ansible.cfg        # Ansible configuration (if present)
``` 

## 🖥️ Infrastructure Details

### App Servers
| Server | Hostname | IP Address | User | File Created | Owner |
|--------|----------|------------|------|--------------|-------|
| App Server 1 | stapp01 | 172.16.238.10 | tony | blog.txt | tony:tony |
| App Server 2 | stapp02 | 172.16.238.11 | steve | story.txt | steve:steve |
| App Server 3 | stapp03 | 172.16.238.12 | banner | media.txt | banner:banner |

## 📝 Tasks Performed

### App Server 1 (stapp01)
1. Create directory `/opt/itadmin/`
2. Create empty file `/opt/itadmin/blog.txt` with owner `tony:tony`
3. Create symbolic link: `/var/www/html` → `/opt/itadmin`

### App Server 2 (stapp02)
1. Create directory `/opt/itadmin/`
2. Create empty file `/opt/itadmin/story.txt` with owner `steve:steve`
3. Create symbolic link: `/var/www/html` → `/opt/itadmin`

### App Server 3 (stapp03)
1. Create directory `/opt/itadmin/`
2. Create empty file `/opt/itadmin/media.txt` with owner `banner:banner`
3. Create symbolic link: `/var/www/html` → `/opt/itadmin`

## 🚀 Quick Start

### Prerequisites
- Ansible 2.9 or higher installed on jump host
- SSH access to all app servers
- Proper credentials configured in inventory file

### Running the Playbook

```bash
# Navigate to ansible directory
cd /home/thor/ansible

# Syntax check
ansible-playbook -i inventory playbook.yml --syntax-check

# Dry run (check mode)
ansible-playbook -i inventory playbook.yml --check

# Execute playbook
ansible-playbook -i inventory playbook.yml
```

## ✅ Validation

The playbook will be validated using:
```bash
ansible-playbook -i inventory playbook.yml
```

### Expected Output
```
PLAY RECAP ******************************************************************************
stapp01                    : ok=4    changed=2    unreachable=0    failed=0
stapp02                    : ok=4    changed=2    unreachable=0    failed=0
stapp03                    : ok=4    changed=2    unreachable=0    failed=0
```

## 🔍 Verification

After running the playbook, verify the results:

```bash
# Check on stapp01
ansible stapp01 -i inventory -m shell -a "ls -la /opt/itadmin/ && ls -la /var/www/html"

# Check on stapp02
ansible stapp02 -i inventory -m shell -a "ls -la /opt/itadmin/ && ls -la /var/www/html"

# Check on stapp03
ansible stapp03 -i inventory -m shell -a "ls -la /opt/itadmin/ && ls -la /var/www/html"
```

## 📚 Ansible Modules Used

- **file module**: Used for creating directories, files, and symbolic links
  - `state: directory` - Creates directories
  - `state: touch` - Creates empty files
  - `state: link` - Creates symbolic links

## 🔧 Troubleshooting

### Common Issues

**Issue**: Permission denied
```bash
# Solution: Ensure become: yes is set in playbook
```

**Issue**: Host unreachable
```bash
# Solution: Check connectivity
ansible -i inventory all -m ping
```

**Issue**: File already exists
```bash
# Solution: Playbook uses force: yes for symlinks
# Rerun the playbook - it's idempotent
```
 
## 📖 Additional Resources

- [Ansible File Module Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html)
- [Ansible Playbook Documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

## 👥 Team
**Nautilus DevOps Team**

## 📅 Last Updated
October 11, 2025

## 📄 License
Internal Use - Nautilus Infrastructure
