# Socket Programming In-Depth Cheatsheet

## Basic Socket Concepts

### Socket Types and Protocols
```python
import socket

# Socket types
SOCK_STREAM    # TCP (reliable, connection-oriented)
SOCK_DGRAM     # UDP (unreliable, connectionless)
SOCK_RAW       # Raw sockets (requires root privileges)

# Address families
AF_INET        # IPv4
AF_INET6       # IPv6
AF_UNIX        # Unix domain sockets (local communication)

# Protocol families
IPPROTO_TCP    # TCP protocol
IPPROTO_UDP    # UDP protocol
IPPROTO_ICMP   # ICMP protocol
```

## TCP Socket Programming

### Basic TCP Server
```python
import socket
import threading
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class TCPServer:
    def __init__(self, host='localhost', port=8888):
        self.host = host
        self.port = port
        self.socket = None
        self.clients = []
        self.running = False
    
    def start(self):
        """Start the TCP server"""
        try:
            # Create socket
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            
            # Set socket options
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
            
            # Bind to address
            self.socket.bind((self.host, self.port))
            
            # Listen for connections
            self.socket.listen(5)  # Max 5 pending connections
            self.running = True
            
            logger.info(f"Server listening on {self.host}:{self.port}")
            
            while self.running:
                try:
                    # Accept connection
                    client_socket, client_address = self.socket.accept()
                    logger.info(f"Connection from {client_address}")
                    
                    # Handle client in separate thread
                    client_thread = threading.Thread(
                        target=self.handle_client,
                        args=(client_socket, client_address)
                    )
                    client_thread.daemon = True
                    client_thread.start()
                    
                except socket.error as e:
                    if self.running:
                        logger.error(f"Socket error: {e}")
                    break
                    
        except Exception as e:
            logger.error(f"Server error: {e}")
        finally:
            self.stop()
    
    def handle_client(self, client_socket, client_address):
        """Handle individual client connection"""
        self.clients.append(client_socket)
        
        try:
            while self.running:
                # Receive data
                data = client_socket.recv(1024)
                if not data:
                    break
                
                message = data.decode('utf-8')
                logger.info(f"Received from {client_address}: {message}")
                
                # Echo back to client
                response = f"Echo: {message}"
                client_socket.send(response.encode('utf-8'))
                
                # Broadcast to all clients
                self.broadcast(f"Broadcast from {client_address}: {message}", client_socket)
                
        except socket.error as e:
            logger.error(f"Client error: {e}")
        finally:
            # Cleanup
            if client_socket in self.clients:
                self.clients.remove(client_socket)
            client_socket.close()
            logger.info(f"Client {client_address} disconnected")
    
    def broadcast(self, message, sender_socket):
        """Broadcast message to all connected clients except sender"""
        for client in self.clients[:]:  # Copy list to avoid modification during iteration
            if client != sender_socket:
                try:
                    client.send(message.encode('utf-8'))
                except socket.error:
                    # Remove disconnected clients
                    self.clients.remove(client)
                    client.close()
    
    def stop(self):
        """Stop the server"""
        self.running = False
        
        # Close all client connections
        for client in self.clients:
            client.close()
        self.clients.clear()
        
        # Close server socket
        if self.socket:
            self.socket.close()
        
        logger.info("Server stopped")

# Run server
if __name__ == "__main__":
    server = TCPServer('localhost', 8888)
    try:
        server.start()
    except KeyboardInterrupt:
        logger.info("Server interrupted by user")
        server.stop()
```

### Basic TCP Client
```python
import socket
import threading
import time

class TCPClient:
    def __init__(self, host='localhost', port=8888):
        self.host = host
        self.port = port
        self.socket = None
        self.connected = False
    
    def connect(self):
        """Connect to server"""
        try:
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.socket.connect((self.host, self.port))
            self.connected = True
            print(f"Connected to {self.host}:{self.port}")
            
            # Start receiving thread
            receive_thread = threading.Thread(target=self.receive_messages)
            receive_thread.daemon = True
            receive_thread.start()
            
            return True
        except socket.error as e:
            print(f"Connection failed: {e}")
            return False
    
    def receive_messages(self):
        """Receive messages from server"""
        while self.connected:
            try:
                message = self.socket.recv(1024).decode('utf-8')
                if message:
                    print(f"Received: {message}")
                else:
                    break
            except socket.error:
                break
        
        self.connected = False
        print("Disconnected from server")
    
    def send_message(self, message):
        """Send message to server"""
        if self.connected:
            try:
                self.socket.send(message.encode('utf-8'))
                return True
            except socket.error as e:
                print(f"Send error: {e}")
                return False
        return False
    
    def disconnect(self):
        """Disconnect from server"""
        self.connected = False
        if self.socket:
            self.socket.close()
        print("Client disconnected")

# Run client
if __name__ == "__main__":
    client = TCPClient('localhost', 8888)
    
    if client.connect():
        try:
            while True:
                message = input("Enter message (or 'quit' to exit): ")
                if message.lower() == 'quit':
                    break
                client.send_message(message)
        except KeyboardInterrupt:
            pass
        finally:
            client.disconnect()
```

