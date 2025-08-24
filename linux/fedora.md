# Comprehensive Linux CLI Cheatsheet (Fedora-focused)

## 1. System Information & Control

### System Info
```bash
# System information
uname -a                    # All system info (kernel, hostname, architecture)
uname -r                    # Kernel version
hostnamectl                 # Detailed hostname and system info
lsb_release -a             # Distribution info
cat /etc/os-release        # OS release information
uptime                     # System uptime and load
who                        # Currently logged in users
whoami                     # Current username
id                         # Current user and group IDs

# Hardware information
lscpu                      # CPU information
lshw                       # Detailed hardware info (needs sudo)
lsblk                      # Block devices (disks, partitions)
lsusb                      # USB devices
lspci                      # PCI devices
dmidecode                  # DMI/SMBIOS hardware info (needs sudo)
free -h                    # Memory usage (human readable)
df -h                      # Disk space usage
fdisk -l                   # Partition tables (needs sudo)
```

### Date & Time
```bash
date                       # Current date and time
timedatectl                # System time and timezone info
timedatectl set-timezone America/New_York  # Set timezone
hwclock                    # Hardware clock
cal                        # Calendar
cal 2024                   # Calendar for specific year
```

### Power Management
```bash
sudo shutdown -h now       # Shutdown immediately
sudo shutdown -r now       # Restart immediately
sudo shutdown -h +10       # Shutdown in 10 minutes
sudo reboot                # Restart system
sudo halt                  # Halt system
sudo poweroff              # Power off system
systemctl suspend          # Suspend system
systemctl hibernate        # Hibernate system
```

## 2. Process & Job Management

### Process Information
```bash
ps aux                     # All running processes (detailed)
ps -ef                     # All processes (different format)
ps -u username             # Processes by specific user
top                        # Real-time process monitor
htop                       # Enhanced process monitor
pgrep process_name         # Find process ID by name
pidof process_name         # Get PID of running process
pstree                     # Process tree view
jobs                       # Active jobs in current shell
```

### Process Control
```bash
kill PID                   # Terminate process by PID
kill -9 PID               # Force kill process
killall process_name       # Kill all instances of process
pkill process_name         # Kill process by name
nohup command &            # Run command immune to hangups
command &                  # Run command in background
fg                        # Bring background job to foreground
bg                        # Send job to background
Ctrl+Z                    # Suspend current process
Ctrl+C                    # Interrupt current process
```

### System Monitoring
```bash
watch command             # Repeat command every 2 seconds
watch -n 5 command        # Repeat command every 5 seconds
iostat                    # I/O statistics
vmstat                    # Virtual memory statistics
sar                       # System activity reporter
lsof                      # List open files
lsof -i                   # List network connections
fuser filename            # Show processes using file
```

## 3. File & Directory Management

### Navigation
```bash
pwd                       # Print working directory
cd /path/to/directory     # Change directory
cd ~                      # Go to home directory
cd -                      # Go to previous directory
cd ..                     # Go up one directory
cd ../..                  # Go up two directories
pushd /path               # Push directory to stack and cd
popd                      # Pop directory from stack and cd
dirs                      # Show directory stack
```

### Listing & Viewing
```bash
ls                        # List files and directories
ls -la                    # Long format with hidden files
ls -lh                    # Long format with human readable sizes
ls -lt                    # Sort by modification time
ls -lS                    # Sort by size
ls -R                     # Recursive listing
tree                      # Tree view of directories
tree -L 2                 # Limit tree depth to 2 levels
file filename             # Determine file type
stat filename             # Detailed file information
```

### File Operations
```bash
touch filename            # Create empty file or update timestamp
mkdir dirname             # Create directory
mkdir -p path/to/dir      # Create directory with parents
rmdir dirname             # Remove empty directory
rm filename               # Remove file
rm -r dirname             # Remove directory recursively
rm -rf dirname            # Force remove directory recursively
rm -i filename            # Interactive removal
cp source dest            # Copy file
cp -r source_dir dest_dir # Copy directory recursively
cp -p source dest         # Copy preserving attributes
mv source dest            # Move/rename file or directory
```

### File Content
```bash
cat filename              # Display file content
less filename             # View file with pagination
more filename             # View file page by page
head filename             # Show first 10 lines
head -n 20 filename       # Show first 20 lines
tail filename             # Show last 10 lines
tail -n 20 filename       # Show last 20 lines
tail -f filename          # Follow file changes (real-time)
grep pattern filename     # Search for pattern in file
grep -r pattern directory # Recursive grep
grep -i pattern filename  # Case insensitive search
grep -n pattern filename  # Show line numbers
```

