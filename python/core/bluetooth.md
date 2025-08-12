# Python Bluetooth Modules Cheatsheet

## 1. PyBluez Module (Classic Bluetooth)

### Installation & Setup
```bash
# Linux
sudo apt-get install libbluetooth-dev
pip install pybluez

# Windows
pip install pybluez

# macOS
brew install bluetooth
pip install pybluez
```

### Basic Import
```python
import bluetooth
import time
```

### Device Discovery
```python
# Basic device discovery
def discover_devices():
    print("Scanning for Bluetooth devices...")
    nearby_devices = bluetooth.discover_devices(duration=8, lookup_names=True)
    
    if nearby_devices:
        print(f"Found {len(nearby_devices)} device(s):")
        for addr, name in nearby_devices:
            print(f"  Device: {name}")
            print(f"  Address: {addr}")
            print("-" * 40)
    else:
        print("No devices found")
    
    return nearby_devices

# Get device info
def get_device_info(addr):
    try:
        name = bluetooth.lookup_name(addr, timeout=10)
        print(f"Device name: {name}")
        
        # Get device class
        device_class = bluetooth.get_device_class(addr)
        print(f"Device class: {hex(device_class)}")
        
        return name
    except Exception as e:
        print(f"Error getting device info: {e}")
        return None

# Advanced discovery with device class filtering
def discover_with_filter():
    devices = bluetooth.discover_devices(lookup_names=True)
    for addr, name in devices:
        device_class = bluetooth.get_device_class(addr)
        
        # Filter by device type (example: phones, computers)
        if (device_class & 0x1F00) == 0x0200:  # Phone
            print(f"Phone found: {name} ({addr})")
        elif (device_class & 0x1F00) == 0x0100:  # Computer
            print(f"Computer found: {name} ({addr})")
```

### Service Discovery
```python
def find_services(addr):
    print(f"Searching for services on {addr}...")
    services = bluetooth.find_service(address=addr)
    
    if services:
        print(f"Found {len(services)} service(s):")
        for service in services:
            print(f"  Service: {service['name']}")
            print(f"  Host: {service['host']}")
            print(f"  Port: {service['port']}")
            print(f"  Protocol: {service['protocol']}")
            print(f"  Service ID: {service.get('service-id', 'N/A')}")
            print("-" * 40)
    else:
        print("No services found")
    
    return services

# Find specific service by UUID
def find_service_by_uuid(addr, uuid):
    services = bluetooth.find_service(address=addr, uuid=uuid)
    return services

# Common UUIDs
SERIAL_PORT_UUID = "00001101-0000-1000-8000-00805F9B34FB"
OBEX_UUID = "00001105-0000-1000-8000-00805f9b34fb"
```

### RFCOMM Communication (Serial-like)
```python
# RFCOMM Client
class BluetoothClient:
    def __init__(self):
        self.sock = None
    
    def connect(self, addr, port=1):
        try:
            self.sock = bluetooth.BluetoothSocket(bluetooth.RFCOMM)
            self.sock.connect((addr, port))
            print(f"Connected to {addr} on port {port}")
            return True
        except Exception as e:
            print(f"Connection failed: {e}")
            return False
    
    def send_data(self, data):
        if self.sock:
            try:
                if isinstance(data, str):
                    data = data.encode('utf-8')
                self.sock.send(data)
                print(f"Sent: {data}")
            except Exception as e:
                print(f"Send failed: {e}")
    
    def receive_data(self, buffer_size=1024):
        if self.sock:
            try:
                data = self.sock.recv(buffer_size)
                print(f"Received: {data.decode('utf-8')}")
                return data
            except Exception as e:
                print(f"Receive failed: {e}")
                return None
    
    def disconnect(self):
        if self.sock:
            self.sock.close()
            print("Disconnected")

# Usage example
client = BluetoothClient()
if client.connect("00:11:22:33:44:55"):
    client.send_data("Hello Bluetooth!")
    response = client.receive_data()
    client.disconnect()
```