### Advanced TCP Server with Connection Pool
```python
import socket
import threading
import queue
import time
from concurrent.futures import ThreadPoolExecutor
import json

class AdvancedTCPServer:
    def __init__(self, host='localhost', port=8888, max_workers=10):
        self.host = host
        self.port = port
        self.max_workers = max_workers
        self.socket = None
        self.running = False
        self.clients = {}
        self.message_queue = queue.Queue()
        self.executor = ThreadPoolExecutor(max_workers=max_workers)
    
    def start(self):
        """Start the advanced TCP server"""
        try:
            # Create and configure socket
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
            self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_KEEPALIVE, 1)
            
            # Set socket timeout for accept operations
            self.socket.settimeout(1.0)
            
            self.socket.bind((self.host, self.port))
            self.socket.listen(50)  # Larger backlog
            self.running = True
            
            print(f"Advanced server listening on {self.host}:{self.port}")
            
            # Start message processor thread
            processor_thread = threading.Thread(target=self.process_messages)
            processor_thread.daemon = True
            processor_thread.start()
            
            # Accept connections
            while self.running:
                try:
                    client_socket, client_address = self.socket.accept()
                    print(f"New connection: {client_address}")
                    
                    # Submit client handling to thread pool
                    future = self.executor.submit(
                        self.handle_client, 
                        client_socket, 
                        client_address
                    )
                    
                except socket.timeout:
                    continue
                except socket.error as e:
                    if self.running:
                        print(f"Accept error: {e}")
                    break
                    
        except Exception as e:
            print(f"Server error: {e}")
        finally:
            self.stop()
    
    def handle_client(self, client_socket, client_address):
        """Handle client with improved error handling and features"""
        client_id = f"{client_address[0]}:{client_address[1]}"
        self.clients[client_id] = {
            'socket': client_socket,
            'address': client_address,
            'connected_at': time.time(),
            'last_activity': time.time()
        }
        
        try:
            # Send welcome message
            welcome_msg = {
                'type': 'welcome',
                'message': 'Connected to advanced server',
                'client_id': client_id
            }
            self.send_to_client(client_socket, welcome_msg)
            
            client_socket.settimeout(30.0)  # 30 second timeout
            
            while self.running:
                try:
                    data = client_socket.recv(4096)  # Larger buffer
                    if not data:
                        break
                    
                    # Update last activity
                    self.clients[client_id]['last_activity'] = time.time()
                    
                    # Parse message
                    try:
                        message = json.loads(data.decode('utf-8'))
                        message['client_id'] = client_id
                        message['timestamp'] = time.time()
                        
                        # Add to message queue for processing
                        self.message_queue.put(message)
                        
                    except json.JSONDecodeError:
                        # Handle plain text messages
                        text_message = {
                            'type': 'text',
                            'content': data.decode('utf-8'),
                            'client_id': client_id,
                            'timestamp': time.time()
                        }
                        self.message_queue.put(text_message)
                        
                except socket.timeout:
                    # Check if client is still alive
                    if not self.ping_client(client_socket):
                        break
                except socket.error:
                    break
                    
        except Exception as e:
            print(f"Client handling error: {e}")
        finally:
            # Cleanup
            if client_id in self.clients:
                del self.clients[client_id]
            client_socket.close()
            print(f"Client {client_address} disconnected")
    
    def process_messages(self):
        """Process messages from the queue"""
        while self.running:
            try:
                message = self.message_queue.get(timeout=1.0)
                self.handle_message(message)
                self.message_queue.task_done()
            except queue.Empty:
                continue
    
    def handle_message(self, message):
        """Handle different types of messages"""
        msg_type = message.get('type', 'text')
        client_id = message.get('client_id')
        
        if msg_type == 'text':
            # Echo and broadcast text messages
            content = message.get('content', '')
            response = {
                'type': 'echo',
                'content': f"Echo: {content}",
                'timestamp': time.time()
            }
            
            client_socket = self.clients.get(client_id, {}).get('socket')
            if client_socket:
                self.send_to_client(client_socket, response)
            
            # Broadcast to other clients
            broadcast_msg = {
                'type': 'broadcast',
                'content': content,
                'from': client_id,
                'timestamp': message.get('timestamp')
            }
            self.broadcast_message(broadcast_msg, exclude=client_id)
            
        elif msg_type == 'ping':
            # Respond to ping
            pong_msg = {
                'type': 'pong',
                'timestamp': time.time()
            }
            client_socket = self.clients.get(client_id, {}).get('socket')
            if client_socket:
                self.send_to_client(client_socket, pong_msg)
                
        elif msg_type == 'list_clients':
            # Send list of connected clients
            client_list = [
                {
                    'client_id': cid,
                    'connected_at': info['connected_at'],
                    'last_activity': info['last_activity']
                }
                for cid, info in self.clients.items()
            ]
            
            response = {
                'type': 'client_list',
                'clients': client_list,
                'total': len(client_list)
            }
            
            client_socket = self.clients.get(client_id, {}).get('socket')
            if client_socket:
                self.send_to_client(client_socket, response)
    
    def send_to_client(self, client_socket, message):
        """Send JSON message to specific client"""
        try:
            json_msg = json.dumps(message)
            client_socket.send(json_msg.encode('utf-8'))
        except (socket.error, json.JSONEncodeError) as e:
            print(f"Send error: {e}")
    
    def broadcast_message(self, message, exclude=None):
        """Broadcast message to all clients except excluded one"""
        disconnected_clients = []
        
        for client_id, client_info in self.clients.items():
            if client_id == exclude:
                continue
                
            try:
                self.send_to_client(client_info['socket'], message)
            except Exception:
                disconnected_clients.append(client_id)
        
        # Remove disconnected clients
        for client_id in disconnected_clients:
            if client_id in self.clients:
                self.clients[client_id]['socket'].close()
                del self.clients[client_id]
    
    def ping_client(self, client_socket):
        """Ping client to check if still alive"""
        try:
            ping_msg = {'type': 'ping', 'timestamp': time.time()}
            self.send_to_client(client_socket, ping_msg)
            return True
        except Exception:
            return False
    
    def get_server_stats(self):
        """Get server statistics"""
        current_time = time.time()
        return {
            'total_clients': len(self.clients),
            'clients': [
                {
                    'client_id': cid,
                    'connected_duration': current_time - info['connected_at'],
                    'idle_time': current_time - info['last_activity']
                }
                for cid, info in self.clients.items()
            ],
            'message_queue_size': self.message_queue.qsize(),
            'running': self.running
        }
    
    def stop(self):
        """Stop the server gracefully"""
        self.running = False
        
        # Close all client connections
        for client_info in self.clients.values():
            client_info['socket'].close()
        self.clients.clear()
        
        # Shutdown thread pool
        self.executor.shutdown(wait=True)
        
        # Close server socket
        if self.socket:
            self.socket.close()
        
        print("Advanced server stopped")

# Run advanced server
if __name__ == "__main__":
    server = AdvancedTCPServer('localhost', 8888, max_workers=20)
    try:
        server.start()
    except KeyboardInterrupt:
        print("\nServer interrupted by user")
        server.stop()
```

## UDP Socket Programming

### Basic UDP Server
```python
import socket
import threading
import time

class UDPServer:
    def __init__(self, host='localhost', port=8889):
        self.host = host
        self.port = port
        self.socket = None
        self.running = False
        self.clients = {}  # Track clients by address
    
    def start(self):
        """Start UDP server"""
        try:
            # Create UDP socket
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            self.socket.bind((self.host, self.port))
            self.running = True
            
            print(f"UDP Server listening on {self.host}:{self.port}")
            
            while self.running:
                try:
                    # Receive data
                    data, client_address = self.socket.recvfrom(1024)
                    
                    # Update client info
                    self.clients[client_address] = {
                        'last_seen': time.time(),
                        'message_count': self.clients.get(client_address, {}).get('message_count', 0) + 1
                    }
                    
                    message = data.decode('utf-8')
                    print(f"Received from {client_address}: {message}")
                    
                    # Handle message in separate thread
                    handler_thread = threading.Thread(
                        target=self.handle_message,
                        args=(message, client_address)
                    )
                    handler_thread.daemon = True
                    handler_thread.start()
                    
                except socket.error as e:
                    if self.running:
                        print(f"UDP error: {e}")
                    break
                    
        except Exception as e:
            print(f"Server error: {e}")
        finally:
            self.stop()
    
    def handle_message(self, message, client_address):
        """Handle received message"""
        # Echo back to sender
        response = f"Echo: {message}"
        self.send_to_client(response, client_address)
        
        # Broadcast to other clients
        broadcast_msg = f"Broadcast from {client_address}: {message}"
        self.broadcast(broadcast_msg, exclude=client_address)
    
    def send_to_client(self, message, client_address):
        """Send message to specific client"""
        try:
            self.socket.sendto(message.encode('utf-8'), client_address)
        except socket.error as e:
            print(f"Send error to {client_address}: {e}")
    
    def broadcast(self, message, exclude=None):
        """Broadcast message to all known clients"""
        current_time = time.time()
        inactive_clients = []
        
        for client_addr, client_info in self.clients.items():
            if client_addr == exclude:
                continue
            
            # Remove clients that haven't been seen for 60 seconds
            if current_time - client_info['last_seen'] > 60:
                inactive_clients.append(client_addr)
                continue
            
            self.send_to_client(message, client_addr)
        
        # Clean up inactive clients
        for addr in inactive_clients:
            del self.clients[addr]
    
    def stop(self):
        """Stop the server"""
        self.running = False
        if self.socket:
            self.socket.close()
        print("UDP Server stopped")

class UDPClient:
    def __init__(self, host='localhost', port=8889):
        self.host = host
        self.port = port
        self.socket = None
        self.listening = False
    
    def start(self):
        """Start UDP client"""
        try:
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            print(f"UDP Client connected to {self.host}:{self.port}")
            
            # Start listening thread
            self.listening = True
            listen_thread = threading.Thread(target=self.listen_for_messages)
            listen_thread.daemon = True
            listen_thread.start()
            
            return True
        except socket.error as e:
            print(f"Client start error: {e}")
            return False
    
    def listen_for_messages(self):
        """Listen for incoming messages"""
        while self.listening:
            try:
                data, server_address = self.socket.recvfrom(1024)
                message = data.decode('utf-8')
                print(f"Received: {message}")
            except socket.error:
                if self.listening:
                    print("Connection lost")
                break
    
    def send_message(self, message):
        """Send message to server"""
        try:
            self.socket.sendto(message.encode('utf-8'), (self.host, self.port))
            return True
        except socket.error as e:
            print(f"Send error: {e}")
            return False
    
    def stop(self):
        """Stop the client"""
        self.listening = False
        if self.socket:
            self.socket.close()
        print("UDP Client stopped")

# Run UDP server
if __name__ == "__main__":
    import sys
    
    if len(sys.argv) > 1 and sys.argv[1] == 'client':
        # Run as client
        client = UDPClient('localhost', 8889)
        if client.start():
            try:
                while True:
                    message = input("Enter message (or 'quit' to exit): ")
                    if message.lower() == 'quit':
                        break
                    client.send_message(message)
            except KeyboardInterrupt:
                pass
            finally:
                client.stop()
    else:
        # Run as server
        server = UDPServer('localhost', 8889)
        try:
            server.start()
        except KeyboardInterrupt:
            print("\nServer interrupted by user")
            server.stop()
```

