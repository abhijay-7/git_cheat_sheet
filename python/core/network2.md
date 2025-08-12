# Advanced Python Network Modules Cheatsheet

## 1. Scapy Module

### Installation & Basic Import
```bash
pip install scapy
```

```python
from scapy.all import *
```

### Packet Creation & Manipulation
```python
# Create basic IP packet
packet = IP(dst="8.8.8.8")
print(packet.show())

# Create ICMP packet (ping)
icmp_packet = IP(dst="google.com")/ICMP()

# Create TCP packet
tcp_packet = IP(dst="example.com")/TCP(dport=80, flags="S")

# Create UDP packet
udp_packet = IP(dst="8.8.8.8")/UDP(dport=53)/DNS(rd=1, qd=DNSQR(qname="google.com"))

# Layer manipulation
packet[IP].ttl = 64
packet[TCP].seq = 1000
```

### Sending Packets
```python
# Send packet (Layer 3)
send(IP(dst="8.8.8.8")/ICMP())

# Send and receive (Layer 3)
response = sr1(IP(dst="8.8.8.8")/ICMP(), timeout=2)
if response:
    response.show()

# Send packet (Layer 2)
sendp(Ether()/IP(dst="8.8.8.8")/ICMP())

# Send multiple packets
packets = [IP(dst=f"192.168.1.{i}")/ICMP() for i in range(1, 255)]
ans, unans = sr(packets, timeout=2)
```

### Packet Sniffing
```python
# Basic sniffing
packets = sniff(count=10)
packets.summary()

# Sniff with filter
packets = sniff(filter="tcp port 80", count=10)

# Sniff with callback
def packet_callback(packet):
    if packet.haslayer(TCP):
        print(f"TCP packet from {packet[IP].src} to {packet[IP].dst}")

sniff(prn=packet_callback, filter="tcp", count=10)

# Sniff on specific interface
sniff(iface="eth0", count=10)
```

### Network Scanning
```python
# Ping sweep
def ping_sweep(network):
    ans, unans = sr(IP(dst=network)/ICMP(), timeout=2, verbose=0)
    for send, recv in ans:
        print(f"Host {recv.src} is alive")

ping_sweep("192.168.1.1/24")

# Port scan
def port_scan(target, ports):
    ans, unans = sr(IP(dst=target)/TCP(dport=ports, flags="S"), timeout=2, verbose=0)
    for send, recv in ans:
        if recv.haslayer(TCP) and recv[TCP].flags == 18:  # SYN-ACK
            print(f"Port {send[TCP].dport} is open")

port_scan("example.com", [22, 80, 443, 8080])

# ARP scan
def arp_scan(network):
    ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst=network), 
                     timeout=2, verbose=0)
    for send, recv in ans:
        print(f"IP: {recv[ARP].psrc} - MAC: {recv[ARP].hwsrc}")

arp_scan("192.168.1.0/24")
```

### Packet Analysis
```python
# Read from pcap file
packets = rdpcap("capture.pcap")

# Write to pcap file
wrpcap("output.pcap", packets)

# Packet filtering
tcp_packets = [pkt for pkt in packets if pkt.haslayer(TCP)]
http_packets = [pkt for pkt in packets if pkt.haslayer(TCP) and pkt[TCP].dport == 80]

# Extract data
for packet in packets:
    if packet.haslayer(Raw):
        print(packet[Raw].load.decode(errors='ignore'))
```

---

## 2. netaddr Module

### Installation & Import
```bash
pip install netaddr
```

```python
from netaddr import IPAddress, IPNetwork, IPRange, IPSet
```

### IP Address Operations
```python
# Create IP address objects
ip4 = IPAddress('192.168.1.100')
ip6 = IPAddress('2001:db8::1')

# IP address properties
print(ip4.version)          # 4
print(ip4.is_private())     # True
print(ip4.is_loopback())    # False
print(ip4.is_multicast())   # False

# IP address arithmetic
next_ip = ip4 + 1           # 192.168.1.101
prev_ip = ip4 - 1           # 192.168.1.99

# Format conversions
print(ip4.bin)              # Binary representation
print(ip4.hex)              # Hexadecimal
print(int(ip4))             # Integer representation
```