### RFCOMM Server
```python
class BluetoothServer:
    def __init__(self, port=1):
        self.port = port
        self.server_sock = None
        self.running = False
    
    def start_server(self):
        try:
            self.server_sock = bluetooth.BluetoothSocket(bluetooth.RFCOMM)
            self.server_sock.bind(("", self.port))
            self.server_sock.listen(1)
            
            print(f"Server listening on port {self.port}")
            
            # Advertise service
            bluetooth.advertise_service(
                self.server_sock,
                "Python Bluetooth Server",
                service_id=SERIAL_PORT_UUID,
                service_classes=[SERIAL_PORT_UUID, bluetooth.SERIAL_PORT_CLASS],
                profiles=[bluetooth.SERIAL_PORT_PROFILE]
            )
            
            self.running = True
            self.accept_connections()
            
        except Exception as e:
            print(f"Server start failed: {e}")
    
    def accept_connections(self):
        while self.running:
            try:
                print("Waiting for connection...")
                client_sock, client_info = self.server_sock.accept()
                print(f"Accepted connection from {client_info}")
                
                # Handle client in separate thread for multiple connections
                self.handle_client(client_sock, client_info)
                
            except Exception as e:
                print(f"Connection error: {e}")
    
    def handle_client(self, client_sock, client_info):
        try:
            while True:
                data = client_sock.recv(1024)
                if not data:
                    break
                
                print(f"Received from {client_info}: {data.decode('utf-8')}")
                
                # Echo back
                client_sock.send(b"Echo: " + data)
                
        except Exception as e:
            print(f"Client handling error: {e}")
        finally:
            client_sock.close()
            print(f"Connection with {client_info} closed")
    
    def stop_server(self):
        self.running = False
        if self.server_sock:
            self.server_sock.close()

# Usage
server = BluetoothServer()
# server.start_server()  # This will block
```

### L2CAP Communication (Lower Level)
```python
# L2CAP Client
def l2cap_client(addr, psm=0x1001):
    sock = bluetooth.BluetoothSocket(bluetooth.L2CAP)
    try:
        sock.connect((addr, psm))
        
        # Send data
        sock.send(b"L2CAP message")
        
        # Receive data
        data = sock.recv(1024)
        print(f"Received: {data}")
        
    except Exception as e:
        print(f"L2CAP client error: {e}")
    finally:
        sock.close()

# L2CAP Server
def l2cap_server(psm=0x1001):
    server_sock = bluetooth.BluetoothSocket(bluetooth.L2CAP)
    try:
        server_sock.bind(("", psm))
        server_sock.listen(1)
        
        print(f"L2CAP server listening on PSM {psm}")
        
        client_sock, address = server_sock.accept()
        print(f"Accepted connection from {address}")
        
        data = client_sock.recv(1024)
        print(f"Received: {data}")
        
        client_sock.send(b"L2CAP response")
        client_sock.close()
        
    except Exception as e:
        print(f"L2CAP server error: {e}")
    finally:
        server_sock.close()
```

### File Transfer Example
```python
def send_file(addr, port, filepath):
    client = BluetoothClient()
    if client.connect(addr, port):
        try:
            with open(filepath, 'rb') as file:
                # Send filename first
                filename = filepath.split('/')[-1]
                client.send_data(f"FILENAME:{filename}")
                
                # Send file size
                file_size = len(file.read())
                file.seek(0)
                client.send_data(f"SIZE:{file_size}")
                
                # Send file content in chunks
                chunk_size = 1024
                bytes_sent = 0
                
                while bytes_sent < file_size:
                    chunk = file.read(chunk_size)
                    if not chunk:
                        break
                    
                    client.sock.send(chunk)
                    bytes_sent += len(chunk)
                    
                    print(f"Sent {bytes_sent}/{file_size} bytes")
                
                print("File transfer complete")
                
        except Exception as e:
            print(f"File transfer error: {e}")
        finally:
            client.disconnect()
```

---

## 2. Bleak Module (Bluetooth Low Energy)

### Installation
```bash
pip install bleak
```

### Basic Import
```python
import asyncio
from bleak import BleakScanner, BleakClient
import logging

# Enable debug logging
logging.basicConfig(level=logging.DEBUG)
```