## Socket.IO Integration

### Socket.IO Server (using python-socketio)
```python
import socketio
import eventlet
import threading
import time
import json
from datetime import datetime

# Create Socket.IO server
sio = socketio.Server(
    cors_allowed_origins="*",  # Allow all origins for development
    logger=True,
    engineio_logger=True
)

# Wrap with WSGI application
app = socketio.WSGIApp(sio)

# Store connected clients and rooms
clients = {}
chat_rooms = {}
game_sessions = {}

@sio.event
def connect(sid, environ):
    """Handle client connection"""
    print(f'Client {sid} connected')
    
    # Store client info
    clients[sid] = {
        'connected_at': datetime.now(),
        'last_activity': datetime.now(),
        'rooms': set(),
        'user_info': {}
    }
    
    # Send welcome message
    sio.emit('welcome', {
        'message': 'Connected to Socket.IO server',
        'sid': sid,
        'server_time': datetime.now().isoformat()
    }, room=sid)

@sio.event
def disconnect(sid):
    """Handle client disconnection"""
    print(f'Client {sid} disconnected')
    
    if sid in clients:
        # Leave all rooms
        for room in list(clients[sid]['rooms']):
            leave_room(sid, room)
        
        # Remove from clients
        del clients[sid]

@sio.event
def join_room(sid, data):
    """Handle room join requests"""
    room_name = data.get('room')
    user_name = data.get('username', f'User_{sid[:8]}')
    
    if not room_name:
        sio.emit('error', {'message': 'Room name required'}, room=sid)
        return
    
    # Join the room
    sio.enter_room(sid, room_name)
    
    # Update client info
    clients[sid]['rooms'].add(room_name)
    clients[sid]['user_info']['username'] = user_name
    
    # Initialize room if it doesn't exist
    if room_name not in chat_rooms:
        chat_rooms[room_name] = {
            'created_at': datetime.now(),
            'members': set(),
            'message_history': []
        }
    
    # Add to room members
    chat_rooms[room_name]['members'].add(sid)
    
    # Notify room members
    sio.emit('user_joined', {
        'username': user_name,
        'sid': sid,
        'room': room_name,
        'timestamp': datetime.now().isoformat()
    }, room=room_name)
    
    # Send room info to the user
    sio.emit('room_joined', {
        'room': room_name,
        'members_count': len(chat_rooms[room_name]['members']),
        'message_history': chat_rooms[room_name]['message_history'][-50:]  # Last 50 messages
    }, room=sid)

@sio.event
def leave_room(sid, data):
    """Handle room leave requests"""
    room_name = data.get('room') if isinstance(data, dict) else data
    
    if not room_name or room_name not in chat_rooms:
        return
    
    # Leave the room
    sio.leave_room(sid, room_name)
    
    # Update client info
    if sid in clients:
        clients[sid]['rooms'].discard(room_name)
    
    # Remove from room members
    if sid in chat_rooms[room_name]['members']:
        chat_rooms[room_name]['members'].remove(sid)
        
        # Notify remaining members
        user_name = clients.get(sid, {}).get('user_info', {}).get('username', f'User_{sid[:8]}')
        sio.emit('user_left', {
            'username': user_name,
            'sid': sid,
            'room': room_name,
            'timestamp': datetime.now().isoformat()
        }, room=room_name)
        
        # Remove room if empty
        if not chat_rooms[room_name]['members']:
            del chat_rooms[room_name]

@sio.event
def send_message(sid, data):
    """Handle chat messages"""
    room_name = data.get('room')
    message = data.get('message')
    
    if not room_name or not message:
        sio.emit('error', {'message': 'Room and message required'}, room=sid)
        return
    
    if room_name not in chat_rooms or sid not in chat_rooms[room_name]['members']:
        sio.emit('error', {'message': 'Not a member of this room'}, room=sid)
        return
    
    # Update last activity
    if sid in clients:
        clients[sid]['last_activity'] = datetime.now()
    
    # Create message object
    message_obj = {
        'id': f"{int(time.time() * 1000)}_{sid}",
        'username': clients.get(sid, {}).get('user_info', {}).get('username', f'User_{sid[:8]}'),
        'message': message,
        'room': room_name,
        'timestamp': datetime.now().isoformat(),
        'sid': sid
    }
    
    # Store in room history
    chat_rooms[room_name]['message_history'].append(message_obj)
    
    # Keep only last 100 messages
    if len(chat_rooms[room_name]['message_history']) > 100:
        chat_rooms[room_name]['message_history'] = chat_rooms[room_name]['message_history'][-100:]
    
    # Broadcast to room
    sio.emit('new_message', message_obj, room=room_name)

@sio.event
def private_message(sid, data):
    """Handle private messages between users"""
    target_sid = data.get('target_sid')
    message = data.get('message')
    
    if not target_sid or not message:
        sio.emit('error', {'message': 'Target SID and message required'}, room=sid)
        return
    
    if target_sid not in clients:
        sio.emit('error', {'message': 'Target user not found'}, room=sid)
        return
    
    # Create private message
    sender_name = clients.get(sid, {}).get('user_info', {}).get('username', f'User_{sid[:8]}')
    message_obj = {
        'from': sender_name,
        'from_sid': sid,
        'message': message,
        'timestamp': datetime.now().isoformat(),
        'type': 'private'
    }
    
    # Send to both sender and receiver
    sio.emit('private_message', message_obj, room=target_sid)
    sio.emit('private_message_sent', message_obj, room=sid)

@sio.event
def typing_start(sid, data):
    """Handle typing indicators"""
    room_name = data.get('room')
    if room_name and room_name in chat_rooms and sid in chat_rooms[room_name]['members']:
        user_name = clients.get(sid, {}).get('user_info', {}).get('username', f'User_{sid[:8]}')
        sio.emit('user_typing', {
            'username': user_name,
            'sid': sid,
            'room': room_name
        }, room=room_name, skip_sid=sid)

@sio.event
def typing_stop(sid, data):
    """Handle typing stop indicators"""
    room_name = data.get('room')
    if room_name and room_name in chat_rooms and sid in chat_rooms[room_name]['members']:
        user_name = clients.get(sid, {}).get('user_info', {}).get('username', f'User_{sid[:8]}')
        sio.emit('user_stop_typing', {
            'username': user_name,
            'sid': sid,
            'room': room_name
        }, room=room_name, skip_sid=sid)

@sio.event
def get_room_list(sid, data):
    """Get list of available rooms"""
    room_list = [
        {
            'name': room_name,
            'members_count': len(room_info['members']),
            'created_at': room_info['created_at'].isoformat(),
            'last_activity': max(
                [clients.get(member, {}).get('last_activity', room_info['created_at']) 
                 for member in room_info['members']] + [room_info['created_at']]
            ).isoformat()
        }
        for room_name, room_info in chat_rooms.items()
    ]
    
    sio.emit('room_list', {'rooms': room_list}, room=sid)

@sio.event
def get_online_users(sid, data):
    """Get list of online users"""
    user_list = [
        {
            'sid': user_sid,
            'username': user_info.get('user_info', {}).get('username', f'User_{user_sid[:8]}'),
            'connected_at': user_info['connected_at'].isoformat(),
            'last_activity': user_info['last_activity'].isoformat(),
            'rooms': list(user_info['rooms'])
        }
        for user_sid, user_info in clients.items()
    ]
    
    sio.emit('online_users', {'users': user_list}, room=sid)

# Game session handlers
@sio.event
def create_game_session(sid, data):
    """Create a new game session"""
    game_type = data.get('game_type', 'generic')
    session_name = data.get('session_name', f'Game_{int(time.time())}')
    max_players = data.get('max_players', 4)
    
    session_id = f"{game_type}_{int(time.time())}_{sid[:8]}"
    
    game_sessions[session_id] = {
        'id': session_id,
        'name': session_name,
        'type': game_type,
        'host': sid,
        'players': {sid: clients.get(sid, {}).get('user_info', {}).get('username', f'User_{sid[:8]}')},
        'max_players': max_players,
        'created_at': datetime.now(),
        'status': 'waiting',  # waiting, playing, finished
        'game_state': {}
    }
    
    # Join the game room
    sio.enter_room(sid, session_id)
    
    sio.emit('game_session_created', {
        'session_id': session_id,
        'session': game_sessions[session_id]
    }, room=sid)

@sio.event
def join_game_session(sid, data):
    """Join an existing game session"""
    session_id = data.get('session_id')
    
    if session_id not in game_sessions:
        sio.emit('error', {'message': 'Game session not found'}, room=sid)
        return
    
    session = game_sessions[session_id]
    
    if len(session['players']) >= session['max_players']:
        sio.emit('error', {'message': 'Game session is full'}, room=sid)
        return
    
    if session['status'] != 'waiting':
        sio.emit('error', {'message': 'Game session is not accepting new players'}, room=sid)
        return
    
    # Join the session
    username = clients.get(sid, {}).get('user_info', {}).get('username', f'User_{sid[:8]}')
    session['players'][sid] = username
    
    sio.enter_room(sid, session_id)
    
    # Notify all players
    sio.emit('player_joined_game', {
        'session_id': session_id,
        'player': username,
        'players': session['players']
    }, room=session_id)

@sio.event
def game_action(sid, data):
    """Handle game actions"""
    session_id = data.get('session_id')
    action = data.get('action')
    
    if session_id not in game_sessions:
        sio.emit('error', {'message': 'Game session not found'}, room=sid)
        return
    
    if sid not in game_sessions[session_id]['players']:
        sio.emit('error', {'message': 'Not a player in this game'}, room=sid)
        return
    
    # Broadcast action to all players
    sio.emit('game_action_broadcast', {
        'session_id': session_id,
        'player': game_sessions[session_id]['players'][sid],
        'action': action,
        'timestamp': datetime.now().isoformat()
    }, room=session_id)

# Background task to clean up inactive sessions
def cleanup_inactive_sessions():
    """Clean up inactive game sessions and rooms"""
    while True:
        current_time = datetime.now()
        
        # Clean up game sessions
        inactive_sessions = []
        for session_id, session in game_sessions.items():
            # Remove sessions inactive for more than 1 hour
            if (current_time - session['created_at']).seconds > 3600:
                inactive_sessions.append(session_id)
        
        for session_id in inactive_sessions:
            # Notify players
            sio.emit('game_session_closed', {
                'session_id': session_id,
                'reason': 'Inactive for too long'
            }, room=session_id)
            
            del game_sessions[session_id]
        
        # Clean up empty rooms
        empty_rooms = []
        for room_name, room_info in chat_rooms.items():
            if not room_info['members']:
                empty_rooms.append(room_name)
        
        for room_name in empty_rooms:
            del chat_rooms[room_name]
        
        time.sleep(300)  # Check every 5 minutes

# Start cleanup thread
cleanup_thread = threading.Thread(target=cleanup_inactive_sessions)
cleanup_thread.daemon = True
cleanup_thread.start()

if __name__ == '__main__':
    # Run the Socket.IO server
    print("Starting Socket.IO server on localhost:5000")
    eventlet.wsgi.server(eventlet.listen(('localhost', 5000)), app)
```