### File Permissions
```bash
chmod 755 filename        # Set permissions (rwxr-xr-x)
chmod u+x filename        # Add execute for user
chmod g-w filename        # Remove write for group
chmod o=r filename        # Set read-only for others
chown user:group filename # Change ownership
chgrp group filename      # Change group
umask                     # View default permissions
umask 022                 # Set default permissions
```

### File Compression
```bash
tar -czf archive.tar.gz directory/    # Create gzip compressed tar
tar -xzf archive.tar.gz              # Extract gzip compressed tar
tar -cjf archive.tar.bz2 directory/  # Create bzip2 compressed tar
tar -xjf archive.tar.bz2             # Extract bzip2 compressed tar
zip -r archive.zip directory/        # Create zip archive
unzip archive.zip                    # Extract zip archive
gzip filename                        # Compress file with gzip
gunzip filename.gz                   # Decompress gzip file
```

## 4. Text Processing & Search

### Text Manipulation
```bash
sort filename             # Sort lines in file
sort -n filename          # Numeric sort
sort -r filename          # Reverse sort
uniq filename             # Remove duplicate lines
uniq -c filename          # Count occurrences
cut -d',' -f1 filename    # Cut first field (CSV)
cut -c1-10 filename       # Cut characters 1-10
awk '{print $1}' filename # Print first column
sed 's/old/new/g' filename # Replace all occurrences
sed -i 's/old/new/g' filename # In-place replacement
tr 'a-z' 'A-Z' < filename # Convert to uppercase
```

### Advanced Search
```bash
find /path -name "*.txt"  # Find files by name
find /path -type f        # Find files only
find /path -type d        # Find directories only
find /path -size +100M    # Find files larger than 100MB
find /path -mtime -7      # Find files modified in last 7 days
find /path -exec command {} \; # Execute command on found files
locate filename           # Fast file search (uses database)
updatedb                  # Update locate database
which command             # Find command location
whereis command           # Find command, source, manual
```

### Text Comparison
```bash
diff file1 file2          # Compare files line by line
diff -u file1 file2       # Unified diff format
vimdiff file1 file2       # Visual diff with vim
comm file1 file2          # Compare sorted files
cmp file1 file2           # Compare files byte by byte
```

## 5. Network Operations

### Network Information
```bash
ip addr show              # Show IP addresses
ip route show             # Show routing table
ifconfig                  # Network interface configuration (deprecated)
hostname                  # Display hostname
hostname -I               # Display IP addresses
ss -tuln                  # Show listening ports (modern netstat)
netstat -tuln             # Show listening ports (legacy)
ss -i                     # Show network interface statistics
arp -a                    # Show ARP table
```

### Network Testing
```bash
ping google.com           # Test connectivity
ping -c 4 google.com      # Ping 4 times
ping6 google.com          # Ping IPv6
traceroute google.com     # Trace route to destination
tracepath google.com      # Trace path (no root needed)
nslookup google.com       # DNS lookup
dig google.com            # DNS lookup (detailed)
host google.com           # DNS lookup (simple)
mtr google.com            # Continuous traceroute
```

### File Transfer
```bash
scp file user@host:/path  # Secure copy to remote
scp user@host:/path file  # Secure copy from remote
scp -r dir user@host:/path # Copy directory recursively
rsync -av source/ dest/   # Sync directories
rsync -av --delete source/ dest/ # Sync with deletion
wget http://example.com/file # Download file
curl -O http://example.com/file # Download file with curl
curl -X POST -d "data" url # POST request with data
```

### Network Services (Fedora-specific)
```bash
sudo systemctl status NetworkManager  # Network service status
sudo nmcli device show    # Show network devices
sudo nmcli connection show # Show connections
sudo nmcli connection up connection_name # Activate connection
sudo firewall-cmd --list-all # Show firewall rules
sudo firewall-cmd --add-service=ssh # Add service to firewall
sudo firewall-cmd --reload  # Reload firewall
```

## 6. Package Management (DNF - Fedora)

### Package Operations
```bash
sudo dnf update           # Update all packages
sudo dnf upgrade          # Upgrade all packages
sudo dnf install package  # Install package
sudo dnf remove package   # Remove package
sudo dnf autoremove       # Remove orphaned packages
sudo dnf clean all        # Clean package cache
sudo dnf check-update     # Check for updates
```