### Network Operations
```python
# Create network objects
network = IPNetwork('192.168.1.0/24')
network6 = IPNetwork('2001:db8::/32')

# Network properties
print(network.ip)           # Network address
print(network.netmask)      # Subnet mask
print(network.broadcast)    # Broadcast address
print(network.hostmask)     # Host mask
print(network.size)         # Number of addresses
print(network.prefixlen)    # Prefix length

# Check if IP is in network
print(IPAddress('192.168.1.50') in network)  # True

# Iterate through network
for ip in network:
    print(ip)
    if ip == network.ip + 10:  # Stop after 10 IPs
        break

# Subnetting
subnets = list(network.subnet(26))  # Split into /26 networks
for subnet in subnets:
    print(subnet)
```

### IP Ranges and Sets
```python
# IP Range
ip_range = IPRange('192.168.1.10', '192.168.1.100')
print(len(ip_range))        # Number of IPs in range

# IP Set (for efficient operations on multiple networks)
ip_set = IPSet(['192.168.1.0/24', '10.0.0.0/8', '172.16.0.0/12'])

# Set operations
ip_set.add('203.0.113.0/24')
ip_set.remove('172.16.0.0/12')

# Check membership
print(IPAddress('192.168.1.50') in ip_set)

# Set arithmetic
set1 = IPSet(['192.168.1.0/24'])
set2 = IPSet(['192.168.1.128/25'])
intersection = set1 & set2
union = set1 | set2
difference = set1 - set2
```

### CIDR Operations
```python
# CIDR calculations
def cidr_to_netmask(cidr):
    network = IPNetwork(f'0.0.0.0/{cidr}')
    return str(network.netmask)

def netmask_to_cidr(netmask):
    return IPAddress(netmask).netmask_bits()

# Supernetting
networks = [IPNetwork('192.168.1.0/25'), IPNetwork('192.168.1.128/25')]
supernet = cidr_merge(networks)[0]
print(supernet)  # 192.168.1.0/24
```

---

## 3. pysnmp Module

### Installation & Import
```bash
pip install pysnmp
```

```python
from pysnmp.hlapi import *
```

### SNMP GET Operations
```python
# Simple SNMP GET
def snmp_get(target, oid, community='public', port=161):
    for (errorIndication, errorStatus, errorIndex, varBinds) in nextCmd(
        SnmpEngine(),
        CommunityData(community),
        UdpTransportTarget((target, port)),
        ContextData(),
        ObjectType(ObjectIdentity(oid)),
        lexicographicMode=False,
        ignoreNonIncreasingOid=True):
        
        if errorIndication:
            print(errorIndication)
            break
        elif errorStatus:
            print('%s at %s' % (errorStatus.prettyPrint(),
                                errorIndex and varBinds[int(errorIndex) - 1][0] or '?'))
            break
        else:
            for varBind in varBinds:
                print(' = '.join([x.prettyPrint() for x in varBind]))

# Get system description
snmp_get('192.168.1.1', '1.3.6.1.2.1.1.1.0')
```

### Common SNMP OIDs
```python
# System OIDs
SYSTEM_OIDs = {
    'sysDescr': '1.3.6.1.2.1.1.1.0',
    'sysObjectID': '1.3.6.1.2.1.1.2.0',
    'sysUpTime': '1.3.6.1.2.1.1.3.0',
    'sysContact': '1.3.6.1.2.1.1.4.0',
    'sysName': '1.3.6.1.2.1.1.5.0',
    'sysLocation': '1.3.6.1.2.1.1.6.0'
}

# Interface OIDs
INTERFACE_OIDs = {
    'ifNumber': '1.3.6.1.2.1.2.1.0',
    'ifDescr': '1.3.6.1.2.1.2.2.1.2',
    'ifType': '1.3.6.1.2.1.2.2.1.3',
    'ifMtu': '1.3.6.1.2.1.2.2.1.4',
    'ifSpeed': '1.3.6.1.2.1.2.2.1.5',
    'ifPhysAddress': '1.3.6.1.2.1.2.2.1.6',
    'ifAdminStatus': '1.3.6.1.2.1.2.2.1.7',
    'ifOperStatus': '1.3.6.1.2.1.2.2.1.8'
}

def get_system_info(target):
    for name, oid in SYSTEM_OIDs.items():
        print(f"{name}:")
        snmp_get(target, oid)
        print("-" * 40)
```