### Socket.IO Client
```python
import socketio
import threading
import time
import json

class SocketIOClient:
    def __init__(self, server_url='http://localhost:5000'):
        self.server_url = server_url
        self.sio = socketio.Client()
        self.connected = False
        self.current_room = None
        self.username = None
        
        # Register event handlers
        self.setup_event_handlers()
    
    def setup_event_handlers(self):
        """Setup Socket.IO event handlers"""
        
        @self.sio.event
        def connect():
            print("Connected to server")
            self.connected = True
        
        @self.sio.event
        def disconnect():
            print("Disconnected from server")
            self.connected = False
        
        @self.sio.event
        def welcome(data):
            print(f"Welcome message: {data['message']}")
            print(f"Your SID: {data['sid']}")
        
        @self.sio.event
        def room_joined(data):
            print(f"Joined room: {data['room']}")
            print(f"Members: {data['members_count']}")
            
            # Display recent messages
            if data['message_history']:
                print("\n--- Recent Messages ---")
                for msg in data['message_history'][-10:]:  # Last 10 messages
                    print(f"[{msg['timestamp']}] {msg['username']}: {msg['message']}")
                print("--- End Messages ---\n")
        
        @self.sio.event
        def user_joined(data):
            if data['username'] != self.username:
                print(f">>> {data['username']} joined the room")
        
        @self.sio.event
        def user_left(data):
            print(f">>> {data['username']} left the room")
        
        @self.sio.event
        def new_message(data):
            if data['username'] != self.username:
                print(f"[{data['timestamp']}] {data['username']}: {data['message']}")
        
        @self.sio.event
        def private_message(data):
            print(f"\n*** Private from {data['from']}: {data['message']} ***")
        
        @self.sio.event
        def private_message_sent(data):
            print(f"*** Private to {data['from']}: {data['message']} ***")
        
        @self.sio.event
        def user_typing(data):
            print(f">>> {data['username']} is typing...")
        
        @self.sio.event
        def user_stop_typing(data):
            print(f">>> {data['username']} stopped typing")
        
        @self.sio.event
        def room_list(data):
            print("\n--- Available Rooms ---")
            for room in data['rooms']:
                print(f"Room: {room['name']} ({room['members_count']} members)")
            print("--- End Rooms ---\n")
        
        @self.sio.event
        def online_users(data):
            print("\n--- Online Users ---")
            for user in data['users']:
                print(f"User: {user['username']} (SID: {user['sid'][:8]}...)")
            print("--- End Users ---\n")
        
        @self.sio.event
        def error(data):
            print(f"Error: {data['message']}")
    
    def connect_to_server(self):
        """Connect to the Socket.IO server"""
        try:
            self.sio.connect(self.server_url)
            return True
        except Exception as e:
            print(f"Connection failed: {e}")
            return False
    
    def disconnect_from_server(self):
        """Disconnect from the server"""
        self.sio.disconnect()
    
    def join_room(self, room_name, username):
        """Join a chat room"""
        self.username = username
        self.current_room = room_name
        self.sio.emit('join_room', {
            'room': room_name,
            'username': username
        })
    
    def leave_room(self, room_name):
        """Leave a chat room"""
        self.sio.emit('leave_room', {'room': room_name})
        if self.current_room == room_name:
            self.current_room = None
    
    def send_message(self, message):
        """Send a message to the current room"""
        if not self.current_room:
            print("You must join a room first!")
            return
        
        self.sio.emit('send_message', {
            'room': self.current_room,
            'message': message
        })
    
    def send_private_message(self, target_sid, message):
        """Send a private message to a specific user"""
        self.sio.emit('private_message', {
            'target_sid': target_sid,
            'message': message
        })
    
    def start_typing(self):
        """Indicate that user is typing"""
        if self.current_room:
            self.sio.emit('typing_start', {'room': self.current_room})
    
    def stop_typing(self):
        """Indicate that user stopped typing"""
        if self.current_room:
            self.sio.emit('typing_stop', {'room': self.current_room})
    
    def get_room_list(self):
        """Get list of available rooms"""
        self.sio.emit('get_room_list', {})
    
    def get_online_users(self):
        """Get list of online users"""
        self.sio.emit('get_online_users', {})
    
    def run_interactive_client(self):
        """Run interactive client interface"""
        if not self.connect_to_server():
            return
        
        print("Socket.IO Client Connected!")
        print("Commands:")
        print("  /join <room> <username> - Join a room")
        print("  /leave <room> - Leave a room")
        print("  /rooms - List available rooms")
        print("  /users - List online users")
        print("  /pm <target_sid> <message> - Send private message")
        print("  /quit - Exit client")
        print("  Any other text will be sent as a message to the current room")
        
        try:
            while self.connected:
                user_input = input(f"[{self.current_room or 'No Room'}] > ").strip()
                
                if not user_input:
                    continue
                
                if user_input.startswith('/'):
                    # Handle commands
                    parts = user_input[1:].split(' ', 2)
                    command = parts[0].lower()
                    
                    if command == 'quit':
                        break
                    elif command == 'join' and len(parts) >= 3:
                        room_name = parts[1]
                        username = parts[2]
                        self.join_room(room_name, username)
                    elif command == 'leave' and len(parts) >= 2:
                        room_name = parts[1]
                        self.leave_room(room_name)
                    elif command == 'rooms':
                        self.get_room_list()
                    elif command == 'users':
                        self.get_online_users()
                    elif command == 'pm' and len(parts) >= 3:
                        target_sid = parts[1]
                        message = parts[2]
                        self.send_private_message(target_sid, message)
                    else:
                        print("Invalid command or missing parameters")
                else:
                    # Send as regular message
                    self.send_message(user_input)
                    
        except KeyboardInterrupt:
            pass
        finally:
            self.disconnect_from_server()
            print("Client disconnected")

# Run the client
if __name__ == '__main__':
    client = SocketIOClient('http://localhost:5000')
    client.run_interactive_client()
```

