# PAN-OS High Availability Upgrade Playbook

An Ansible playbook for automated PAN-OS software upgrades in High Availability (HA) configurations with zero-downtime deployment.

## Overview

This playbook automates the complex process of upgrading Palo Alto Networks firewalls configured in High Availability pairs. It ensures a controlled, sequential upgrade process that minimizes downtime and maintains network security throughout the upgrade process.

## Prerequisites

### System Requirements

#### Ansible Control Node
- **Ansible Version**: 2.9+ (recommended: 4.0+)
- **Python**: 3.6+ (required for PAN-OS collection)
- **Operating System**: Linux/macOS (Windows with WSL)

#### Network Requirements
- Management network connectivity to both firewalls
- Internet access for downloading PAN-OS software and content updates
- Sufficient bandwidth for software downloads (typically 200-500MB per firewall)

#### Palo Alto Networks Firewalls
- **HA Configuration**: Both firewalls must be properly configured in HA mode
- **Disk Space**: Minimum 2GB free space on both firewalls for software installation
- **Administrative Access**: Account with administrative privileges
- **API Access**: Management interface accessible via HTTPS (default port 443)

### Required Ansible Collections

#### Primary Collection
```bash
# Install the Palo Alto Networks Ansible collection
ansible-galaxy collection install paloaltonetworks.panos
```

#### Dependencies
The PAN-OS collection automatically installs these Python dependencies:
- `pan-os-python` - PAN-OS SDK for Python
- `xmltodict` - XML parsing library
- `requests` - HTTP library
- `urllib3` - HTTP client library

## Installation Instructions

### 1. Install Ansible
```bash
# On Ubuntu/Debian
sudo apt update
sudo apt install ansible

# On CentOS/RHEL/Fedora
sudo yum install ansible
# or
sudo dnf install ansible

# On macOS
brew install ansible

# Via Python pip (all platforms)
pip3 install ansible
```

### 2. Install PAN-OS Collection
```bash
# Install from Ansible Galaxy
ansible-galaxy collection install paloaltonetworks.panos

# Verify installation
ansible-galaxy collection list | grep panos
```

### 3. Verify Python Dependencies
```bash
# Check if pan-os-python is installed
python3 -c "import panos; print('PAN-OS Python SDK is installed')"

# If not installed, install manually
pip3 install pan-os-python
```

### 4. Download the Playbook
```bash
# Create directory for the playbook
mkdir panos-ha-upgrade
cd panos-ha-upgrade

# Download the playbook file
# (Place your playbook file here as PA_PAN-OS_Upgrade_HA.yml)
```

### 5. Verify Connectivity
Test connectivity to your firewalls:
```bash
# Test basic connectivity
ping <primary_firewall_ip>
ping <secondary_firewall_ip>

# Test API connectivity
curl -k https://<firewall_ip>/api/?type=keygen&user=<username>&password=<password>
```

## Usage

### Basic Execution
```bash
# Run the playbook (interactive prompts will appear)
ansible-playbook PA_PAN-OS_Upgrade_HA.yml
```

### Dry Run Mode
```bash
# Test the playbook without making changes
ansible-playbook PA_PAN-OS_Upgrade_HA.yml
# When prompted, select 'y' for dry-run mode
```

### Advanced Usage with Tags
```bash
# Run only pre-checks
ansible-playbook PA_PAN-OS_Upgrade_HA.yml --tags "pre_check"

# Run content updates only
ansible-playbook PA_PAN-OS_Upgrade_HA.yml --tags "content"

# Skip content updates
ansible-playbook PA_PAN-OS_Upgrade_HA.yml --skip-tags "content"
```

## Playbook Workflow

### Phase 1: Pre-Upgrade Validation
1. **HA Status Check**: Validates HA configuration and health
2. **Version Detection**: Identifies current software versions
3. **Preemption Management**: Handles HA preemption settings
4. **Content Updates**: Updates threat intelligence databases

### Phase 2: Passive Firewall Upgrade
1. **Software Download**: Downloads target PAN-OS version
2. **Installation**: Installs software and reboots passive firewall
3. **Verification**: Confirms successful upgrade and API readiness

