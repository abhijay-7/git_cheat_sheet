# Python Network Modules Cheatsheet

## 1. Socket Module

### Basic TCP Client
```python
import socket

# Create socket
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Connect to server
client_socket.connect(('localhost', 8080))

# Send data
client_socket.send(b'Hello Server')

# Receive data
response = client_socket.recv(1024)
print(response.decode())

# Close connection
client_socket.close()
```

### Basic TCP Server
```python
import socket

# Create socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Bind to address
server_socket.bind(('localhost', 8080))

# Listen for connections
server_socket.listen(5)
print("Server listening on port 8080")

while True:
    # Accept connection
    client_socket, address = server_socket.accept()
    print(f"Connection from {address}")
    
    # Receive data
    data = client_socket.recv(1024)
    print(f"Received: {data.decode()}")
    
    # Send response
    client_socket.send(b'Hello Client')
    
    # Close client connection
    client_socket.close()
```

### UDP Socket
```python
import socket

# UDP Client
udp_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
udp_socket.sendto(b'UDP Message', ('localhost', 8080))
response, server = udp_socket.recvfrom(1024)
udp_socket.close()

# UDP Server
server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_socket.bind(('localhost', 8080))
data, client_address = server_socket.recvfrom(1024)
server_socket.sendto(b'UDP Response', client_address)
```

### Socket Options
```python
# Set socket options
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.settimeout(10)  # 10 second timeout
sock.setblocking(False)  # Non-blocking mode
```

---

## 2. urllib Module

### urllib.request - Making HTTP Requests
```python
import urllib.request
import urllib.parse

# Simple GET request
response = urllib.request.urlopen('http://httpbin.org/get')
data = response.read()
print(data.decode())

# GET with parameters
params = urllib.parse.urlencode({'key1': 'value1', 'key2': 'value2'})
url = f'http://httpbin.org/get?{params}'
response = urllib.request.urlopen(url)

# POST request
post_data = urllib.parse.urlencode({'name': 'John', 'age': 30}).encode()
request = urllib.request.Request('http://httpbin.org/post', data=post_data)
response = urllib.request.urlopen(request)

# With headers
headers = {'User-Agent': 'Python Client', 'Content-Type': 'application/json'}
request = urllib.request.Request('http://httpbin.org/headers', headers=headers)
response = urllib.request.urlopen(request)
```

### urllib.parse - URL Parsing
```python
import urllib.parse

# Parse URL
parsed = urllib.parse.urlparse('https://example.com/path?query=value#fragment')
print(parsed.scheme)    # https
print(parsed.netloc)    # example.com
print(parsed.path)      # /path
print(parsed.query)     # query=value

# URL encoding/decoding
encoded = urllib.parse.quote('hello world')  # hello%20world
decoded = urllib.parse.unquote('hello%20world')  # hello world

# Query string handling
params = {'name': 'John Doe', 'city': 'New York'}
query_string = urllib.parse.urlencode(params)
```

### Error Handling
```python
import urllib.error

try:
    response = urllib.request.urlopen('http://httpbin.org/status/404')
except urllib.error.HTTPError as e:
    print(f"HTTP Error: {e.code} - {e.reason}")
except urllib.error.URLError as e:
    print(f"URL Error: {e.reason}")
```

---

## 3. smtplib Module

### Basic Email Sending
```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

# Create message
msg = MIMEMultipart()
msg['From'] = 'sender@example.com'
msg['To'] = 'recipient@example.com'
msg['Subject'] = 'Test Email'

# Add body
body = "This is a test email sent using Python!"
msg.attach(MIMEText(body, 'plain'))

# Connect and send
try:
    server = smtplib.SMTP('smtp.gmail.com', 587)
    server.starttls()  # Enable TLS
    server.login('your_email@gmail.com', 'your_password')
    
    text = msg.as_string()
    server.sendmail('sender@example.com', 'recipient@example.com', text)
    server.quit()
    print("Email sent successfully!")
    
except Exception as e:
    print(f"Error: {e}")
```

### HTML Email with Attachments
```python
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders
import os

msg = MIMEMultipart()
msg['From'] = 'sender@example.com'
msg['To'] = 'recipient@example.com'
msg['Subject'] = 'HTML Email with Attachment'

# HTML body
html_body = """
<html>
  <body>
    <h2>Hello from Python!</h2>
    <p>This is an <b>HTML</b> email.</p>
  </body>
</html>
"""
msg.attach(MIMEText(html_body, 'html'))

# Add attachment
filename = "document.pdf"
with open(filename, "rb") as attachment:
    part = MIMEBase('application', 'octet-stream')
    part.set_payload(attachment.read())

encoders.encode_base64(part)
part.add_header(
    'Content-Disposition',
    f'attachment; filename= {filename}',
)
msg.attach(part)
```

### Different SMTP Servers
```python
# Gmail
server = smtplib.SMTP('smtp.gmail.com', 587)

# Outlook/Hotmail
server = smtplib.SMTP('smtp.live.com', 587)

# Yahoo
server = smtplib.SMTP('smtp.mail.yahoo.com', 587)

# Local SMTP server
server = smtplib.SMTP('localhost', 25)
```

---

## 4. ftplib Module