### Package Information
```bash
dnf search keyword        # Search for packages
dnf info package          # Package information
dnf list installed        # List installed packages
dnf list available        # List available packages
dnf provides */filename   # Find package providing file
dnf history               # Package transaction history
dnf history undo ID       # Undo transaction
```

### Repository Management
```bash
dnf repolist              # List enabled repositories
sudo dnf config-manager --add-repo URL # Add repository
sudo dnf config-manager --set-enabled repo # Enable repository
sudo dnf config-manager --set-disabled repo # Disable repository
```

## 7. System Services (systemd)

### Service Control
```bash
sudo systemctl start service    # Start service
sudo systemctl stop service     # Stop service
sudo systemctl restart service  # Restart service
sudo systemctl reload service   # Reload service config
sudo systemctl status service   # Service status
sudo systemctl enable service   # Enable at boot
sudo systemctl disable service  # Disable at boot
sudo systemctl is-active service # Check if active
sudo systemctl is-enabled service # Check if enabled
```

### Service Information
```bash
systemctl list-units         # List all units
systemctl list-units --failed # List failed units
systemctl list-unit-files    # List unit files
journalctl -u service        # View service logs
journalctl -f                # Follow system logs
journalctl --since "1 hour ago" # Logs from last hour
systemctl daemon-reload      # Reload systemd configuration
```

## 8. Environment & Variables

### Environment Variables
```bash
env                       # Show all environment variables
echo $PATH                # Show PATH variable
export VAR=value          # Set environment variable
unset VAR                 # Remove environment variable
printenv VAR              # Print specific variable
set                       # Show all variables (including shell)
```

### Shell Configuration
```bash
echo $SHELL               # Current shell
which bash                # Location of bash
history                   # Command history
history | grep command    # Search command history
!!                        # Repeat last command
!n                        # Repeat command number n
alias ll='ls -la'         # Create alias
unalias ll                # Remove alias
type command              # Show command type
```

## 9. Archive & Backup

### Backup Commands
```bash
rsync -av --progress source/ dest/ # Sync with progress
rsync -av --exclude='*.tmp' source/ dest/ # Sync excluding files
dd if=/dev/sda of=backup.img # Create disk image
dd if=/dev/sda1 of=partition.img # Backup partition
cpio -o < filelist > archive.cpio # Create cpio archive
find . -type f | cpio -o > archive.cpio # Archive current directory
```

### Mount Operations
```bash
mount                     # Show mounted filesystems
mount /dev/sdb1 /mnt      # Mount filesystem
umount /mnt               # Unmount filesystem
mount -o ro /dev/sdb1 /mnt # Mount read-only
lsblk -f                  # Show filesystem information
blkid                     # Show block device UUIDs
```

## 10. Performance & Monitoring

### System Resources
```bash
top                       # Process monitor
htop                      # Enhanced process monitor
iotop                     # I/O monitor
nethogs                   # Network usage per process
iftop                     # Network traffic monitor
atop                      # Advanced system monitor
glances                   # System overview
```

### Log Analysis
```bash
journalctl                # View systemd logs
journalctl -f             # Follow logs
journalctl -p err         # Error level logs only
journalctl --since yesterday # Logs since yesterday
tail -f /var/log/messages # Follow system log
dmesg                     # Kernel messages
dmesg | tail              # Recent kernel messages
last                      # Login history
lastlog                   # Last login per user
```

## 11. Useful Shortcuts & Tips

### Keyboard Shortcuts
```bash
Ctrl+C                    # Interrupt current command
Ctrl+Z                    # Suspend current command
Ctrl+D                    # End of input/logout
Ctrl+L                    # Clear screen
Ctrl+A                    # Move to beginning of line
Ctrl+E                    # Move to end of line
Ctrl+U                    # Delete from cursor to beginning
Ctrl+K                    # Delete from cursor to end
Ctrl+R                    # Reverse search history
Tab                       # Auto-complete
!!                        # Repeat last command
```

### Command Chaining
```bash
command1 && command2      # Run command2 if command1 succeeds
command1 || command2      # Run command2 if command1 fails
command1; command2        # Run both commands sequentially
command1 | command2       # Pipe output of command1 to command2
command > file            # Redirect output to file
command >> file           # Append output to file
command 2> file           # Redirect errors to file
command &> file           # Redirect both output and errors
```

### Special Variables
```bash
$?                        # Exit status of last command
$$                        # Current process ID
$!                        # PID of last background process
$0                        # Script name
$1, $2, etc.             # Script arguments
$#                        # Number of arguments
$@                        # All arguments
$*                        # All arguments as single string
```