### Phase 3: Active Firewall Upgrade
1. **Failover**: Initiates controlled failover to upgraded firewall
2. **Software Installation**: Upgrades the original active firewall
3. **HA Restoration**: Restores normal HA operations

### Phase 4: Post-Upgrade Validation
1. **Final Verification**: Confirms both firewalls running target version
2. **HA Status Check**: Validates HA pair health
3. **Preemption Restoration**: Restores original preemption settings

## Interactive Prompts

The playbook will prompt for the following information:

| Prompt | Description | Example |
|--------|-------------|---------|
| Primary Firewall IP | Management IP of primary firewall | `192.168.1.10` |
| Secondary Firewall IP | Management IP of secondary firewall | `192.168.1.11` |
| API Port (if custom) | Custom API port if not 443 | `8443` |
| Username | Administrative username | `admin` |
| Password | Administrative password | `<hidden>` |
| Target Version | Desired PAN-OS version | `11.1.0` |
| Dry Run | Test mode without changes | `y` or `n` |

## Safety Features

### 🔒 **Critical Safety Checks**
- **Backup Verification**: Playbook reminds users to create backups
- **HA Health Validation**: Ensures HA is functional before proceeding
- **Version Compatibility**: Warns about version compatibility issues
- **Rollback Information**: Provides guidance for manual rollback if needed

### ⏱️ **Timeout Management**
- **Reboot Timeout**: 30 minutes (1800 seconds) for firewall reboots
- **Upgrade Timeout**: 60 minutes (3600 seconds) for full upgrade process
- **API Readiness**: Additional wait times for API availability

### 📝 **Comprehensive Logging**
All operations are logged with detailed status information, making troubleshooting easier if issues arise.

## Troubleshooting

### Common Issues

#### Connection Problems
```bash
# Test API connectivity (default port)
curl -k -X GET "https://<firewall-ip>/api/?type=keygen&user=<username>&password=<password>"

# Test API connectivity with custom port
curl -k -X GET "https://<firewall-ip>:<custom_port>/api/?type=keygen&user=<username>&password=<password>"

# Example: Testing with custom port 8443
curl -k -X GET "https://192.168.1.10:8443/api/?type=keygen&user=admin&password=yourpassword"
```

#### Collection Not Found
```bash
# Reinstall PAN-OS collection
ansible-galaxy collection install --force paloaltonetworks.panos
```

#### Python Dependencies Missing
```bash
# Install required Python packages
pip3 install pan-os-python xmltodict requests
```

#### HA State Issues
- Verify HA configuration on both firewalls
- Check HA interface connectivity
- Ensure both firewalls can communicate over HA links

### Recovery Procedures

#### If Upgrade Fails
1. **Stop Playbook**: Use Ctrl+C to halt execution
2. **Check Firewall Status**: Manually verify firewall states
3. **Restore from Backup**: Use previously created configuration backups
4. **Manual Recovery**: Follow Palo Alto's manual upgrade procedures

#### Emergency Contact
- Maintain contact information for Palo Alto Networks support
- Have backup network access methods available
- Ensure out-of-band management access is configured

## Version History

### Available Versions
- **Complete Version**: Includes HA preemption management
- **Simplified Version**: Basic upgrade without preemption handling
- **Port-Aware Version**: Supports custom API ports

## Best Practices

### Before Running
1. **Create Full Backups** of both firewalls
2. **Schedule Maintenance Window** (typically 45-90 minutes)
3. **Verify HA Health** manually before automation
4. **Test in Lab Environment** if possible
5. **Prepare Rollback Plan** in case of issues

### During Execution
1. **Monitor Progress** through playbook output
2. **Avoid Interruption** unless absolutely necessary
3. **Have Backup Access** to firewalls ready
4. **Document Any Issues** encountered

### After Completion
1. **Verify Functionality** of all security policies
2. **Check Performance** metrics
3. **Update Documentation** with new version information
4. **Archive Upgrade Logs** for future reference

## Support and Contributing

For issues, questions, or contributions:
- Review Ansible and PAN-OS documentation
- Check Palo Alto Networks community forums
- Consult your organization's network security team

## License

This playbook is provided as-is for educational and operational purposes. Always test thoroughly in non-production environments before using in production.