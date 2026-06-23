# Ansible Password-less SSH Setup - Stratos DC

## 📋 Project Overview

This project documents the setup and configuration of password-less SSH connections between the Ansible controller (Jump Host) and managed nodes (App Servers) in the Stratos DC environment. The setup enables the Nautilus DevOps team to run Ansible playbooks seamlessly across different application servers.
 
## 🏗️ Architecture

| Component | Host | IP Address | User | Description |
|-----------|------|------------|------|-------------|
| **Ansible Controller** | Jump Host | - | `thor` | Main control node for running Ansible playbooks |
| **App Server 1** | stapp01 | 172.16.238.10 | `tony` | Managed node - Application server |
| **App Server 2** | stapp02 | 172.16.238.11 | `steve` | Managed node - Application server |
| **App Server 3** | stapp03 | 172.16.238.12 | `banner` | Managed node - Application server |

## 🚀 Quick Start

### Prerequisites

- SSH access to all servers
- `sshpass` utility installed on jump host
- Ansible installed on jump host
- Root/sudo access on all nodes

### 1. Inventory File Location

```
/home/thor/ansible/inventory
```

### 2. Generate SSH Key Pair

```bash
# As thor user on jump host
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

### 3. Set Up Password-less SSH

```bash
# Copy SSH keys to all app servers
sshpass -p 'Ir0nM@n' ssh-copy-id -o StrictHostKeyChecking=no tony@172.16.238.10
sshpass -p 'Am3ric@' ssh-copy-id -o StrictHostKeyChecking=no steve@172.16.238.11
sshpass -p 'BigGr33n' ssh-copy-id -o StrictHostKeyChecking=no banner@172.16.238.12
```

### 4. Test Ansible Connectivity

```bash
# Test specific server
ansible stapp01 -m ping -i /home/thor/ansible/inventory

# Test all servers
ansible all -m ping -i /home/thor/ansible/inventory
```

## 📁 Project Structure

```
/home/thor/ansible/
├── inventory              # Ansible inventory file
├── playbooks/            # Ansible playbooks directory
└── logs/                 # Execution logs
```

## 🔧 Configuration Files

### Inventory File Format

```ini
stapp01 ansible_host=172.16.238.10 ansible_user=tony
stapp02 ansible_host=172.16.238.11 ansible_user=steve
stapp03 ansible_host=172.16.238.12 ansible_user=banner
```

### SSH Configuration

Ensure the following SSH settings are configured on all nodes:
- `PubkeyAuthentication yes`
- `PasswordAuthentication no` (after key setup)
- `PermitRootLogin no`

## 🧪 Testing

### Basic Connectivity Test

```bash
# Ping test to App Server 1
ansible stapp01 -m ping -i /home/thor/ansible/inventory

# Expected output:
# stapp01 | SUCCESS => {
#     "ansible_facts": {
#         "discovered_interpreter_python": "/usr/bin/python"
#     },
#     "changed": false,
#     "ping": "pong"
# }
```

### Advanced Testing

```bash
# Test with verbose output
ansible stapp01 -m ping -i /home/thor/ansible/inventory -vvv

# Test command execution
ansible stapp01 -m shell -a "hostname && uptime" -i /home/thor/ansible/inventory
```

## 🔍 Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| SSH connection refused | Check SSH service status: `systemctl status sshd` |
| Permission denied | Verify SSH key permissions: `chmod 600 ~/.ssh/id_rsa` |
| Host key verification failed | Add `-o StrictHostKeyChecking=no` flag |
| Ansible command not found | Install Ansible: `pip3 install ansible` |

### Debug Commands

```bash
# Check inventory syntax
ansible-inventory -i /home/thor/ansible/inventory --list

# Test direct SSH connection
ssh tony@172.16.238.10 "echo 'SSH connection successful'"

# View Ansible configuration
ansible-config view
```

## 📊 Monitoring & Verification

### Daily Verification Script

```bash
#!/bin/bash
# Save as /home/thor/ansible/verify_connectivity.sh

echo "Testing Ansible connectivity to all servers..."
ansible all -m ping -i /home/thor/ansible/inventory
echo "Connectivity test completed."
```

### Log Monitoring

```bash
# Monitor Ansible logs
tail -f /var/log/ansible.log

# Check SSH authentication logs
sudo tail -f /var/log/auth.log
```

## 🔐 Security Considerations

- SSH keys are generated without passphrases for automation purposes
- Private keys are stored securely with 600 permissions
- Root login is disabled on all managed nodes
- All connections use SSH protocol version 2
- Regular key rotation is recommended

## 📚 Additional Resources

- [Ansible Inventory Documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)
- [SSH Key Management Best Practices](https://www.ssh.com/ssh/key)
- [Ansible Ad-Hoc Commands](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly using the verification scripts
5. Submit a pull request

## 📞 Support

For issues or questions regarding this setup:
- Create an issue in the project repository
- Contact the DevOps team lead
- Check troubleshooting section for common solutions

## 📄 License

This documentation is part of the Nautilus DevOps project and is intended for internal use within the Stratos DC infrastructure.

---
**Last Updated:** December 2025  
**Version:** 1.0  
**Maintainer:** Nautilus DevOps Team
