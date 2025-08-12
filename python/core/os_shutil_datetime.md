# Python os, shutil, datetime Modules Cheatsheet

## os Module - Operating System Interface

### File and Directory Operations

```python
import os

# Current working directory
os.getcwd()                           # Get current directory
os.chdir('/path/to/directory')        # Change directory
os.listdir('.')                       # List directory contents
os.listdir('/path')                   # List specific directory

# Directory operations
os.mkdir('new_folder')                # Create single directory
os.makedirs('path/to/nested/folder')  # Create nested directories
os.makedirs('path', exist_ok=True)    # Don't error if exists
os.rmdir('empty_folder')              # Remove empty directory
os.removedirs('path/to/empty')        # Remove empty nested directories

# File operations
os.remove('file.txt')                 # Delete file
os.unlink('file.txt')                 # Delete file (alias)
os.rename('old.txt', 'new.txt')       # Rename file/directory
os.replace('old.txt', 'new.txt')      # Replace file (cross-platform)

# Path information
os.path.exists('file.txt')            # Check if path exists
os.path.isfile('file.txt')            # Check if it's a file
os.path.isdir('folder')               # Check if it's a directory
os.path.islink('symlink')             # Check if it's a symbolic link
os.path.isabs('/absolute/path')       # Check if path is absolute

# Path manipulation
os.path.join('path', 'to', 'file')    # Join paths (cross-platform)
os.path.split('/path/to/file.txt')    # Split path: ('/path/to', 'file.txt')
os.path.dirname('/path/to/file.txt')  # Get directory: '/path/to'
os.path.basename('/path/to/file.txt') # Get filename: 'file.txt'
os.path.splitext('file.txt')          # Split extension: ('file', '.txt')
os.path.abspath('relative/path')      # Get absolute path
os.path.realpath('symlink')           # Resolve symbolic links
os.path.normpath('path//to/../file')  # Normalize path

# File information
os.path.getsize('file.txt')           # Get file size in bytes
os.path.getmtime('file.txt')          # Get modification time (timestamp)
os.path.getctime('file.txt')          # Get creation time (timestamp)
os.path.getatime('file.txt')          # Get access time (timestamp)

# File permissions and stats
os.stat('file.txt')                   # Get file stats
os.chmod('file.txt', 0o755)           # Change file permissions
os.access('file.txt', os.R_OK)        # Check read permission
os.access('file.txt', os.W_OK)        # Check write permission
os.access('file.txt', os.X_OK)        # Check execute permission
```

### Directory Walking and Globbing

```python
# Walk directory tree
for root, dirs, files in os.walk('/path'):
    for file in files:
        filepath = os.path.join(root, file)
        print(filepath)

# Get all Python files recursively
import glob
python_files = glob.glob('**/*.py', recursive=True)
text_files = glob.glob('*.txt')       # Files in current directory
config_files = glob.glob('config.*')  # Files with any extension
```

### Environment Variables

```python
# Environment variables
os.environ['PATH']                    # Get environment variable
os.environ.get('HOME', '/default')   # Get with default value
os.environ['NEW_VAR'] = 'value'      # Set environment variable
del os.environ['VAR_NAME']           # Delete environment variable

# Common environment variables
os.environ.get('HOME')               # User home directory (Unix)
os.environ.get('USERPROFILE')        # User profile (Windows)
os.environ.get('TEMP')               # Temporary directory
```

### Process and System Information

```python
# Process information
os.getpid()                          # Get current process ID
os.getppid()                         # Get parent process ID
os.getuid()                          # Get user ID (Unix)
os.getgid()                          # Get group ID (Unix)

# System information
os.name                              # OS name: 'posix', 'nt', 'java'
os.uname()                           # System information (Unix)
os.cpu_count()                       # Number of CPU cores

# Execute commands
os.system('ls -la')                  # Execute shell command
os.popen('ls -la').read()           # Execute and capture output
```

### Special Directories

```python
# Special directories
os.path.expanduser('~')              # Expand ~ to home directory
os.path.expandvars('$HOME/file')     # Expand environment variables
os.path.join(os.path.expanduser('~'), 'Documents')  # User documents

# Temporary directory
import tempfile
temp_dir = tempfile.gettempdir()     # Get temp directory
```

## shutil Module - High-Level File Operations

### File Operations