### SNMP Walk
```python
def snmp_walk(target, oid, community='public'):
    for (errorIndication, errorStatus, errorIndex, varBinds) in nextCmd(
        SnmpEngine(),
        CommunityData(community),
        UdpTransportTarget((target, 161)),
        ContextData(),
        ObjectType(ObjectIdentity(oid)),
        lexicographicMode=False,
        ignoreNonIncreasingOid=True):
        
        if errorIndication:
            print(errorIndication)
            break
        elif errorStatus:
            print('%s at %s' % (errorStatus.prettyPrint(),
                                errorIndex and varBinds[int(errorIndex) - 1][0] or '?'))
            break
        else:
            for varBind in varBinds:
                print(' = '.join([x.prettyPrint() for x in varBind]))

# Walk interface table
snmp_walk('192.168.1.1', '1.3.6.1.2.1.2.2.1.2')  # Interface descriptions
```

### SNMP SET Operations
```python
def snmp_set(target, oid, value, community='private'):
    for (errorIndication, errorStatus, errorIndex, varBinds) in setCmd(
        SnmpEngine(),
        CommunityData(community),
        UdpTransportTarget((target, 161)),
        ContextData(),
        ObjectType(ObjectIdentity(oid), value)):
        
        if errorIndication:
            print(errorIndication)
        elif errorStatus:
            print('%s at %s' % (errorStatus.prettyPrint(),
                                errorIndex and varBinds[int(errorIndex) - 1][0] or '?'))
        else:
            for varBind in varBinds:
                print(' = '.join([x.prettyPrint() for x in varBind]))

# Set system contact
snmp_set('192.168.1.1', '1.3.6.1.2.1.1.4.0', 'admin@company.com')
```

---

## 4. SSL Module

### Basic SSL Context
```python
import ssl
import socket

# Create SSL context
context = ssl.create_default_context()

# Connect to HTTPS server
with socket.create_connection(('www.google.com', 443)) as sock:
    with context.wrap_socket(sock, server_hostname='www.google.com') as ssock:
        print(f"SSL version: {ssock.version()}")
        print(f"Cipher: {ssock.cipher()}")
        
        # Send HTTP request
        ssock.send(b'GET / HTTP/1.1\r\nHost: www.google.com\r\n\r\n')
        response = ssock.recv(1024)
        print(response.decode())
```

### SSL Server
```python
import ssl
import socket

# Create SSL context for server
context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER)
context.load_cert_chain('server.pem', 'server.key')

# Create server socket
with socket.socket(socket.AF_INET, socket.SOCK_STREAM, 0) as sock:
    sock.bind(('127.0.0.1', 8443))
    sock.listen(5)
    
    with context.wrap_socket(sock, server_side=True) as ssock:
        while True:
            client_sock, addr = ssock.accept()
            print(f"Connection from {addr}")
            
            data = client_sock.recv(1024)
            print(f"Received: {data.decode()}")
            
            client_sock.send(b'HTTP/1.1 200 OK\r\n\r\nHello HTTPS!')
            client_sock.close()
```

### Certificate Verification
```python
import ssl
import socket

def get_certificate_info(hostname, port=443):
    context = ssl.create_default_context()
    
    with socket.create_connection((hostname, port)) as sock:
        with context.wrap_socket(sock, server_hostname=hostname) as ssock:
            cert = ssock.getpeercert()
            
            print(f"Subject: {cert['subject']}")
            print(f"Issuer: {cert['issuer']}")
            print(f"Version: {cert['version']}")
            print(f"Serial Number: {cert['serialNumber']}")
            print(f"Not Before: {cert['notBefore']}")
            print(f"Not After: {cert['notAfter']}")
            
            # Subject Alternative Names
            if 'subjectAltName' in cert:
                print("Subject Alternative Names:")
                for name in cert['subjectAltName']:
                    print(f"  {name[0]}: {name[1]}")

get_certificate_info('google.com')
```

### Custom SSL Context
```python
import ssl

# Create custom SSL context
context = ssl.SSLContext(ssl.PROTOCOL_TLS_CLIENT)

# Set verification mode
context.check_hostname = False
context.verify_mode = ssl.CERT_NONE

# Load custom CA certificates
context.load_verify_locations('ca-bundle.crt')

# Set ciphers
context.set_ciphers('ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS')

# Client certificate authentication
context.load_cert_chain('client.pem', 'client.key')
```

---

## 5. Select Module

