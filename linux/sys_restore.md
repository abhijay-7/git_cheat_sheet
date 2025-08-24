# Fedora Package Backup and Restore Guide

## Method 1: Using DNF History (Recommended)

### Backup Package List
```bash
# Method 1: Get user-installed packages (works on all Fedora versions)
dnf repoquery --userinstalled --qf "%{name}" > user-packages.txt


# Method 2: Using RPM directly (most reliable)
rpm -qa --qf "%{NAME}\n" | sort > all-rpm-packages.txt
```

### Restore on New System
```bash
# Install all packages from the list
sudo dnf install $(cat user-packages.txt)

# Or install one by one with error handling
while read package; do
    sudo dnf install -y "$package"
done < user-packages.txt
```

## Method 2: Using RPM Database

### Backup Package List
```bash
# List all installed packages with RPM
rpm -qa > all-rpm-packages.txt

# Get only package names (without version info)
rpm -qa --qf "%{NAME}\n" > rpm-package-names.txt

# Get packages installed by user (not from base system)
rpm -qa --qf "%{NAME} %{INSTALLTIME:date}\n" | sort -k2 > packages-with-dates.txt
```

### Restore on New System
```bash
# Install packages from RPM list
sudo dnf install $(cat rpm-package-names.txt)
```

## Method 3: Complete System Backup (Most Comprehensive)

### Create Complete Package Backup
```bash
# Create a comprehensive backup script
#!/bin/bash

# Create backup directory
mkdir -p ~/package-backup
cd ~/package-backup

# 1. List user-installed packages
dnf history userinstalled > userinstalled-packages.txt

# 2. List all installed packages
dnf list installed > all-packages.txt

# 3. List enabled repositories
dnf repolist enabled > enabled-repos.txt

# 4. Export RPM database
rpm -qa --qf "%{NAME}-%{VERSION}-%{RELEASE}.%{ARCH}\n" > rpm-packages.txt

# 5. Get Flatpak packages (if using Flatpak)
flatpak list --app --columns=application > flatpak-packages.txt 2>/dev/null || echo "No Flatpak packages"

# 6. Get Snap packages (if using Snap)
snap list > snap-packages.txt 2>/dev/null || echo "No Snap packages"

# 7. Create a summary
echo "Package backup created on $(date)" > backup-info.txt
echo "Total packages: $(wc -l < all-packages.txt)" >> backup-info.txt
echo "User-installed packages: $(wc -l < userinstalled-packages.txt)" >> backup-info.txt

echo "Backup completed in ~/package-backup/"
```

### Restore Script for New System
```bash
#!/bin/bash

# Comprehensive restore script
BACKUP_DIR="~/package-backup"

# Check if backup directory exists
if [ ! -d "$BACKUP_DIR" ]; then
    echo "Backup directory not found!"
    exit 1
fi

cd "$BACKUP_DIR"

# 1. Enable repositories first (if you have custom repos)
echo "Setting up repositories..."
# You might need to manually add custom repositories

# 2. Update system first
sudo dnf update -y

# 3. Install user packages
echo "Installing user packages..."
if [ -f "userinstalled-packages.txt" ]; then
    while read package; do
        echo "Installing: $package"
        sudo dnf install -y "$package" 2>/dev/null || echo "Failed to install: $package"
    done < userinstalled-packages.txt
fi

# 4. Install Flatpak packages
if [ -f "flatpak-packages.txt" ] && command -v flatpak >/dev/null; then
    echo "Installing Flatpak packages..."
    while read app; do
        flatpak install -y "$app" 2>/dev/null || echo "Failed to install Flatpak: $app"
    done < flatpak-packages.txt
fi

# 5. Install Snap packages
if [ -f "snap-packages.txt" ] && command -v snap >/dev/null; then
    echo "Installing Snap packages..."
    tail -n +2 snap-packages.txt | while read snap version rev tracking publisher notes; do
        sudo snap install "$snap" 2>/dev/null || echo "Failed to install Snap: $snap"
    done
fi

echo "Restore completed!"
```

## Method 4: Using Third-Party Tools

### Using `dnf-utils` Package
```bash
# Install dnf-utils if not already installed
sudo dnf install dnf-utils

# Generate package list
dnf repoquery --userinstalled > user-installed.txt

# On new system, install from list
sudo dnf install $(cat user-installed.txt)
```

## Advanced Backup Options