### Basic FTP Operations
```python
import ftplib

# Connect to FTP server
ftp = ftplib.FTP('ftp.example.com')
ftp.login('username', 'password')  # or ftp.login() for anonymous

# List directory contents
ftp.dir()  # Detailed listing
files = ftp.nlst()  # Simple list
print(files)

# Change directory
ftp.cwd('/path/to/directory')
print(f"Current directory: {ftp.pwd()}")

# Upload file (STOR)
with open('local_file.txt', 'rb') as file:
    ftp.storbinary('STOR remote_file.txt', file)

# Download file (RETR)
with open('downloaded_file.txt', 'wb') as file:
    ftp.retrbinary('RETR remote_file.txt', file.write)

# Delete file
ftp.delete('remote_file.txt')

# Create/Remove directory
ftp.mkd('new_directory')
ftp.rmd('directory_to_remove')

# Close connection
ftp.quit()
```

### FTP with Context Manager
```python
import ftplib

with ftplib.FTP('ftp.example.com') as ftp:
    ftp.login('username', 'password')
    
    # Upload multiple files
    local_files = ['file1.txt', 'file2.txt', 'file3.txt']
    for filename in local_files:
        with open(filename, 'rb') as file:
            ftp.storbinary(f'STOR {filename}', file)
            print(f"Uploaded {filename}")
```

### Secure FTP (FTPS)
```python
import ftplib

# Explicit FTPS
ftps = ftplib.FTP_TLS('ftps.example.com')
ftps.login('username', 'password')
ftps.prot_p()  # Set up secure data connection

# Operations same as regular FTP
ftps.dir()
ftps.quit()
```

---

## 5. telnetlib Module

### Basic Telnet Connection
```python
import telnetlib

# Connect to telnet server
tn = telnetlib.Telnet('telnet.example.com', 23)

# Read until login prompt
tn.read_until(b"login: ")
tn.write(b"username\n")

# Read until password prompt
tn.read_until(b"Password: ")
tn.write(b"password\n")

# Send command
tn.write(b"ls -la\n")

# Read response
response = tn.read_until(b"$ ")
print(response.decode())

# Close connection
tn.close()
```

### Interactive Telnet Session
```python
import telnetlib
import sys

HOST = "localhost"
PORT = 23

tn = telnetlib.Telnet(HOST, PORT)

# Start interactive session
tn.interact()  # This will give you direct control

# Or automated interaction
def telnet_automation():
    tn = telnetlib.Telnet(HOST, PORT)
    
    # Wait for login
    tn.read_until(b"login: ")
    tn.write(b"admin\n")
    
    # Wait for password
    tn.read_until(b"Password: ")
    tn.write(b"admin123\n")
    
    # Execute commands
    commands = [b"show version\n", b"show interfaces\n", b"exit\n"]
    
    for cmd in commands:
        tn.write(cmd)
        response = tn.read_until(b"#")
        print(f"Command: {cmd.decode().strip()}")
        print(f"Response: {response.decode()}")
        print("-" * 50)
    
    tn.close()
```

### Telnet with Timeout
```python
import telnetlib

try:
    tn = telnetlib.Telnet('example.com', 23, timeout=10)
    
    # Read with timeout
    response = tn.read_until(b"login: ", timeout=5)
    
    if b"login:" in response:
        tn.write(b"username\n")
        # Continue with login process
    else:
        print("Login prompt not received within timeout")
        
except Exception as e:
    print(f"Telnet connection failed: {e}")
finally:
    if 'tn' in locals():
        tn.close()
```

---

## Common Patterns & Best Practices

### Error Handling
```python
import socket
import urllib.error
import smtplib
import ftplib

# Socket error handling
try:
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(('example.com', 80))
except socket.gaierror as e:
    print(f"Address resolution error: {e}")
except socket.error as e:
    print(f"Socket error: {e}")

# URL error handling
try:
    response = urllib.request.urlopen('http://example.com')
except urllib.error.HTTPError as e:
    print(f"HTTP {e.code}: {e.reason}")
except urllib.error.URLError as e:
    print(f"URL error: {e.reason}")

# SMTP error handling
try:
    server = smtplib.SMTP('smtp.gmail.com', 587)
except smtplib.SMTPConnectError:
    print("Failed to connect to SMTP server")
except smtplib.SMTPAuthenticationError:
    print("SMTP authentication failed")
```

### Context Managers
```python
# Socket context manager
import socket
import contextlib

@contextlib.contextmanager
def socket_connection(host, port):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        sock.connect((host, port))
        yield sock
    finally:
        sock.close()

# Usage
with socket_connection('example.com', 80) as sock:
    sock.send(b'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n')
    response = sock.recv(1024)
```

## Quick Reference

| Module | Primary Use | Key Functions |
|--------|-------------|---------------|
| socket | Low-level networking | `socket()`, `connect()`, `send()`, `recv()` |
| urllib | HTTP operations | `urlopen()`, `urlparse()`, `urlencode()` |
| smtplib | Email sending | `SMTP()`, `login()`, `sendmail()` |
| ftplib | File transfer | `FTP()`, `login()`, `storbinary()`, `retrbinary()` |
| telnetlib | Remote terminal access | `Telnet()`, `read_until()`, `write()` |