### Basic Select Usage
```python
import select
import socket

# Create server socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(('localhost', 8080))
server.listen(5)
server.setblocking(False)

# Lists to track sockets
inputs = [server]
outputs = []
message_queues = {}

while inputs:
    # Wait for at least one socket to be ready
    readable, writable, exceptional = select.select(inputs, outputs, inputs, 1.0)
    
    # Handle readable sockets
    for sock in readable:
        if sock is server:
            # New connection
            connection, client_address = sock.accept()
            connection.setblocking(False)
            inputs.append(connection)
            message_queues[connection] = []
            print(f"New connection from {client_address}")
        else:
            # Existing connection
            try:
                data = sock.recv(1024)
                if data:
                    print(f"Received: {data.decode()}")
                    message_queues[sock].append(data)
                    if sock not in outputs:
                        outputs.append(sock)
                else:
                    # Connection closed
                    if sock in outputs:
                        outputs.remove(sock)
                    inputs.remove(sock)
                    sock.close()
                    del message_queues[sock]
            except:
                # Handle error
                if sock in outputs:
                    outputs.remove(sock)
                inputs.remove(sock)
                sock.close()
                del message_queues[sock]
    
    # Handle writable sockets
    for sock in writable:
        try:
            if message_queues[sock]:
                next_msg = message_queues[sock].pop(0)
                sock.send(next_msg)
            else:
                outputs.remove(sock)
        except:
            outputs.remove(sock)
    
    # Handle exceptional sockets
    for sock in exceptional:
        inputs.remove(sock)
        if sock in outputs:
            outputs.remove(sock)
        sock.close()
        del message_queues[sock]
```

### Poll Object
```python
import select
import socket

# Create poll object
poll_obj = select.poll()

# Create server socket
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('localhost', 8080))
server.listen(5)

# Register server socket
poll_obj.register(server.fileno(), select.POLLIN)

fd_to_socket = {server.fileno(): server}

while True:
    # Poll for events
    events = poll_obj.poll(1000)  # 1 second timeout
    
    for fd, event in events:
        sock = fd_to_socket[fd]
        
        if event & select.POLLIN:
            if sock is server:
                # New connection
                connection, address = server.accept()
                connection.setblocking(False)
                fd_to_socket[connection.fileno()] = connection
                poll_obj.register(connection.fileno(), select.POLLIN)
                print(f"Connection from {address}")
            else:
                # Data available
                data = sock.recv(1024)
                if data:
                    print(f"Received: {data.decode()}")
                    # Echo back
                    sock.send(data)
                else:
                    # Connection closed
                    poll_obj.unregister(fd)
                    del fd_to_socket[fd]
                    sock.close()
        
        elif event & select.POLLHUP:
            # Connection closed
            poll_obj.unregister(fd)
            del fd_to_socket[fd]
            sock.close()
```

---

## 6. DNS Module (dnspython)

### Installation & Import
```bash
pip install dnspython
```

```python
import dns.resolver
import dns.reversename
import dns.zone
import dns.query
```

### Basic DNS Queries
```python
# A record lookup
def lookup_a_record(domain):
    try:
        result = dns.resolver.resolve(domain, 'A')
        for ipval in result:
            print(f"A record for {domain}: {ipval.to_text()}")
    except dns.resolver.NXDOMAIN:
        print(f"Domain {domain} does not exist")
    except Exception as e:
        print(f"Error: {e}")

lookup_a_record('google.com')

# Multiple record types
def dns_lookup(domain, record_type):
    try:
        answers = dns.resolver.resolve(domain, record_type)
        print(f"{record_type} records for {domain}:")
        for rdata in answers:
            print(f"  {rdata}")
    except Exception as e:
        print(f"Error looking up {record_type} for {domain}: {e}")

# Common record types
record_types = ['A', 'AAAA', 'MX', 'NS', 'TXT', 'CNAME']
domain = 'google.com'

for record_type in record_types:
    dns_lookup(domain, record_type)
    print("-" * 40)
```

### Reverse DNS Lookup
```python
def reverse_dns(ip_address):
    try:
        # Create reverse name
        rev_name = dns.reversename.from_address(ip_address)
        
        # Perform reverse lookup
        result = dns.resolver.resolve(rev_name, "PTR")
        
        for ptr in result:
            print(f"Reverse DNS for {ip_address}: {ptr}")
            
    except Exception as e:
        print(f"Reverse DNS lookup failed: {e}")

reverse_dns('8.8.8.8')
reverse_dns('2001:4860:4860::8888')  # IPv6
```

