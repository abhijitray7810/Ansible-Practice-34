# Ansible HTTPD Installation and Configuration

## Overview 

This Ansible playbook automates the installation and configuration of Apache HTTP Server (httpd) on application servers in the Stratos Data Center. The playbook ensures httpd is installed, started, and enabled to run on system boot across all app servers.

## Project Structure

``` 
/home/thor/ansible/
├── inventory          # Ansible inventory file (pre-configured)
├── playbook.yml      # Main playbook for httpd deployment
└── README.md         # This documentation file
```

## Prerequisites
 
- Ansible installed on the jump host
- SSH access configured from jump host to all app servers
- User `thor` has sudo privileges on target servers
- Target servers are RHEL/CentOS based systems

## Playbook Details

### Target Hosts
- All hosts defined in the inventory file
- Serial execution (one host at a time) to prevent resource exhaustion

### Tasks Performed

1. **Install httpd package**
   - Uses yum package manager
   - Updates yum cache before installation
   - Includes retry logic (3 attempts with 5-second delays)

2. **Start and enable httpd service**
   - Starts the httpd service immediately
   - Enables httpd to start automatically on system boot

## Usage

### Running the Playbook

Execute the playbook from the `/home/thor/ansible` directory:

```bash
cd /home/thor/ansible
ansible-playbook -i inventory playbook.yml
```

### Expected Output

A successful run should show:

```
PLAY [Install and configure httpd on app servers] ***************************************

TASK [Gathering Facts] ******************************************************************
ok: [stapp01]

TASK [Install httpd package] ************************************************************
changed: [stapp01]

TASK [Start and enable httpd service] ***************************************************
changed: [stapp01]

PLAY RECAP ******************************************************************************
stapp01                    : ok=3    changed=2    unreachable=0    failed=0
stapp02                    : ok=3    changed=2    unreachable=0    failed=0
stapp03                    : ok=3    changed=2    unreachable=0    failed=0
```

## Playbook Features

### Idempotency
The playbook is idempotent, meaning you can run it multiple times safely. It will only make changes when necessary:
- If httpd is already installed, it skips installation
- If httpd is already running and enabled, it skips service configuration

### Error Handling
- **Retry Logic**: Automatically retries package installation up to 3 times
- **Serial Execution**: Processes one host at a time to avoid memory/resource issues
- **Privilege Escalation**: Uses `become: yes` to run tasks with root privileges

## Verification

After running the playbook, verify httpd is working:

### Check Service Status
```bash
ansible all -i inventory -b -m shell -a "systemctl status httpd"
```

### Check if httpd is enabled
```bash
ansible all -i inventory -b -m shell -a "systemctl is-enabled httpd"
```

### Test HTTP Response
```bash
ansible all -i inventory -b -m shell -a "curl -I localhost"
```

## Troubleshooting

### Common Issues

**Issue**: Module failure with error code 137
- **Cause**: Memory constraints or timeout
- **Solution**: The playbook uses serial execution to prevent this

**Issue**: Package installation fails
- **Cause**: Network issues or repository problems
- **Solution**: The playbook includes retry logic with delays

**Issue**: Permission denied
- **Cause**: User doesn't have sudo privileges
- **Solution**: Ensure thor user has sudo access on app servers

### Debug Mode

Run the playbook with verbose output for troubleshooting:

```bash
ansible-playbook -i inventory playbook.yml -v    # Verbose
ansible-playbook -i inventory playbook.yml -vv   # More verbose
ansible-playbook -i inventory playbook.yml -vvv  # Very verbose
```

### Check Connectivity

Test connection to all app servers:

```bash
ansible all -i inventory -m ping
```

## Maintenance

### Updating the Playbook

To modify the playbook:

1. Edit `/home/thor/ansible/playbook.yml`
2. Test changes with `--check` flag (dry run):
   ```bash
   ansible-playbook -i inventory playbook.yml --check
   ```
3. Run the playbook normally after verification

### Adding More Tasks

To extend functionality, add new tasks to the playbook following this format:

```yaml
- name: Description of task
  module_name:
    parameter: value
    state: present
```

## Security Considerations

- The playbook uses privilege escalation (`become: yes`)
- Ensure SSH keys are properly secured
- Review sudo permissions for the thor user
- Keep Ansible and system packages updated

## Support

For issues or questions:
- Check Ansible documentation: https://docs.ansible.com
- Review playbook execution logs
- Verify inventory configuration
- Ensure network connectivity between jump host and app servers

## Version History

- **v1.0** - Initial release with httpd installation and service management
- **v1.1** - Added serial execution and retry logic for stability

## License

Internal use only - Stratos Data Center DevOps Team

---

**Last Updated**: October 2025  
**Maintained By**: DevOps Team  
**Environment**: Stratos DC - Nautilus App Servers