### Device Discovery (BLE)
```python
# Basic BLE device discovery
async def discover_ble_devices():
    print("Scanning for BLE devices...")
    devices = await BleakScanner.discover(timeout=10.0)
    
    if devices:
        print(f"Found {len(devices)} BLE device(s):")
        for device in devices:
            print(f"  Name: {device.name or 'Unknown'}")
            print(f"  Address: {device.address}")
            print(f"  RSSI: {device.rssi} dBm")
            print(f"  Details: {device.details}")
            print("-" * 40)
    else:
        print("No BLE devices found")
    
    return devices

# Run discovery
# devices = asyncio.run(discover_ble_devices())

# Discovery with callback (real-time)
async def discovery_callback():
    def detection_callback(device, advertisement_data):
        print(f"Found: {device.name or 'Unknown'} ({device.address})")
        print(f"  RSSI: {device.rssi} dBm")
        print(f"  Services: {advertisement_data.service_uuids}")
        print(f"  Manufacturer: {advertisement_data.manufacturer_data}")
        print("-" * 30)
    
    scanner = BleakScanner(detection_callback)
    await scanner.start()
    await asyncio.sleep(10.0)  # Scan for 10 seconds
    await scanner.stop()

# Filter devices by name or service
async def find_specific_devices():
    devices = await BleakScanner.discover()
    
    # Filter by name
    heart_rate_devices = [d for d in devices if d.name and "heart" in d.name.lower()]
    
    # Filter by service UUID
    service_uuid = "0000180d-0000-1000-8000-00805f9b34fb"  # Heart Rate Service
    hr_service_devices = [d for d in devices if service_uuid in (d.metadata.get("uuids", []))]
    
    return heart_rate_devices
```

### BLE Client Operations
```python
# Basic BLE client
class BLEClient:
    def __init__(self, address):
        self.address = address
        self.client = None
        self.connected = False
    
    async def connect(self, timeout=10.0):
        try:
            self.client = BleakClient(self.address, timeout=timeout)
            await self.client.connect()
            self.connected = True
            print(f"Connected to {self.address}")
            return True
        except Exception as e:
            print(f"Connection failed: {e}")
            return False
    
    async def disconnect(self):
        if self.client and self.connected:
            await self.client.disconnect()
            self.connected = False
            print("Disconnected")
    
    async def get_services(self):
        if not self.connected:
            print("Not connected")
            return None
        
        services = self.client.services
        print("Available services:")
        
        for service in services:
            print(f"Service: {service.uuid} - {service.description}")
            for char in service.characteristics:
                print(f"  Characteristic: {char.uuid}")
                print(f"    Properties: {char.properties}")
                print(f"    Descriptors: {len(char.descriptors)}")
    
    async def read_characteristic(self, char_uuid):
        if not self.connected:
            print("Not connected")
            return None
        
        try:
            value = await self.client.read_gatt_char(char_uuid)
            print(f"Read {char_uuid}: {value}")
            return value
        except Exception as e:
            print(f"Read failed: {e}")
            return None
    
    async def write_characteristic(self, char_uuid, data, response=True):
        if not self.connected:
            print("Not connected")
            return False
        
        try:
            await self.client.write_gatt_char(char_uuid, data, response=response)
            print(f"Wrote to {char_uuid}: {data}")
            return True
        except Exception as e:
            print(f"Write failed: {e}")
            return False
    
    async def start_notify(self, char_uuid, callback):
        if not self.connected:
            print("Not connected")
            return False
        
        try:
            await self.client.start_notify(char_uuid, callback)
            print(f"Started notifications for {char_uuid}")
            return True
        except Exception as e:
            print(f"Start notify failed: {e}")
            return False
    
    async def stop_notify(self, char_uuid):
        if not self.connected:
            return
        
        try:
            await self.client.stop_notify(char_uuid)
            print(f"Stopped notifications for {char_uuid}")
        except Exception as e:
            print(f"Stop notify failed: {e}")

# Usage example
async def ble_client_example():
    device_address = "00:11:22:33:44:55"  # Replace with actual address
    
    client = BLEClient(device_address)
    
    if await client.connect():
        await client.get_services()
        
        # Example: Read battery level
        battery_char = "00002a19-0000-1000-8000-00805f9b34fb"
        battery_level = await client.read_characteristic(battery_char)
        
        if battery_level:
            print(f"Battery level: {int.from_bytes(battery_level, 'little')}%")
        
        await client.disconnect()
```

### Notifications and Indications
```python
# Handle BLE notifications
async def notification_example():
    device_address = "00:11:22:33:44:55"
    
    def notification_handler(characteristic, data):
        """Handle incoming notifications"""
        print(f"Notification from {characteristic.uuid}:")
        print(f"  Raw data: {data}")
        print(f"  Hex: {data.hex()}")
        
        # Example: Parse heart rate measurement
        if characteristic.uuid == "00002a37-0000-1000-8000-00805f9b34fb":
            if len(data) >= 2:
                hr_value = int.from_bytes(data[1:3], 'little')
                print(f"  Heart rate: {hr_value} BPM")
    
    client = BLEClient(device_address)
    
    if await client.connect():
        # Start heart rate notifications
        hr_char = "00002a37-0000-1000-8000-00805f9b34fb"
        await client.start_notify(hr_char, notification_handler)
        
        # Listen for 30 seconds
        print("Listening for notifications...")
        await asyncio.sleep(30)
        
        await client.stop_notify(hr_char)
        await client.disconnect()
```

