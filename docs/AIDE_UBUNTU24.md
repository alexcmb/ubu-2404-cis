# AIDE Configuration for Ubuntu 24.04

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
ubtu24cis_aide_db_recreate: true  # Force rebuild (recommended for USG compliance)

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