### Custom DNS Server
```python
def query_specific_server(domain, record_type, server):
    try:
        # Create resolver with specific nameserver
        resolver = dns.resolver.Resolver()
        resolver.nameservers = [server]
        
        answers = resolver.resolve(domain, record_type)
        print(f"Querying {server} for {record_type} records of {domain}:")
        for rdata in answers:
            print(f"  {rdata}")
            
    except Exception as e:
        print(f"Error: {e}")

# Query different DNS servers
servers = ['8.8.8.8', '1.1.1.1', '208.67.222.222']
for server in servers:
    query_specific_server('google.com', 'A', server)
    print("-" * 40)
```

### DNS Zone Transfer
```python
def zone_transfer(domain, nameserver):
    try:
        # Get SOA record first
        soa_answer = dns.resolver.resolve(domain, 'SOA')
        
        # Attempt zone transfer
        zone = dns.zone.from_xfr(dns.query.xfr(nameserver, domain))
        
        print(f"Zone transfer for {domain} from {nameserver}:")
        for name, node in zone.nodes.items():
            for rdataset in node.rdatasets:
                print(f"{name}.{domain} {rdataset}")
                
    except Exception as e:
        print(f"Zone transfer failed: {e}")

# Note: Most servers don't allow zone transfers for security reasons
zone_transfer('example.com', 'ns1.example.com')
```

### DNS Record Creation
```python
import dns.rrset
import dns.rdata
import dns.rdataclass
import dns.rdatatype

def create_dns_records():
    # Create A record
    a_record = dns.rrset.from_text('example.com', 300, 'IN', 'A', '192.0.2.1')
    print(a_record)
    
    # Create MX record
    mx_record = dns.rrset.from_text('example.com', 300, 'IN', 'MX', '10 mail.example.com')
    print(mx_record)
    
    # Create TXT record
    txt_record = dns.rrset.from_text('example.com', 300, 'IN', 'TXT', '"v=spf1 include:_spf.example.com ~all"')
    print(txt_record)

create_dns_records()
```

---

## Common Patterns & Integration Examples

### Network Scanner with Multiple Tools
```python
from scapy.all import *
from netaddr import IPNetwork
import dns.resolver

def comprehensive_scan(network):
    net = IPNetwork(network)
    
    print(f"Scanning network: {network}")
    print("=" * 50)
    
    # ARP scan for live hosts
    live_hosts = []
    ans, unans = srp(Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(pdst=str(net)), 
                     timeout=2, verbose=0)
    
    for send, recv in ans:
        ip = recv[ARP].psrc
        mac = recv[ARP].hwsrc
        live_hosts.append(ip)
        
        # Try reverse DNS lookup
        try:
            hostname = dns.resolver.resolve(dns.reversename.from_address(ip), "PTR")[0]
        except:
            hostname = "No PTR record"
        
        print(f"Host: {ip} - MAC: {mac} - Hostname: {hostname}")
    
    return live_hosts
```

### SSL Certificate Monitor
```python
import ssl
import socket
import datetime

def check_ssl_expiry(hostname, port=443, days_warning=30):
    try:
        context = ssl.create_default_context()
        with socket.create_connection((hostname, port), timeout=10) as sock:
            with context.wrap_socket(sock, server_hostname=hostname) as ssock:
                cert = ssock.getpeercert()
                
                # Parse expiry date
                expiry_str = cert['notAfter']
                expiry_date = datetime.datetime.strptime(expiry_str, '%b %d %H:%M:%S %Y %Z')
                days_until_expiry = (expiry_date - datetime.datetime.now()).days
                
                if days_until_expiry <= days_warning:
                    status = "WARNING"
                else:
                    status = "OK"
                
                print(f"{hostname}: {status} - Expires in {days_until_expiry} days ({expiry_date})")
                
    except Exception as e:
        print(f"Error checking {hostname}: {e}")

# Monitor multiple sites
sites = ['google.com', 'github.com', 'stackoverflow.com']
for site in sites:
    check_ssl_expiry(site)
```

## Quick Reference Table

| Module | Primary Use | Key Classes/Functions |
|--------|-------------|----------------------|
| scapy | Packet manipulation | `IP()`, `TCP()`, `sr1()`, `sniff()` |
| netaddr | IP address operations | `IPAddress()`, `IPNetwork()`, `IPSet()` |
| pysnmp | SNMP operations | `nextCmd()`, `setCmd()`, `CommunityData()` |
| ssl | SSL/TLS operations | `SSLContext()`, `wrap_socket()`, `create_default_context()` |
| select | I/O multiplexing | `select()`, `poll()`, `epoll()` |
| dns | DNS operations | `resolve()`, `reversename()`, `zone.from_xfr()` |