### Common BLE Services and Characteristics
```python
# Standard BLE UUIDs
STANDARD_SERVICES = {
    "heart_rate": "0000180d-0000-1000-8000-00805f9b34fb",
    "battery_service": "0000180f-0000-1000-8000-00805f9b34fb",
    "device_information": "0000180a-0000-1000-8000-00805f9b34fb",
    "generic_access": "00001800-0000-1000-8000-00805f9b34fb",
    "generic_attribute": "00001801-0000-1000-8000-00805f9b34fb",
}

STANDARD_CHARACTERISTICS = {
    "heart_rate_measurement": "00002a37-0000-1000-8000-00805f9b34fb",
    "battery_level": "00002a19-0000-1000-8000-00805f9b34fb",
    "device_name": "00002a00-0000-1000-8000-00805f9b34fb",
    "manufacturer_name": "00002a29-0000-1000-8000-00805f9b34fb",
    "model_number": "00002a24-0000-1000-8000-00805f9b34fb",
    "serial_number": "00002a25-0000-1000-8000-00805f9b34fb",
    "firmware_revision": "00002a26-0000-1000-8000-00805f9b34fb",
}

async def read_device_info(client):
    """Read common device information"""
    info = {}
    
    for name, uuid in STANDARD_CHARACTERISTICS.items():
        if name in ["device_name", "manufacturer_name", "model_number", "serial_number", "firmware_revision"]:
            try:
                data = await client.read_characteristic(uuid)
                if data:
                    info[name] = data.decode('utf-8')
            except:
                info[name] = "Not available"
    
    return info
```

### BLE Data Parsing Utilities
```python
def parse_heart_rate(data):
    """Parse heart rate measurement data"""
    if len(data) < 2:
        return None
    
    flags = data[0]
    hr_format = flags & 0x01
    
    if hr_format == 0:  # 8-bit heart rate
        return data[1]
    else:  # 16-bit heart rate
        return int.from_bytes(data[1:3], 'little')

def parse_battery_level(data):
    """Parse battery level (0-100%)"""
    if len(data) >= 1:
        return data[0]
    return None

def parse_temperature(data, scale='celsius'):
    """Parse temperature data (IEEE 11073-20601 format)"""
    if len(data) >= 5:
        # Temperature value (4 bytes, little endian)
        temp_raw = int.from_bytes(data[1:5], 'little', signed=True)
        
        # Exponent from flags
        exponent = data[0]
        if exponent > 127:
            exponent = exponent - 256  # Convert to signed
        
        temperature = temp_raw * (10 ** exponent)
        
        if scale == 'fahrenheit':
            temperature = (temperature * 9/5) + 32
        
        return temperature
    return None

# Usage in notification handler
def enhanced_notification_handler(characteristic, data):
    uuid = characteristic.uuid
    
    if uuid == STANDARD_CHARACTERISTICS["heart_rate_measurement"]:
        hr = parse_heart_rate(data)
        if hr:
            print(f"Heart Rate: {hr} BPM")
    
    elif uuid == STANDARD_CHARACTERISTICS["battery_level"]:
        battery = parse_battery_level(data)
        if battery is not None:
            print(f"Battery: {battery}%")
    
    else:
        print(f"Data from {uuid}: {data.hex()}")
```

### Advanced BLE Operations
```python
# Write with response vs without response
async def write_operations_example(client):
    char_uuid = "12345678-1234-1234-1234-123456789abc"
    data = b"Hello BLE"
    
    # Write with response (confirmed)
    await client.write_characteristic(char_uuid, data, response=True)
    
    # Write without response (faster, no confirmation)
    await client.write_characteristic(char_uuid, data, response=False)

# Read/Write descriptors
async def descriptor_operations(client):
    char_uuid = "12345678-1234-1234-1234-123456789abc"
    
    # Get characteristic object
    services = client.services
    for service in services:
        for char in service.characteristics:
            if char.uuid == char_uuid:
                # Read descriptors
                for desc in char.descriptors:
                    try:
                        value = await client.read_gatt_descriptor(desc.handle)
                        print(f"Descriptor {desc.uuid}: {value}")
                    except Exception as e:
                        print(f"Failed to read descriptor: {e}")

# Connection parameters
async def connection_management():
    client = BleakClient("00:11:22:33:44:55")
    
    # Connect with custom timeout
    await client.connect(timeout=20.0)
    
    # Check connection status
    print(f"Connected: {client.is_connected}")
    
    # Get connection info (platform dependent)
    try:
        mtu = await client.get_mtu()
        print(f"MTU: {mtu}")
    except:
        print("MTU not available on this platform")
```