```python
import shutil

# Copy files
shutil.copy('src.txt', 'dst.txt')         # Copy file (permissions preserved)
shutil.copy2('src.txt', 'dst.txt')        # Copy with metadata (timestamps)
shutil.copyfile('src.txt', 'dst.txt')     # Copy file content only
shutil.copystat('src.txt', 'dst.txt')     # Copy file metadata only

# Copy directories
shutil.copytree('src_dir', 'dst_dir')     # Copy entire directory tree
shutil.copytree('src', 'dst', dirs_exist_ok=True)  # Don't error if dst exists

# Move/rename
shutil.move('src.txt', 'dst.txt')         # Move file or directory
shutil.move('file.txt', 'new_dir/')       # Move to directory

# Remove directories
shutil.rmtree('directory')                # Remove directory and contents
shutil.rmtree('dir', ignore_errors=True)  # Ignore errors during removal
```

### Advanced File Operations

```python
# Copy with custom behavior
def ignore_patterns(*patterns):
    return shutil.ignore_patterns(*patterns)

# Copy directory but ignore certain files
shutil.copytree('src', 'dst', 
               ignore=ignore_patterns('*.pyc', 'tmp*', '.git*'))

# Disk usage
total, used, free = shutil.disk_usage('/')  # Get disk usage in bytes

# Which command (find executable)
shutil.which('python')                    # Find python executable path
shutil.which('git')                       # Find git executable path
```

### Archive Operations

```python
# Create archives
shutil.make_archive('archive_name', 'zip', 'source_dir')     # Create zip
shutil.make_archive('backup', 'tar', 'project')             # Create tar
shutil.make_archive('data', 'gztar', 'data_dir')           # Create tar.gz

# Extract archives
shutil.unpack_archive('archive.zip', 'extract_to')          # Extract archive
shutil.unpack_archive('backup.tar.gz', 'restore')           # Extract tar.gz

# Get supported formats
shutil.get_archive_formats()             # List available archive formats
shutil.get_unpack_formats()              # List supported extract formats
```

## datetime Module - Date and Time Handling

### Basic Date and Time Objects

```python
from datetime import datetime, date, time, timedelta

# Current date and time
now = datetime.now()                     # Current local datetime
utc_now = datetime.utcnow()             # Current UTC datetime
today = date.today()                     # Current date
current_time = datetime.now().time()     # Current time

# Create specific dates and times
dt = datetime(2024, 12, 25, 14, 30, 0)  # Christmas 2024, 2:30 PM
d = date(2024, 12, 25)                  # Christmas date
t = time(14, 30, 0)                     # 2:30 PM

# Extract components
dt.year, dt.month, dt.day               # Date components
dt.hour, dt.minute, dt.second           # Time components
dt.microsecond                          # Microseconds
dt.weekday()                            # Monday=0, Sunday=6
dt.isoweekday()                         # Monday=1, Sunday=7
```

### String Formatting and Parsing

```python
# Format datetime to string
dt = datetime.now()
dt.strftime('%Y-%m-%d')                 # '2024-08-12'
dt.strftime('%Y-%m-%d %H:%M:%S')        # '2024-08-12 14:30:00'
dt.strftime('%A, %B %d, %Y')           # 'Monday, August 12, 2024'
dt.strftime('%Y-%m-%d %I:%M %p')        # '2024-08-12 02:30 PM'

# Parse string to datetime
datetime.strptime('2024-12-25', '%Y-%m-%d')
datetime.strptime('Dec 25, 2024', '%b %d, %Y')
datetime.strptime('2024-12-25 14:30:00', '%Y-%m-%d %H:%M:%S')

# ISO format
dt.isoformat()                          # '2024-08-12T14:30:00'
datetime.fromisoformat('2024-08-12T14:30:00')  # Parse ISO format
```

### Common Format Codes

```python
# Date format codes
'%Y'  # 4-digit year (2024)
'%y'  # 2-digit year (24)
'%m'  # Month as number (01-12)
'%B'  # Full month name (August)
'%b'  # Abbreviated month (Aug)
'%d'  # Day of month (01-31)
'%A'  # Full weekday name (Monday)
'%a'  # Abbreviated weekday (Mon)

# Time format codes
'%H'  # Hour 24-hour format (00-23)
'%I'  # Hour 12-hour format (01-12)
'%M'  # Minutes (00-59)
'%S'  # Seconds (00-59)
'%p'  # AM/PM
'%f'  # Microseconds

# Combined formats
'%c'  # Complete date/time
'%x'  # Date representation
'%X'  # Time representation
```

### Date Arithmetic with timedelta

```python
from datetime import timedelta

# Create timedelta objects
delta = timedelta(days=7)               # 7 days
delta = timedelta(hours=2, minutes=30)  # 2.5 hours
delta = timedelta(weeks=2, days=3)      # 2 weeks and 3 days

# Date arithmetic
now = datetime.now()
future = now + timedelta(days=30)       # 30 days from now
past = now - timedelta(weeks=2)         # 2 weeks ago
difference = future - now               # Returns timedelta object

# Working with timedelta
delta.days                              # Number of days
delta.seconds                           # Seconds (0-86399)
delta.total_seconds()                   # Total seconds as float

# Examples
tomorrow = date.today() + timedelta(days=1)
next_week = datetime.now() + timedelta(weeks=1)
one_hour_ago = datetime.now() - timedelta(hours=1)
```

