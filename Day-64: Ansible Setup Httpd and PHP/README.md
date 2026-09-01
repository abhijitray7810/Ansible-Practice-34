# Apache and PHP Setup with Ansible 

This playbook automates the installation and configuration of Apache (httpd) and PHP on App Server 1 in the Stratos Datacenter.

## Prerequisites

- Access to the jump host
- Ansible installed on the jump host
- SSH connectivity to App Server 1
- Sudo/root privileges on App Server 1
- Valid inventory file at `~/playbooks/inventory`
- Template file `~/playbooks/templates/phpinfo.php.j2`

## Project Structure

```
~/playbooks/
├── inventory
├── httpd.yml
├── templates/
│   └── phpinfo.php.j2
└── README.md
```

## What This Playbook Does

1. **Package Installation**
   - Installs Apache HTTP Server (`httpd`)
   - Installs PHP runtime (`php`)

2. **Document Root Configuration**
   - Creates custom document root: `/var/www/html/myroot`
   - Updates Apache configuration file: `/etc/httpd/conf/httpd.conf`
   - Changes `DocumentRoot` directive to point to new location
   - Updates `<Directory>` directive to match new document root

3. **PHP Info Page Deployment**
   - Copies `phpinfo.php.j2` template to document root
   - Names the file `phpinfo.php`
   - Sets ownership to `apache:apache`
   - Sets file permissions to `644`

4. **Service Management**
   - Starts the httpd service
   - Enables httpd to start automatically on boot

## Usage

### Step 1: Verify Inventory

Check that your inventory file contains App Server 1. Example inventory:

```ini
[app_servers]
stapp01 ansible_host=172.16.238.10 ansible_user=tony ansible_ssh_pass=Ir0nM@n
```

**Note:** Adjust the hostname (`stapp01`) if your inventory uses a different name.

### Step 2: Verify Template Exists

Ensure the PHP template file exists:

```bash
ls -l ~/playbooks/templates/phpinfo.php.j2
```

### Step 3: Run the Playbook

Execute from the `~/playbooks/` directory:

```bash
cd ~/playbooks/
ansible-playbook -i inventory httpd.yml
```

### Step 4: Verify Installation

After successful execution, verify the setup:

```bash
# Check httpd service status
ansible stapp01 -i inventory -m shell -a "systemctl status httpd"

# Check if phpinfo.php exists
ansible stapp01 -i inventory -m shell -a "ls -l /var/www/html/myroot/phpinfo.php"

# Test Apache configuration
ansible stapp01 -i inventory -m shell -a "httpd -t"
```

## Expected Output

Successful playbook execution should show:

```
PLAY [Setup Apache and PHP on App Server 1] ************************************

TASK [Gathering Facts] *********************************************************
ok: [stapp01]

TASK [Install httpd and php packages] ******************************************
changed: [stapp01]

TASK [Create custom document root directory] ***********************************
changed: [stapp01]

TASK [Change Apache document root in httpd.conf] *******************************
changed: [stapp01]

TASK [Update Directory directive for new document root] ************************
changed: [stapp01]

TASK [Copy phpinfo.php template to document root] ******************************
changed: [stapp01]

TASK [Start and enable httpd service] ******************************************
changed: [stapp01]

PLAY RECAP *********************************************************************
stapp01                    : ok=7    changed=6    unreachable=0    failed=0
```

## Testing the Web Server

Once deployed, test the Apache and PHP installation:

### From Jump Host:

```bash
# Get the IP address of App Server 1
APP_SERVER_IP=$(grep stapp01 ~/playbooks/inventory | awk '{print $2}' | cut -d'=' -f2)

# Test with curl
curl http://$APP_SERVER_IP/phpinfo.php
```

### From a Browser:

Navigate to: `http://<app-server-1-ip>/phpinfo.php`

You should see the PHP information page displaying PHP version, modules, and configuration.

## Troubleshooting

### Issue: Playbook fails at package installation

**Solution:** Check network connectivity and yum repository configuration:
```bash
ansible stapp01 -i inventory -m shell -a "yum repolist"
```

### Issue: Permission denied errors

**Solution:** Ensure you have sudo privileges:
```bash
ansible stapp01 -i inventory -m shell -a "sudo -l"
```

### Issue: httpd service fails to start

**Solution:** Check Apache configuration syntax:
```bash
ansible stapp01 -i inventory -m shell -a "httpd -t"
```

Check for port conflicts:
```bash
ansible stapp01 -i inventory -m shell -a "netstat -tulpn | grep :80"
```

### Issue: Template file not found

**Solution:** Verify the template exists in the correct location:
```bash
ls -l ~/playbooks/templates/phpinfo.php.j2
```

### Issue: SELinux preventing Apache access

**Solution:** Check SELinux context:
```bash
ansible stapp01 -i inventory -m shell -a "ls -Z /var/www/html/myroot/"
```

If needed, restore SELinux context:
```bash
ansible stapp01 -i inventory -m shell -a "restorecon -Rv /var/www/html/myroot/"
```

## Configuration Details

### Modified Files

- `/etc/httpd/conf/httpd.conf` - Apache main configuration (backup created automatically)
- `/var/www/html/myroot/phpinfo.php` - PHP info test page

### New Directories

- `/var/www/html/myroot/` - Custom Apache document root

### Services

- `httpd.service` - Apache HTTP Server (enabled and started)

## Rollback

To revert changes:

1. Stop and disable httpd:
   ```bash
   ansible stapp01 -i inventory -m systemd -a "name=httpd state=stopped enabled=no" --become
   ```

2. Restore original Apache configuration:
   ```bash
   ansible stapp01 -i inventory -m shell -a "cp /etc/httpd/conf/httpd.conf.backup /etc/httpd/conf/httpd.conf" --become
   ```

3. Remove installed packages:
   ```bash
   ansible stapp01 -i inventory -m yum -a "name=httpd,php state=absent" --become
   ```

## Additional Notes

- This playbook is idempotent - running it multiple times produces the same result
- Configuration backups are created automatically in `/etc/httpd/conf/`
- The playbook uses `become: yes` for privilege escalation
- Default yum repository packages are installed (no specific versions)

## Author

DevOps Team - Nautilus Application Development

## Version

1.0.0 - Initial Release