---

## Integration Examples

### Bluetooth Device Manager
```python
class BluetoothDeviceManager:
    def __init__(self):
        self.classic_devices = []
        self.ble_devices = []
    
    async def full_discovery(self):
        """Discover both Classic and BLE devices"""
        
        # Classic Bluetooth discovery
        print("Discovering Classic Bluetooth devices...")
        try:
            classic = bluetooth.discover_devices(duration=8, lookup_names=True)
            self.classic_devices = classic
            print(f"Found {len(classic)} Classic Bluetooth devices")
        except Exception as e:
            print(f"Classic discovery failed: {e}")
        
        # BLE discovery
        print("Discovering BLE devices...")
        try:
            ble = await BleakScanner.discover(timeout=10.0)
            self.ble_devices = ble
            print(f"Found {len(ble)} BLE devices")
        except Exception as e:
            print(f"BLE discovery failed: {e}")
    
    def print_all_devices(self):
        print("\n=== CLASSIC BLUETOOTH DEVICES ===")
        for addr, name in self.classic_devices:
            print(f"{name} ({addr})")
        
        print("\n=== BLE DEVICES ===")
        for device in self.ble_devices:
            print(f"{device.name or 'Unknown'} ({device.address}) - RSSI: {device.rssi}")

# Usage
async def main():
    manager = BluetoothDeviceManager()
    await manager.full_discovery()
    manager.print_all_devices()

# asyncio.run(main())
```

### Cross-Platform Bluetooth Chat
```python
# Simplified chat application
class BluetoothChat:
    def __init__(self, mode='client', target_addr=None):
        self.mode = mode
        self.target_addr = target_addr
        self.running = False
    
    def start_classic_server(self):
        """Classic Bluetooth chat server"""
        server = BluetoothServer(port=22)  # Custom port
        server.handle_client = self.chat_handler
        server.start_server()
    
    def chat_handler(self, client_sock, client_info):
        """Handle chat messages"""
        print(f"Chat started with {client_info}")
        
        try:
            while self.running:
                data = client_sock.recv(1024)
                if not data:
                    break
                
                message = data.decode('utf-8')
                print(f"Them: {message}")
                
                # Echo or implement actual chat logic
                response = input("You: ")
                client_sock.send(response.encode('utf-8'))
                
        except Exception as e:
            print(f"Chat error: {e}")
    
    async def start_ble_notifications(self):
        """BLE notification-based chat"""
        if not self.target_addr:
            print("No target address specified")
            return
        
        client = BLEClient(self.target_addr)
        
        if await client.connect():
            # Implement BLE-based messaging
            # This would require a custom BLE service
            pass

# Usage would depend on specific requirements
```

## Quick Reference

### PyBluez vs Bleak Comparison

| Feature | PyBluez | Bleak |
|---------|---------|--------|
| **Protocol** | Classic Bluetooth | Bluetooth Low Energy |
| **Programming Model** | Synchronous | Asynchronous |
| **Socket Types** | RFCOMM, L2CAP | GATT |
| **Range** | ~10m (Class 2) | ~50m |
| **Power Usage** | Higher | Very Low |
| **Data Rate** | Up to 2.1 Mbps | ~1 Mbps |
| **Best For** | File transfer, Audio, Serial | Sensors, IoT, Fitness |

### Common Use Cases

| Use Case | Recommended Module | Key Functions |
|----------|-------------------|---------------|
| File Transfer | PyBluez | `RFCOMM`, `send()`, `recv()` |
| Serial Communication | PyBluez | `RFCOMM`, `BluetoothSocket` |
| IoT Sensors | Bleak | `BleakClient`, `start_notify()` |
| Fitness Devices | Bleak | Heart rate, step counter services |
| Device Configuration | PyBluez/Bleak | Depends on device type |
| Real-time Data | Bleak | BLE notifications |