## Asynchronous Socket Programming

### AsyncIO Socket Server
```python
import asyncio
import json
import logging
import weakref
from datetime import datetime
from typing import Dict, Set

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class AsyncTCPServer:
    def __init__(self, host='localhost', port=8890):
        self.host = host
        self.port = port
        self.clients: Dict[str, 'ClientConnection'] = {}
        self.rooms: Dict[str, Set[str]] = {}
        self.server = None
        self.running = False
    
    async def start(self):
        """Start the async TCP server"""
        try:
            self.server = await asyncio.start_server(
                self.handle_client,
                self.host,
                self.port
            )
            self.running = True
            
            logger.info(f"Async server listening on {self.host}:{self.port}")
            
            # Start background tasks
            asyncio.create_task(self.cleanup_task())
            asyncio.create_task(self.heartbeat_task())
            
            async with self.server:
                await self.server.serve_forever()
                
        except Exception as e:
            logger.error(f"Server error: {e}")
        finally:
            await self.stop()
    
    async def handle_client(self, reader, writer):
        """Handle individual client connection"""
        client_address = writer.get_extra_info('peername')
        client_id = f"{client_address[0]}:{client_address[1]}:{id(writer)}"
        
        logger.info(f"New connection: {client_address}")
        
        # Create client connection object
        client = ClientConnection(client_id, reader, writer, client_address)
        self.clients[client_id] = client
        
        try:
            # Send welcome message
            await client.send_message({
                'type': 'welcome',
                'client_id': client_id,
                'server_time': datetime.now().isoformat()
            })
            
            # Handle client messages
            async for message in client.read_messages():
                if message:
                    await self.process_message(client_id, message)
                    
        except asyncio.CancelledError:
            logger.info(f"Client {client_address} connection cancelled")
        except Exception as e:
            logger.error(f"Client handling error: {e}")
        finally:
            # Cleanup
            await self.disconnect_client(client_id)
    
    async def process_message(self, client_id: str, message: dict):
        """Process message from client"""
        client = self.clients.get(client_id)
        if not client:
            return
        
        msg_type = message.get('type')
        
        if msg_type == 'ping':
            await client.send_message({
                'type': 'pong',
                'timestamp': datetime.now().isoformat()
            })
            
        elif msg_type == 'join_room':
            room_name = message.get('room')
            if room_name:
                await self.join_room(client_id, room_name)
                
        elif msg_type == 'leave_room':
            room_name = message.get('room')
            if room_name:
                await self.leave_room(client_id, room_name)
                
        elif msg_type == 'message':
            room_name = message.get('room')
            content = message.get('content')
            if room_name and content:
                await self.broadcast_to_room(room_name, {
                    'type': 'room_message',
                    'from': client_id,
                    'room': room_name,
                    'content': content,
                    'timestamp': datetime.now().isoformat()
                }, exclude=client_id)
                
        elif msg_type == 'private_message':
            target_id = message.get('target')
            content = message.get('content')
            if target_id and content:
                await self.send_private_message(client_id, target_id, content)
                
        elif msg_type == 'list_clients':
            await client.send_message({
                'type': 'client_list',
                'clients': list(self.clients.keys()),
                'total': len(self.clients)
            })
            
        elif msg_type == 'list_rooms':
            await client.send_message({
                'type': 'room_list',
                'rooms': {
                    room: len(members) 
                    for room, members in self.rooms.items()
                }
            })
    
    async def join_room(self, client_id: str, room_name: str):
        """Add client to room"""
        if room_name not in self.rooms:
            self.rooms[room_name] = set()
        
        self.rooms[room_name].add(client_id)
        
        client = self.clients.get(client_id)
        if client:
            client.rooms.add(room_name)
            
            await client.send_message({
                'type': 'room_joined',
                'room': room_name,
                'members': len(self.rooms[room_name])
            })
            
            # Notify other room members
            await self.broadcast_to_room(room_name, {
                'type': 'user_joined',
                'user': client_id,
                'room': room_name
            }, exclude=client_id)
    
    async def leave_room(self, client_id: str, room_name: str):
        """Remove client from room"""
        if room_name in self.rooms:
            self.rooms[room_name].discard(client_id)
            
            # Remove empty rooms
            if not self.rooms[room_name]:
                del self.rooms[room_name]
        
        client = self.clients.get(client_id)
        if client:
            client.rooms.discard(room_name)
            
            await client.send_message({
                'type': 'room_left',
                'room': room_name
            })
            
            # Notify remaining room members
            if room_name in self.rooms:
                await self.broadcast_to_room(room_name, {
                    'type': 'user_left',
                    'user': client_id,
                    'room': room_name
                })
    
    async def broadcast_to_room(self, room_name: str, message: dict, exclude: str = None):
        """Broadcast message to all clients in room"""
        if room_name not in self.rooms:
            return
        
        tasks = []
        for client_id in self.rooms[room_name]:
            if client_id != exclude and client_id in self.clients:
                task = asyncio.create_task(
                    self.clients[client_id].send_message(message)
                )
                tasks.append(task)
        
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)
    
    async def send_private_message(self, from_id: str, to_id: str, content: str):
        """Send private message between clients"""
        target_client = self.clients.get(to_id)
        sender_client = self.clients.get(from_id)
        
        if target_client:
            await target_client.send_message({
                'type': 'private_message',
                'from': from_id,
                'content': content,
                'timestamp': datetime.now().isoformat()
            })
        
        if sender_client:
            await sender_client.send_message({
                'type': 'private_message_sent',
                'to': to_id,
                'content': content,
                'timestamp': datetime.now().isoformat()
            })
    
    async def disconnect_client(self, client_id: str):
        """Handle client disconnection"""
        client = self.clients.get(client_id)
        if not client:
            return
        
        # Leave all rooms
        for room_name in list(client.rooms):
            await self.leave_room(client_id, room_name)
        
        # Close connection
        await client.close()
        
        # Remove from clients
        del self.clients[client_id]
        
        logger.info(f"Client {client_id} disconnected")
    
    async def cleanup_task(self):
        """Background task to clean up disconnected clients"""
        while self.running:
            try:
                disconnected_clients = []
                
                for client_id, client in self.clients.items():
                    if client.writer.is_closing():
                        disconnected_clients.append(client_id)
                
                for client_id in disconnected_clients:
                    await self.disconnect_client(client_id)
                
                await asyncio.sleep(30)  # Check every 30 seconds
                
            except asyncio.CancelledError:
                break
            except Exception as e:
                logger.error(f"Cleanup task error: {e}")
    
    async def heartbeat_task(self):
        """Send periodic heartbeat to all clients"""
        while self.running:
            try:
                heartbeat_message = {
                    'type': 'heartbeat',
                    'timestamp': datetime.now().isoformat(),
                    'clients_count': len(self.clients)
                }
                
                tasks = []
                for client in self.clients.values():
                    task = asyncio.create_task(client.send_message(heartbeat_message))
                    tasks.append(task)
                
                if tasks:
                    await asyncio.gather(*tasks, return_exceptions=True)
                
                await asyncio.sleep(60)  # Send every 60 seconds
                
            except asyncio.CancelledError:
                break
            except Exception as e:
                logger.error(f"Heartbeat task error: {e}")
    
    async def stop(self):
        """Stop the server"""
        self.running = False
        
        # Disconnect all clients
        tasks = []
        for client_id in list(self.clients.keys()):
            task = asyncio.create_task(self.disconnect_client(client_id))
            tasks.append(task)
        
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)
        
        # Close server
        if self.server:
            self.server.close()
            await self.server.wait_closed()
        
        logger.info("Async server stopped")

class ClientConnection:
    def __init__(self, client_id: str, reader: asyncio.StreamReader, 
                 writer: asyncio.StreamWriter, address):
        self.client_id = client_id
        self.reader = reader
        self.writer = writer
        self.address = address
        self.rooms: Set[str] = set()
        self.connected_at = datetime.now()
        self.last_activity = datetime.now()
    
    async def read_messages(self):
        """Async generator to read messages from client"""
        try:
            while not self.writer.is_closing():
                data = await self.reader.read(4096)
                if not data:
                    break
                
                self.last_activity = datetime.now()
                
                try:
                    # Handle multiple JSON messages in one packet
                    messages = data.decode('utf-8').strip().split('\n')
                    for msg_str in messages:
                        if msg_str:
                            message = json.loads(msg_str)
                            yield message
                except json.JSONDecodeError as e:
                    logger.error(f"JSON decode error from {self.client_id}: {e}")
                    yield None
                    
        except asyncio.CancelledError:
            logger.info(f"Read cancelled for {self.client_id}")
        except Exception as e:
            logger.error(f"Read error for {self.client_id}: {e}")
    
    async def send_message(self, message: dict):
        """Send message to client"""
        try:
            if self.writer.is_closing():
                return False
            
            json_message = json.dumps(message) + '\n'
            self.writer.write(json_message.encode('utf-8'))
            await self.writer.drain()
            return True
            
        except Exception as e:
            logger.error(f"Send error to {self.client_id}: {e}")
            return False
    
    async def close(self):
        """Close the client connection"""
        try:
            if not self.writer.is_closing():
                self.writer.close()
                await self.writer.wait_closed()
        except Exception as e:
            logger.error(f"Close error for {self.client_id}: {e}")

# Async client
class AsyncTCPClient:
    def __init__(self, host='localhost', port=8890):
        self.host = host
        self.port = port
        self.reader = None
        self.writer = None
        self.connected = False
        self.client_id = None
    
    async def connect(self):
        """Connect to the async server"""
        try:
            self.reader, self.writer = await asyncio.open_connection(
                self.host, self.port
            )
            self.connected = True
            logger.info(f"Connected to {self.host}:{self.port}")
            
            # Start message receiving task
            asyncio.create_task(self.receive_messages())
            
            return True
        except Exception as e:
            logger.error(f"Connection error: {e}")
            return False
    
    async def receive_messages(self):
        """Receive messages from server"""
        try:
            while self.connected and not self.writer.is_closing():
                data = await self.reader.read(4096)
                if not data:
                    break
                
                try:
                    messages = data.decode('utf-8').strip().split('\n')
                    for msg_str in messages:
                        if msg_str:
                            message = json.loads(msg_str)
                            await self.handle_message(message)
                except json.JSONDecodeError as e:
                    logger.error(f"JSON decode error: {e}")
                    
        except asyncio.CancelledError:
            logger.info("Receive task cancelled")
        except Exception as e:
            logger.error(f"Receive error: {e}")
        finally:
            self.connected = False
    
    async def handle_message(self, message: dict):
        """Handle received messages"""
        msg_type = message.get('type')
        
        if msg_type == 'welcome':
            self.client_id = message.get('client_id')
            print(f"Welcome! Your client ID: {self.client_id}")
            
        elif msg_type == 'pong':
            print(f"Pong received at {message.get('timestamp')}")
            
        elif msg_type == 'room_joined':
            print(f"Joined room: {message.get('room')} ({message.get('members')} members)")
            
        elif msg_type == 'room_left':
            print(f"Left room: {message.get('room')}")
            
        elif msg_type == 'user_joined':
            print(f"User {message.get('user')} joined room {message.get('room')}")
            
        elif msg_type == 'user_left':
            print(f"User {message.get('user')} left room {message.get('room')}")
            
        elif msg_type == 'room_message':
            print(f"[{message.get('room')}] {message.get('from')}: {message.get('content')}")
            
        elif msg_type == 'private_message':
            print(f"Private from {message.get('from')}: {message.get('content')}")
            
        elif msg_type == 'client_list':
            print(f"Connected clients ({message.get('total')}): {message.get('clients')}")
            
        elif msg_type == 'room_list':
            print("Available rooms:")
            for room, count in message.get('rooms', {}).items():
                print(f"  {room}: {count} members")
                
        elif msg_type == 'heartbeat':
            # Respond to heartbeat
            await self.send_message({'type': 'ping'})
    
    async def send_message(self, message: dict):
        """Send message to server"""
        try:
            if not self.connected or self.writer.is_closing():
                return False
            
            json_message = json.dumps(message) + '\n'
            self.writer.write(json_message.encode('utf-8'))
            await self.writer.drain()
            return True
            
        except Exception as e:
            logger.error(f"Send error: {e}")
            return False
    
    async def disconnect(self):
        """Disconnect from server"""
        self.connected = False
        if self.writer and not self.writer.is_closing():
            self.writer.close()
            await self.writer.wait_closed()
        logger.info("Disconnected from server")

# Run async server
async def run_server():
    server = AsyncTCPServer('localhost', 8890)
    await server.start()

# Run async client
async def run_client():
    client = AsyncTCPClient('localhost', 8890)
    
    if await client.connect():
        try:
            print("Async client connected!")
            print("Commands: /join <room>, /leave <room>, /pm <target> <message>, /ping, /clients, /rooms, /quit")
            
            while client.connected:
                user_input = await asyncio.get_event_loop().run_in_executor(
                    None, input, "> "
                )
                
                if user_input.strip() == '/quit':
                    break
                elif user_input.startswith('/join '):
                    room = user_input[6:].strip()
                    await client.send_message({'type': 'join_room', 'room': room})
                elif user_input.startswith('/leave '):
                    room = user_input[7:].strip()
                    await client.send_message({'type': 'leave_room', 'room': room})
                elif user_input.startswith('/pm '):
                    parts = user_input[4:].split(' ', 1)
                    if len(parts) == 2:
                        target, content = parts
                        await client.send_message({
                            'type': 'private_message',
                            'target': target,
                            'content': content
                        })
                elif user_input.strip() == '/ping':
                    await client.send_message({'type': 'ping'})
                elif user_input.strip() == '/clients':
                    await client.send_message({'type': 'list_clients'})
                elif user_input.strip() == '/rooms':
                    await client.send_message({'type': 'list_rooms'})
                else:
                    # Regular message (needs room context)
                    print("Use /join <room> first, then send messages")
                    
        except KeyboardInterrupt:
            pass
        finally:
            await client.disconnect()

if __name__ == '__main__':
    import sys
    
    if len(sys.argv) > 1 and sys.argv[1] == 'client':
        asyncio.run(run_client())
    else:
        asyncio.run(run_server())
```