### Timezone Handling

```python
from datetime import timezone, timedelta
import pytz  # Third-party library for better timezone support

# Basic timezone (UTC offset)
utc = timezone.utc
est = timezone(timedelta(hours=-5))     # UTC-5
dt_utc = datetime.now(utc)              # UTC datetime
dt_est = datetime.now(est)              # EST datetime

# Convert between timezones
dt_local = datetime.now()
dt_utc = dt_local.replace(tzinfo=timezone.utc)
dt_est = dt_utc.astimezone(est)

# Using pytz for better timezone support
eastern = pytz.timezone('US/Eastern')
pacific = pytz.timezone('US/Pacific')
dt_eastern = eastern.localize(datetime.now())
dt_pacific = dt_eastern.astimezone(pacific)
```

### Practical Examples

```python
# Age calculation
from datetime import date

def calculate_age(birth_date):
    today = date.today()
    return today.year - birth_date.year - ((today.month, today.day) < 
                                          (birth_date.month, birth_date.day))

# Working days calculation
def add_business_days(start_date, business_days):
    current = start_date
    while business_days > 0:
        current += timedelta(days=1)
        if current.weekday() < 5:  # Monday = 0, Friday = 4
            business_days -= 1
    return current

# Time elapsed formatting
def format_duration(start_time, end_time):
    duration = end_time - start_time
    hours, remainder = divmod(duration.total_seconds(), 3600)
    minutes, seconds = divmod(remainder, 60)
    return f"{int(hours):02d}:{int(minutes):02d}:{int(seconds):02d}"

# Parse various date formats
def flexible_date_parse(date_string):
    formats = [
        '%Y-%m-%d',
        '%m/%d/%Y',
        '%d/%m/%Y',
        '%Y-%m-%d %H:%M:%S',
        '%B %d, %Y'
    ]
    for fmt in formats:
        try:
            return datetime.strptime(date_string, fmt)
        except ValueError:
            continue
    raise ValueError(f"Unable to parse date: {date_string}")
```

### File Operations with Timestamps

```python
# Combining os, shutil, and datetime for file operations

# Get file modification time as datetime
import os
from datetime import datetime

file_mtime = os.path.getmtime('file.txt')
mod_datetime = datetime.fromtimestamp(file_mtime)
print(f"File modified: {mod_datetime.strftime('%Y-%m-%d %H:%M:%S')}")

# Backup files with timestamp
def backup_with_timestamp(filename):
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    backup_name = f"{filename}.backup.{timestamp}"
    shutil.copy2(filename, backup_name)
    return backup_name

# Find files modified in last N days
def find_recent_files(directory, days):
    cutoff = datetime.now() - timedelta(days=days)
    recent_files = []
    
    for root, dirs, files in os.walk(directory):
        for file in files:
            filepath = os.path.join(root, file)
            mod_time = datetime.fromtimestamp(os.path.getmtime(filepath))
            if mod_time > cutoff:
                recent_files.append(filepath)
    
    return recent_files

# Archive files by date
def archive_files_by_date(source_dir, archive_dir):
    for filename in os.listdir(source_dir):
        filepath = os.path.join(source_dir, filename)
        if os.path.isfile(filepath):
            mod_time = datetime.fromtimestamp(os.path.getmtime(filepath))
            date_folder = mod_time.strftime('%Y-%m')
            archive_path = os.path.join(archive_dir, date_folder)
            os.makedirs(archive_path, exist_ok=True)
            shutil.move(filepath, os.path.join(archive_path, filename))
```

### Quick Reference Summary

```python
# Common patterns for file operations with dates

# Check if file is newer than 7 days
file_age = datetime.now() - datetime.fromtimestamp(os.path.getmtime('file.txt'))
is_recent = file_age < timedelta(days=7)

# Create timestamped directories
timestamp_dir = datetime.now().strftime('backup_%Y%m%d_%H%M%S')
os.makedirs(timestamp_dir)

# Log file with rotation
log_file = f"app_{date.today().strftime('%Y%m%d')}.log"

# Cleanup old files
cutoff_date = datetime.now() - timedelta(days=30)
for file in os.listdir('.'):
    if os.path.isfile(file):
        file_date = datetime.fromtimestamp(os.path.getmtime(file))
        if file_date < cutoff_date:
            os.remove(file)
```