### Include Configuration Files
```bash
# Backup important configuration directories
tar -czf config-backup.tar.gz \
    ~/.config \
    ~/.bashrc \
    ~/.bash_profile \
    ~/.vimrc \
    /etc/dnf/dnf.conf \
    /etc/yum.repos.d/ 2>/dev/null

# List of important system config files to consider
echo "Consider backing up these config files:" > config-files-list.txt
echo "/etc/fstab" >> config-files-list.txt
echo "/etc/hosts" >> config-files-list.txt
echo "/etc/resolv.conf" >> config-files-list.txt
echo "/etc/ssh/sshd_config" >> config-files-list.txt
echo "~/.ssh/" >> config-files-list.txt
```

### Repository Backup and Restore
```bash
# Backup custom repositories
sudo tar -czf repos-backup.tar.gz /etc/yum.repos.d/

# On new system, restore repositories
sudo tar -xzf repos-backup.tar.gz -C /
sudo dnf makecache
```

## Automated Backup Script

### Complete Automated Solution
```bash
#!/bin/bash
# save as: backup-fedora-packages.sh

BACKUP_DIR="$HOME/fedora-package-backup-$(date +%Y-%m-%d)"
mkdir -p "$BACKUP_DIR"

echo "Creating Fedora package backup in $BACKUP_DIR"

# Package lists
dnf history userinstalled > "$BACKUP_DIR/userinstalled.txt"
dnf list installed > "$BACKUP_DIR/all-installed.txt"
rpm -qa > "$BACKUP_DIR/rpm-list.txt"

# Repository information
dnf repolist enabled > "$BACKUP_DIR/enabled-repos.txt"
sudo cp -r /etc/yum.repos.d/ "$BACKUP_DIR/yum-repos-backup/"

# Flatpak and Snap
flatpak list --app --columns=application > "$BACKUP_DIR/flatpak.txt" 2>/dev/null || touch "$BACKUP_DIR/flatpak.txt"
snap list > "$BACKUP_DIR/snap.txt" 2>/dev/null || touch "$BACKUP_DIR/snap.txt"

# System information
uname -a > "$BACKUP_DIR/system-info.txt"
cat /etc/os-release >> "$BACKUP_DIR/system-info.txt"

# Create restore script
cat > "$BACKUP_DIR/restore.sh" << 'EOF'
#!/bin/bash
echo "Restoring Fedora packages..."
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Update system
sudo dnf update -y

# Install packages
echo "Installing user packages..."
while read package; do
    sudo dnf install -y "$package" || echo "Failed: $package"
done < "$SCRIPT_DIR/userinstalled.txt"

echo "Restoration complete!"
EOF

chmod +x "$BACKUP_DIR/restore.sh"

echo "Backup completed in: $BACKUP_DIR"
echo "To restore on new system:"
echo "1. Copy the backup directory to new system"
echo "2. Run: cd $BACKUP_DIR && ./restore.sh"
```

## Best Practices

### Before Running Backup
1. **Clean your system**: Remove unnecessary packages
```bash
sudo dnf autoremove
sudo dnf clean all
```

2. **Update package database**:
```bash
sudo dnf makecache
```

### During Restore
1. **Update system first**:
```bash
sudo dnf update -y
```

2. **Install in batches**: Don't install all packages at once if the list is very large

3. **Handle failures gracefully**: Some packages might not be available or have dependency conflicts

### Verification
```bash
# After restore, verify installation
dnf list installed | wc -l  # Count installed packages
dnf check  # Check for problems
```

## Quick Commands Summary

```bash
# Quick backup (one-liner) - WORKING VERSION
dnf repoquery --userinstalled --qf "%{name}" > my-packages.txt

# Alternative quick backup using RPM
rpm -qa --qf "%{NAME}\n" > my-packages.txt

# Quick restore (one-liner)
sudo dnf install $(cat my-packages.txt)

# Include Flatpak in one command
(dnf repoquery --userinstalled --qf "%{name}"; flatpak list --app --columns=application) > all-my-packages.txt
```

## Troubleshooting

### Common Issues and Solutions

1. **Package not found**: Package might be from a disabled repository
```bash
dnf search package_name
dnf provides package_name
```

2. **Dependency conflicts**: Install packages individually
```bash
while read package; do sudo dnf install -y "$package"; done < package-list.txt
```

3. **Different Fedora versions**: Some packages might not be available in newer/older versions
```bash
dnf list available | grep package_name
```