## WebSocket Implementation

### WebSocket Server with websockets library
```python
import asyncio
import websockets
import json
import logging
from datetime import datetime
from typing import Dict, Set
import uuid

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class WebSocketServer:
    def __init__(self, host='localhost', port=8765):
        self.host = host
        self.port = port
        self.clients: Dict[str, websockets.WebSocketServerProtocol] = {}
        self.rooms: Dict[str, Set[str]] = {}
        self.client_info: Dict[str, dict] = {}
    
    async def register_client(self, websocket, path):
        """Register a new WebSocket client"""
        client_id = str(uuid.uuid4())
        self.clients[client_id] = websocket
        self.client_info[client_id] = {
            'connected_at': datetime.now(),
            'last_activity': datetime.now(),
            'rooms': set(),
            'username': f'User_{client_id[:8]}'
        }
        
        logger.info(f"Client {client_id} connected from {websocket.remote_address}")
        
        # Send welcome message
        await self.send_to_client(client_id, {
            'type': 'welcome',
            'client_id': client_id,
            'message': 'Connected to WebSocket server'
        })
        
        try:
            # Handle messages from this client
            async for message in websocket:
                try:
                    data = json.loads(message)
                    await self.handle_message(client_id, data)
                except json.JSONDecodeError:
                    await self.send_to_client(client_id, {
                        'type': 'error',
                        'message': 'Invalid JSON format'
                    })
        except websockets.exceptions.ConnectionClosed:
            logger.info(f"Client {client_id} disconnected")
        except Exception as e:
            logger.error(f"Error handling client {client_id}: {e}")
        finally:
            await self.unregister_client(client_id)
    
    async def unregister_client(self, client_id: str):
        """Unregister a WebSocket client"""
        if client_id not in self.clients:
            return
        
        # Leave all rooms
        client_rooms = self.client_info.get(client_id, {}).get('rooms', set())
        for room_name in list(client_rooms):
            await self.leave_room(client_id, room_name)
        
        # Remove client
        del self.clients[client_id]
        if client_id in self.client_info:
            del self.client_info[client_id]
        
        logger.info(f"Client {client_id} unregistered")
    
    async def handle_message(self, client_id: str, data: dict):
        """Handle message from client"""
        if client_id not in self.clients:
            return
        
        # Update last activity
        if client_id in self.client_info:
            self.client_info[client_id]['last_activity'] = datetime.now()
        
        message_type = data.get('type')
        
        if message_type == 'set_username':
            username = data.get('username', '').strip()
            if username:
                self.client_info[client_id]['username'] = username
                await self.send_to_client(client_id, {
                    'type': 'username_set',
                    'username': username
                })
        
        elif message_type == 'join_room':
            room_name = data.get('room', '').strip()
            if room_name:
                await self.join_room(client_id, room_name)
        
        elif message_type == 'leave_room':
            room_name = data.get('room', '').strip()
            if room_name:
                await self.leave_room(client_id, room_name)
        
        elif message_type == 'room_message':
            room_name = data.get('room', '').strip()
            content = data.get('content', '').strip()
            if room_name and content:
                await self.broadcast_to_room(room_name, {
                    'type': 'room_message',
                    'room': room_name,
                    'from': self.client_info[client_id]['username'],
                    'from_id': client_id,
                    'content': content,
                    'timestamp': datetime.now().isoformat()
                }, exclude=client_id)
        
        elif message_type == 'private_message':
            target_id = data.get('target_id', '').strip()
            content = data.get('content', '').strip()
            if target_id and content and target_id in self.clients:
                await self.send_to_client(target_id, {
                    'type': 'private_message',
                    'from': self.client_info[client_id]['username'],
                    'from_id': client_id,
                    'content': content,
                    'timestamp': datetime.now().isoformat()
                })
                
                # Confirm to sender
                await self.send_to_client(client_id, {
                    'type': 'private_message_sent',
                    'to': self.client_info.get(target_id, {}).get('username', target_id),
                    'to_id': target_id,
                    'content': content
                })
        
        elif message_type == 'list_rooms':
            room_list = {
                room: len(members) for room, members in self.rooms.items()
            }
            await self.send_to_client(client_id, {
                'type': 'room_list',
                'rooms': room_list
            })
        
        elif message_type == 'list_clients':
            client_list = [
                {
                    'id': cid,
                    'username': info['username'],
                    'connected_at': info['connected_at'].isoformat(),
                    'rooms': list(info['rooms'])
                }
                for cid, info in self.client_info.items()
            ]
            await self.send_to_client(client_id, {
                'type': 'client_list',
                'clients': client_list
            })
        
        elif message_type == 'ping':
            await self.send_to_client(client_id, {
                'type': 'pong',
                'timestamp': datetime.now().isoformat()
            })
    
    async def join_room(self, client_id: str, room_name: str):
        """Add client to room"""
        if room_name not in self.rooms:
            self.rooms[room_name] = set()
        
        self.rooms[room_name].add(client_id)
        self.client_info[client_id]['rooms'].add(room_name)
        
        username = self.client_info[client_id]['username']
        
        # Notify client
        await self.send_to_client(client_id, {
            'type': 'room_joined',
            'room': room_name,
            'members_count': len(self.rooms[room_name])
        })
        
        # Notify other room members
        await self.broadcast_to_room(room_name, {
            'type': 'user_joined_room',
            'room': room_name,
            'username': username,
            'user_id': client_id
        }, exclude=client_id)
    
    async def leave_room(self, client_id: str, room_name: str):
        """Remove client from room"""
        if room_name in self.rooms:
            self.rooms[room_name].discard(client_id)
            if not self.rooms[room_name]:
                del self.rooms[room_name]
        
        if client_id in self.client_info:
            self.client_info[client_id]['rooms'].discard(room_name)
            username = self.client_info[client_id]['username']
            
            # Notify client
            await self.send_to_client(client_id, {
                'type': 'room_left',
                'room': room_name
            })
            
            # Notify remaining room members
            if room_name in self.rooms:
                await self.broadcast_to_room(room_name, {
                    'type': 'user_left_room',
                    'room': room_name,
                    'username': username,
                    'user_id': client_id
                })
    
    async def send_to_client(self, client_id: str, message: dict):
        """Send message to specific client"""
        if client_id not in self.clients:
            return
        
        try:
            websocket = self.clients[client_id]
            await websocket.send(json.dumps(message))
        except websockets.exceptions.ConnectionClosed:
            await self.unregister_client(client_id)
        except Exception as e:
            logger.error(f"Error sending to client {client_id}: {e}")
    
    async def broadcast_to_room(self, room_name: str, message: dict, exclude: str = None):
        """Broadcast message to all clients in room"""
        if room_name not in self.rooms:
            return
        
        tasks = []
        for client_id in self.rooms[room_name]:
            if client_id != exclude:
                task = asyncio.create_task(self.send_to_client(client_id, message))
                tasks.append(task)
        
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)
    
    async def broadcast_to_all(self, message: dict, exclude: str = None):
        """Broadcast message to all connected clients"""
        tasks = []
        for client_id in self.clients:
            if client_id != exclude:
                task = asyncio.create_task(self.send_to_client(client_id, message))
                tasks.append(task)
        
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)
    
    async def start_server(self):
        """Start the WebSocket server"""
        logger.info(f"Starting WebSocket server on {self.host}:{self.port}")
        
        # Start periodic cleanup task
        asyncio.create_task(self.cleanup_task())
        
        async with websockets.serve(self.register_client, self.host, self.port):
            logger.info(f"WebSocket server listening on {self.host}:{self.port}")
            await asyncio.Future()  # Run forever
    
    async def cleanup_task(self):
        """Periodic cleanup of inactive connections"""
        while True:
            try:
                current_time = datetime.now()
                inactive_clients = []
                
                for client_id, info in self.client_info.items():
                    # Remove clients inactive for more than 5 minutes
                    if (current_time - info['last_activity']).seconds > 300:
                        inactive_clients.append(client_id)
                
                for client_id in inactive_clients:
                    logger.info(f"Removing inactive client: {client_id}")
                    await self.unregister_client(client_id)
                
                await asyncio.sleep(60)  # Check every minute
                
            except Exception as e:
                logger.error(f"Cleanup task error: {e}")
                await asyncio.sleep(60)

# WebSocket client
class WebSocketClient:
    def __init__(self, uri='ws://localhost:8765'):
        self.uri = uri
        self.websocket = None
        self.connected = False
        self.client_id = None
        self.username = None
    
    async def connect(self):
        """Connect to WebSocket server"""
        try:
            self.websocket = await websockets.connect(self.uri)
            self.connected = True
            logger.info(f"Connected to {self.uri}")
            
            # Start message receiving task
            asyncio.create_task(self.receive_messages())
            
            return True
        except Exception as e:
            logger.error(f"Connection error: {e}")
            return False
    
    async def receive_messages(self):
        """Receive messages from server"""
        try:
            async for message in self.websocket:
                try:
                    data = json.loads(message)
                    await self.handle_message(data)
                except json.JSONDecodeError:
                    logger.error("Invalid JSON received")
        except websockets.exceptions.ConnectionClosed:
            logger.info("Connection closed")
            self.connected = False
        except Exception as e:
            logger.error(f"Receive error: {e}")
            self.connected = False
    
    async def handle_message(self, data: dict):
        """Handle received messages"""
        message_type = data.get('type')
        
        if message_type == 'welcome':
            self.client_id = data.get('client_id')
            print(f"Connected! Your ID: {self.client_id}")
        
        elif message_type == 'username_set':
            self.username = data.get('username')
            print(f"Username set to: {self.username}")
        
        elif message_type == 'room_joined':
            print(f"Joined room: {data.get('room')} ({data.get('members_count')} members)")
        
        elif message_type == 'room_left':
            print(f"Left room: {data.get('room')}")
        
        elif message_type == 'user_joined_room':
            print(f"{data.get('username')} joined room {data.get('room')}")
        
        elif message_type == 'user_left_room':
            print(f"{data.get('username')} left room {data.get('room')}")
        
        elif message_type == 'room_message':
            print(f"[{data.get('room')}] {data.get('from')}: {data.get('content')}")
        
        elif message_type == 'private_message':
            print(f"Private from {data.get('from')}: {data.get('content')}")
        
        elif message_type == 'private_message_sent':
            print(f"Private to {data.get('to')}: {data.get('content')}")
        
        elif message_type == 'room_list':
            print("Available rooms:")
            for room, count in data.get('rooms', {}).items():
                print(f"  {room}: {count} members")
        
        elif message_type == 'client_list':
            print("Connected clients:")
            for client in data.get('clients', []):
                print(f"  {client['username']} (ID: {client['id'][:8]}...)")
        
        elif message_type == 'pong':
            print(f"Pong: {data.get('timestamp')}")
        
        elif message_type == 'error':
            print(f"Error: {data.get('message')}")
    
    async def send_message(self, message: dict):
        """Send message to server"""
        if not self.connected or not self.websocket:
            return False
        
        try:
            await self.websocket.send(json.dumps(message))
            return True
        except Exception as e:
            logger.error(f"Send error: {e}")
            return False
    
    async def disconnect(self):
        """Disconnect from server"""
        self.connected = False
        if self.websocket:
            await self.websocket.close()
        logger.info("Disconnected from server")

# Run WebSocket server
async def run_websocket_server():
    server = WebSocketServer('localhost', 8765)
    await server.start_server()

# Run WebSocket client
async def run_websocket_client():
    client = WebSocketClient('ws://localhost:8765')
    
    if await client.connect():
        try:
            print("WebSocket client connected!")
            print("Commands:")
            print("  /username <name> - Set username")
            print("  /join <room> - Join room")
            print("  /leave <room> - Leave room")
            print("  /msg <room> <message> - Send room message")
            print("  /pm <user_id> <message> - Send private message")
            print("  /rooms - List rooms")
            print("  /clients - List clients")
            print("  /ping - Ping server")
            print("  /quit - Exit")
            
            while client.connected:
                user_input = await asyncio.get_event_loop().run_in_executor(
                    None, input, "> "
                )
                
                parts = user_input.strip().split(' ', 2)
                if not parts[0]:
                    continue
                
                command = parts[0]
                
                if command == '/quit':
                    break
                elif command == '/username' and len(parts) >= 2:
                    await client.send_message({
                        'type': 'set_username',
                        'username': parts[1]
                    })
                elif command == '/join' and len(parts) >= 2:
                    await client.send_message({
                        'type': 'join_room',
                        'room': parts[1]
                    })
                elif command == '/leave' and len(parts) >= 2:
                    await client.send_message({
                        'type': 'leave_room',
                        'room': parts[1]
                    })
                elif command == '/msg' and len(parts) >= 3:
                    await client.send_message({
                        'type': 'room_message',
                        'room': parts[1],
                        'content': parts[2]
                    })
                elif command == '/pm' and len(parts) >= 3:
                    await client.send_message({
                        'type': 'private_message',
                        'target_id': parts[1],
                        'content': parts[2]
                    })
                elif command == '/rooms':
                    await client.send_message({'type': 'list_rooms'})
                elif command == '/clients':
                    await client.send_message({'type': 'list_clients'})
                elif command == '/ping':
                    await client.send_message({'type': 'ping'})
                else:
                    print("Unknown command or missing parameters")
                    
        except KeyboardInterrupt:
            pass
        finally:
            await client.disconnect()

if __name__ == '__main__':
    import sys
    
    if len(sys.argv) > 1 and sys.argv[1] == 'client':
        asyncio.run(run_websocket_client())
    else:
        asyncio.run(run_websocket_server())
```
