# Ubuntu 24 CIS Level 1 Compliance Guide

> **📌 Comprehensive guide to achieve CIS Level 1 Server/Workstation compliance for Ubuntu 24.04 LTS using Ansible**

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [CIS Level 1 Configuration](#cis-level-1-configuration)
- [Inventory Setup](#inventory-setup)
- [Running the Playbook](#running-the-playbook)
- [Key Variables for Level 1](#key-variables-for-level-1)
- [Section-by-Section Breakdown](#section-by-section-breakdown)
- [AIDE Configuration for Ubuntu 24.04](#aide-configuration-for-ubuntu-2404)
- [Post-Remediation Audit](#post-remediation-audit)
- [Troubleshooting](#troubleshooting)
- [Important Warnings](#important-warnings)

---

## Overview

This Ansible role configures an Ubuntu 24.04 LTS system to be compliant with the **CIS (Center for Internet Security) Benchmark v1.0.0**. The role supports both:

- **Level 1 - Server**: Basic security hardening for servers
- **Level 1 - Workstation**: Basic security hardening for workstations

### What is CIS Level 1?

CIS Level 1 benchmarks are designed to be practical and prudent security configurations that:
- Can be implemented quickly with minimal disruption
- Reduce system attack surface without impacting usability
- Are considered essential baseline security controls

---

## Prerequisites

### System Requirements

| Requirement | Details |
|-------------|---------|
| **Target OS** | Ubuntu 24.04 LTS (other versions NOT supported) |
| **Ansible Version** | 2.12+ |
| **Python Version** | 3.8+ |
| **Network** | SSH access to target hosts |

### Control Node Requirements

Install Ansible on your control node:

```bash
# Install Ansible
pip3 install ansible>=2.12

# Verify installation
ansible --version
```

### Required Ansible Collections

Install the required collections from `collections/requirements.yml`:

```bash
ansible-galaxy collection install -r collections/requirements.yml
```

Or install manually:

```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install community.crypto
ansible-galaxy collection install ansible.posix
```

---

## Quick Start

### 1. Clone/Download the Role

Ensure this role is in your Ansible roles directory or current working directory.

### 2. Configure Inventory

Create or edit `inventory.yml`:

```yaml
---
all:
  children:
    ubuntu_servers:
      hosts:
        my-server:
          ansible_host: <YOUR_SERVER_IP>
          ansible_user: <SSH_USER>
          ansible_ssh_private_key_file: ~/.ssh/your_private_key
          ansible_become: true
          ansible_become_method: sudo
          ansible_become_password: ""  # Leave empty if using passwordless sudo
```

### 3. Create Level 1 Configuration File

Create a file `group_vars/all/cis_level1.yml`:

```yaml
---
# ==========================================
# CIS LEVEL 1 ONLY CONFIGURATION
# ==========================================

# Enable Level 1, disable Level 2
ubtu24cis_level_1: true
ubtu24cis_level_2: false

# All sections enabled
ubtu24cis_section1: true
ubtu24cis_section2: true
ubtu24cis_section3: true
ubtu24cis_section4: true
ubtu24cis_section5: true
ubtu24cis_section6: true
ubtu24cis_section7: true

# Disable highly disruptive changes initially
ubtu24cis_disruption_high: false

# Skip automatic reboot
skip_reboot: true
```

### 4. Run the Playbook

```bash
# Dry-run (check mode) - ALWAYS do this first!
ansible-playbook -i inventory.yml site.yml --check --diff --tags "level1-server"

# Apply Level 1 Server compliance
ansible-playbook -i inventory.yml site.yml --tags "level1-server"

# Apply Level 1 Workstation compliance
ansible-playbook -i inventory.yml site.yml --tags "level1-workstation"
```

---

## CIS Level 1 Configuration

### Enabling Level 1 Only

To run **only Level 1 controls** (not Level 2), set these variables:

```yaml
# In your group_vars or extra vars
ubtu24cis_level_1: true
ubtu24cis_level_2: false
```

### Using Tags

The playbook supports tags to target specific security levels:

| Tag | Description |
|-----|-------------|
| `level1-server` | Level 1 controls for server deployments |
| `level1-workstation` | Level 1 controls for workstation deployments |
| `level2-server` | Level 2 controls for server deployments |
| `level2-workstation` | Level 2 controls for workstation deployments |

**Example commands:**

```bash
# Run ONLY Level 1 Server controls
ansible-playbook -i inventory.yml site.yml --tags "level1-server"

# Run ONLY Level 1 Workstation controls
ansible-playbook -i inventory.yml site.yml --tags "level1-workstation"

# Skip specific sections
ansible-playbook -i inventory.yml site.yml --tags "level1-server" --skip-tags "section3"
```

---

## Inventory Setup

### Example Complete Inventory

```yaml
---
all:
  children:
    # Production Ubuntu Servers
    ubuntu_servers:
      hosts:
        prod-web-01:
          ansible_host: 192.168.1.10
          ansible_user: ansible_admin
          ansible_ssh_private_key_file: ~/.ssh/ansible_key
          ansible_become: true
          ansible_become_method: sudo
        
        prod-db-01:
          ansible_host: 192.168.1.11
          ansible_user: ansible_admin
          ansible_ssh_private_key_file: ~/.ssh/ansible_key
          ansible_become: true
          ansible_become_method: sudo

    # Ubuntu Workstations
    ubuntu_workstations:
      hosts:
        dev-workstation-01:
          ansible_host: 192.168.1.50
          ansible_user: dev_user
          ansible_ssh_private_key_file: ~/.ssh/dev_key
          ansible_become: true
          ansible_become_method: sudo

  vars:
    # Global variables applied to all hosts
    ansible_python_interpreter: /usr/bin/python3
```

### Testing Connectivity

```bash
# Test connectivity to all hosts
ansible -i inventory.yml all -m ping

# Test with become
ansible -i inventory.yml all -m ping --become
```

---

## Running the Playbook

### Step-by-Step Execution

#### Step 1: Check Mode (Dry Run)

**ALWAYS** start with check mode to see what changes would be made:

```bash
ansible-playbook -i inventory.yml site.yml \
  --tags "level1-server" \
  --check \
  --diff
```

#### Step 2: Run on Single Host First

Test on one host before applying to all:

```bash
ansible-playbook -i inventory.yml site.yml \
  --tags "level1-server" \
  --limit "prod-web-01"
```

#### Step 3: Full Deployment

Apply to all target hosts:

```bash
ansible-playbook -i inventory.yml site.yml \
  --tags "level1-server"
```

### Command-Line Variable Overrides

```bash
# Example: Override variables at runtime
ansible-playbook -i inventory.yml site.yml \
  --tags "level1-server" \
  -e "ubtu24cis_disruption_high=false" \
  -e "skip_reboot=true"
```

---

## Key Variables for Level 1

Create a configuration file (e.g., `group_vars/all/cis_config.yml`) with these important variables:

### Core Settings

```yaml
---
# ==========================================
# CORE CIS SETTINGS
# ==========================================

# Benchmark Configuration
benchmark: UBUNTU24-CIS
benchmark_version: v1.0.0

# Level Selection (Level 1 ONLY)
ubtu24cis_level_1: true
ubtu24cis_level_2: false

# Section Toggles (all enabled for full Level 1 compliance)
ubtu24cis_section1: true  # Initial Setup
ubtu24cis_section2: true  # Services
ubtu24cis_section3: true  # Network Configuration
ubtu24cis_section4: true  # Host Based Firewall
ubtu24cis_section5: true  # Access Control
ubtu24cis_section6: true  # Logging and Auditing
ubtu24cis_section7: true  # System Maintenance

# Safety Settings
ubtu24cis_disruption_high: false  # Disable highly disruptive tasks initially
skip_reboot: true                  # Skip automatic reboot
```

### Network & Firewall Settings

```yaml
# ==========================================
# NETWORK CONFIGURATION
# ==========================================

# IPv4/IPv6 Settings
ubtu24cis_ipv4_required: true
ubtu24cis_ipv6_required: false  # Disable IPv6 if not needed

# Router Mode (set to false for regular servers)
ubtu24cis_is_router: false

# Firewall Package Selection
# Options: "ufw", "iptables", "nftables", or "none"
ubtu24cis_firewall_package: "ufw"

# UFW Allowed Outbound Ports
ubtu24cis_ufw_allow_out_ports:
  - port: 22
    proto: tcp
  - port: 53
    proto: tcp
  - port: 53
    proto: udp
  - port: 80
    proto: tcp
  - port: 443
    proto: tcp
  - port: 123
    proto: udp
```

### SSH Configuration

```yaml
# ==========================================
# SSH HARDENING
# ==========================================

ubtu24cis_sshd_log_level: "INFO"
ubtu24cis_sshd_max_auth_tries: 4
ubtu24cis_sshd_max_sessions: 10
ubtu24cis_sshd_login_grace_time: 60
ubtu24cis_sshd_client_alive_interval: 300
ubtu24cis_sshd_client_alive_count_max: 3

# Allowed/Denied Users (customize for your environment)
ubtu24cis_sshd_allow_users: ""
ubtu24cis_sshd_allow_groups: ""
ubtu24cis_sshd_deny_users: ""
ubtu24cis_sshd_deny_groups: ""

# SSH Ciphers (FIPS 140 compliant)
ubtu24cis_sshd_ciphers:
  - aes256-gcm@openssh.com
  - aes128-gcm@openssh.com
  - aes256-ctr
  - aes192-ctr
  - aes128-ctr
```

### Password Policy

```yaml
# ==========================================
# PASSWORD POLICY
# ==========================================

# Password Aging
ubtu24cis_pass_max_days: 365   # Max password age
ubtu24cis_pass_min_days: 1     # Min days between changes
ubtu24cis_pass_warn_age: 7     # Warning before expiry
ubtu24cis_pass_inactive: 45    # Days before account lock

# Password Quality (pwquality)
ubtu24cis_passwd_minlen_value: 14     # Minimum length
ubtu24cis_passwd_minclass: 4          # Character classes required
ubtu24cis_passwd_dcredit: -1          # Digit requirement
ubtu24cis_passwd_ucredit: -2          # Uppercase requirement
ubtu24cis_passwd_ocredit: -1          # Special char requirement
ubtu24cis_passwd_lcredit: -2          # Lowercase requirement
ubtu24cis_passwd_maxrepeat_value: 3   # Max repeated chars
ubtu24cis_passwd_dictcheck_value: 1   # Dictionary check

# Account Lockout
ubtu24cis_faillock_deny: 5            # Failed attempts before lock
ubtu24cis_faillock_unlock_time: 900   # Lockout duration (seconds)
```

### Services Configuration

```yaml
# ==========================================
# SERVICES CONFIGURATION
# ==========================================

# Disable unnecessary services (set to false to remove/disable)
ubtu24cis_autofs_services: false
ubtu24cis_avahi_server: false
ubtu24cis_dhcp_server: false
ubtu24cis_dns_server: false
ubtu24cis_ftp_server: false
ubtu24cis_ldap_server: false
ubtu24cis_nfs_server: false
ubtu24cis_print_server: false
ubtu24cis_rsync_server: false
ubtu24cis_samba_server: false
ubtu24cis_snmp_server: false
ubtu24cis_telnet_server: false
ubtu24cis_tftp_server: false
ubtu24cis_apache2_server: false
ubtu24cis_nginx_server: false
ubtu24cis_xwindow_server: false

# Client Services
ubtu24cis_nis_client_required: false
ubtu24cis_rsh_client: false
ubtu24cis_talk_client: false
ubtu24cis_telnet_required: false
ubtu24cis_ftp_client: false
```

### Time Synchronization

```yaml
# ==========================================
# TIME SYNCHRONIZATION
# ==========================================

# Time sync tool: "chrony" or "systemd-timesyncd"
ubtu24cis_time_sync_tool: "systemd-timesyncd"

# NTP Servers
ubtu24cis_time_servers:
  - name: time-a-g.nist.gov
    options: iburst
  - name: time-b-g.nist.gov
    options: iburst

# NTP Pools
ubtu24cis_time_pool:
  - name: time.nist.gov
    options: iburst maxsources 4
```

---

## Section-by-Section Breakdown

### Section 1: Initial Setup

| Rule | Description | Level |
|------|-------------|-------|
| 1.1.x | Filesystem Configuration | L1 |
| 1.2.x | Package Management | L1 |
| 1.3.x | AppArmor Configuration | L1 |
| 1.4.x | Bootloader Configuration | L1 |
| 1.5.x | Process Hardening | L1 |
| 1.6.x | Warning Banners | L1 |
| 1.7.x | GNOME Display Manager | L1 |

**Key files affected:**
- `/etc/fstab` - Mount options
- `/etc/modprobe.d/` - Kernel module blacklisting
- `/etc/issue`, `/etc/issue.net`, `/etc/motd` - Banners

### Section 2: Services

| Rule | Description | Level |
|------|-------------|-------|
| 2.1.x | Server Services | L1 |
| 2.2.x | Client Services | L1 |
| 2.3.x | Time Synchronization | L1 |
| 2.4.x | Job Schedulers | L1 |

**Services typically disabled:**
- avahi-daemon, cups, dhcp, dns, ftp, ldap, nfs, rsync, samba, snmp, telnet, tftp, xserver

### Section 3: Network Configuration

| Rule | Description | Level |
|------|-------------|-------|
| 3.1.x | Network Devices | L1 |
| 3.2.x | Kernel Modules | L1 |
| 3.3.x | Kernel Parameters | L1 |

**Key configurations:**
- IPv6 disable (if not required)
- Wireless interface disable
- IP forwarding settings
- Source routing, ICMP redirects, etc.

### Section 4: Host Based Firewall

| Rule | Description | Level |
|------|-------------|-------|
| 4.1.x | Single Firewall | L1 |
| 4.2.x | UFW Configuration | L1 |
| 4.3.x | nftables | L1 |
| 4.4.x | iptables | L1 |

**Recommended:** Use UFW (`ubtu24cis_firewall_package: "ufw"`)

### Section 5: Access Control

| Rule | Description | Level |
|------|-------------|-------|
| 5.1.x | SSH Server | L1 |
| 5.2.x | Privilege Escalation (sudo) | L1 |
| 5.3.x | PAM Configuration | L1 |
| 5.4.x | User Accounts | L1 |

**Critical configurations:**
- SSH hardening
- sudo configuration
- Password quality
- Account lockout policies

### Section 6: Logging and Auditing

| Rule | Description | Level |
|------|-------------|-------|
| 6.1.x | System Logging | L1 |
| 6.2.x | auditd Configuration | L1 |
| 6.3.x | AIDE (File Integrity) | L1 |

---

## AIDE Configuration for Ubuntu 24.04

> [!IMPORTANT]
> Ubuntu 24.04 **does not provide** the systemd files required by CIS for AIDE (`aidecheck.service` and `aidecheck.timer`). You must create them manually.

### Step 1: Create AIDE Systemd Service

Create the service file:

```bash
sudo tee /etc/systemd/system/aidecheck.service > /dev/null << 'EOF'
[Unit]
Description=AIDE Check

[Service]
Type=oneshot
ExecStart=/usr/bin/aide --check --config /etc/aide/aide.conf
EOF
```

### Step 2: Create AIDE Systemd Timer

Create the timer file for daily checks:

```bash
sudo tee /etc/systemd/system/aidecheck.timer > /dev/null << 'EOF'
[Unit]
Description=Run AIDE check daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
EOF
```

### Step 3: Enable and Start the Timer

```bash
sudo systemctl daemon-reload
sudo systemctl enable aidecheck.timer
sudo systemctl start aidecheck.timer

# Verify the timer is active
sudo systemctl status aidecheck.timer
```

### Step 4: Configure AIDE Exclusions

AIDE will report false positives for volatile files that change frequently. Add these exclusions to `/etc/aide/aide.conf`:

```bash
sudo tee -a /etc/aide/aide.conf > /dev/null << 'EOF'

# ==========================================
# VOLATILE FILE EXCLUSIONS FOR CIS COMPLIANCE
# ==========================================

# Systemd runtime files
!/run/systemd/dynamic-uid/.*
!/run/systemd/propagate/systemd-journal-upload.service
!/run/faillock/.*

# Systemd private data
!/var/lib/private/systemd/journal-upload/.*

# Logging (rotated frequently)
!/var/log/sudo.log
!/var/log/audit/audit.log
!/var/log/audit/audit.log.*
!/var/log/journal/.*

# Azure Agent (for Azure VMs)
!/var/lib/waagent/.*
!/var/log/waagent.log

# Sysstat
!/var/log/sysstat/.*
!/var/log/sa/.*

# Ubuntu Security Guide (USG)
!/var/lib/usg/.*
!/tmp/usg-.*\.html
!/tmp/usg-.*\.xml

# Exclusions systemd dynamic UID (volatile)
!/run/systemd/dynamic-uid
!/run/systemd/dynamic-uid/.*

# Exclusion journal-upload private directory (volatile)
!/var/lib/private/systemd/journal-upload
!/var/lib/private/systemd/journal-upload/.*

# Systemd timers (fwupd and others)
!/var/lib/systemd/timers/.*

# Canonical Livepatch (for Ubuntu Pro systems)
!/var/snap/canonical-livepatch/common/.*
EOF
```

### Step 5: Regenerate AIDE Database

For CIS compliance, the AIDE database must have **0 Added / 0 Changed** files:

```bash
# 1. Remove old database files
sudo rm -f /var/lib/aide/aide.db
sudo rm -f /var/lib/aide/aide.db.new

# 2. Initialize new database
sudo aide --init --config /etc/aide/aide.conf

# 3. Install the new database
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# 4. Verify (should show 0 differences)
sudo aide --check --config /etc/aide/aide.conf
```

### AIDE Variables Reference

Configure AIDE behavior in your Ansible variables:

```yaml
# Enable AIDE configuration
ubtu24cis_config_aide: true

# Scan method: cron or timer
ubtu24cis_aide_scan: timer  # Use 'timer' for systemd-based scheduling

# Cron schedule (if using cron method)
ubtu24cis_aide_cron_user: root
ubtu24cis_aide_cron_hour: 5
ubtu24cis_aide_cron_minute: 0

# Database settings
ubtu24cis_aide_db_file: /var/lib/aide/aide.db
ubtu24cis_aide_db_file_age: 1w  # Rebuild if older than 1 week
ubtu24cis_aide_db_recreate: false

# Initialization timeout
ubtu24cis_aide_init_async: 600    # Max seconds to wait
ubtu24cis_aide_init_poll: 15      # Polling interval
```

### Verifying AIDE Compliance

After configuration, verify the AIDE rules pass:

```bash
# Check timer is running
systemctl is-active aidecheck.timer

# Run manual check
sudo aide --check --config /etc/aide/aide.conf

# Expected output should show:
#   AIDE found NO differences between database and filesystem.
```

---

### Section 7: System Maintenance

| Rule | Description | Level |
|------|-------------|-------|
| 7.1.x | File Permissions | L1 |
| 7.2.x | User/Group Settings | L1 |

---

## Post-Remediation Audit

### Enable Auditing

The role includes automated auditing using [Goss](https://github.com/goss-org/goss):

```yaml
# In your variables file
setup_audit: true
run_audit: true
audit_only: false
```

### Run with Auditing

```bash
ansible-playbook -i inventory.yml site.yml \
  --tags "level1-server,run_audit"
```

### Audit Results

Audit results are stored in `/opt/` on the target host and include:
- Pre-remediation audit
- Post-remediation audit
- Comparison summary

### Manual Compliance Check

For additional verification, use the [UBUNTU24-CIS-Audit](https://github.com/ansible-lockdown/UBUNTU24-CIS-Audit) role.

---

## Troubleshooting

### Common Issues

#### 1. SSH Connection Lost After Running

**Problem:** SSH access is blocked after firewall configuration.

**Solution:** Pre-configure UFW to allow SSH:

```yaml
ubtu24cis_ufw_allow_out_ports:
  - port: 22
    proto: tcp
```

Or add to your inventory:
```yaml
ansible_port: 22  # Ensure correct SSH port
```

#### 2. Root Password Not Set Error

**Problem:** Role fails with "root password is set" assertion.

**Solution:** Set a root password before running:
```bash
sudo passwd root
```

#### 3. Sudo Password Required

**Problem:** Role fails because connecting user has no password.

**Solution:** Either set a password for the user or exclude from check:
```yaml
ubtu24cis_sudoers_exclude_nopasswd_list:
  - your_ansible_user
```

#### 4. Specific Rule Failures

**Solution:** Disable specific problematic rules:
```yaml
ubtu24cis_rule_1_1_1_1: false  # Example: disable cramfs check
```

#### 5. AIDE Timer/Service Missing (Ubuntu 24.04)

**Problem:** CIS rule "AIDE Timer" fails because Ubuntu 24.04 doesn't provide `aidecheck.service` or `aidecheck.timer`.

**Solution:** Create the systemd files manually. See the [AIDE Configuration for Ubuntu 24.04](#aide-configuration-for-ubuntu-2404) section.

#### 6. AIDE Reports Changes After Clean Install

**Problem:** AIDE check shows "Added" or "Changed" files even after database initialization.

**Solution:** Add exclusions for volatile files in `/etc/aide/aide.conf` and regenerate the database:
```bash
# Add exclusions for volatile files (see AIDE section above)
# Then regenerate:
sudo rm -f /var/lib/aide/aide.db*
sudo aide --init --config /etc/aide/aide.conf
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

### Verbose Output

Run with increased verbosity for debugging:

```bash
ansible-playbook -i inventory.yml site.yml --tags "level1-server" -vvv
```

### Check Mode Doesn't Guarantee Safety

> ⚠️ **WARNING:** Check mode is NOT fully supported. Always test on a non-production system first!

---

## Important Warnings

> [!CAUTION]
> This role **will make changes** to the system which may have unintended consequences. This is a **remediation tool**, not an auditing tool.

### Critical Considerations

1. **⚠️ ALWAYS TEST FIRST** - Run on a test system before production
2. **💾 BACKUP YOUR SYSTEM** - Create snapshots/backups before applying
3. **🔐 SSH ACCESS** - Ensure you won't lock yourself out (keep a console session open)
4. **📋 REVIEW CHANGES** - Study the `defaults/main.yml` file thoroughly
5. **🔄 CHECK MODE LIMITATIONS** - Check mode doesn't catch all issues
6. **🖥️ CLEAN INSTALL** - The role was developed against clean OS installs

### Pre-Flight Checklist

- [ ] Read through `defaults/main.yml` and understand all variables
- [ ] Create a backup/snapshot of target systems
- [ ] Test on a non-production system first
- [ ] Run in check mode (`--check`) before applying
- [ ] Ensure console/out-of-band access is available
- [ ] Review services that will be disabled
- [ ] Verify firewall rules won't block SSH
- [ ] Confirm root password is set (if required)

---

## Additional Resources

- **CIS Benchmarks:** [CIS Website](https://www.cisecurity.org/cis-benchmarks/)
- **Ansible Lockdown:** [GitHub Organization](https://github.com/ansible-lockdown)
- **Documentation:** [Read The Docs](https://ansible-lockdown.readthedocs.io/en/latest/)
- **Community:** [Discord Server](https://www.lockdownenterprise.com/discord)

---

## License

This role is licensed under the terms specified in the [LICENSE](LICENSE) file.

---

## Credits

Based on the [ansible-lockdown/UBUNTU24-CIS](https://github.com/ansible-lockdown/UBUNTU24-CIS) role.

**Original Authors:** Mark Bolwell, George Nalen, Steve Williams, Fred Witty

---

*Last Updated: January 2026 - Compatible with CIS Ubuntu 24.04 LTS Benchmark v1.0.0*
