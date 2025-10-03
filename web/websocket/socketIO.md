# Socket.IO Complete Cheat Sheet

## Table of Contents
- [Backend (Server)](#backend-server)
  - [Setup & Basic Connection](#setup--basic-connection)
  - [Socket Object Properties](#socket-object-properties)
  - [Event Handling](#event-handling)
  - [Emitting Events](#emitting-events)
  - [Room Management](#room-management)
  - [Socket Inspection & Management](#socket-inspection--management)
  - [Middleware](#middleware)
  - [Namespaces](#namespaces)
  - [Error Handling](#error-handling)
  - [Broadcasting Patterns](#broadcasting-patterns)
- [Frontend (Client)](#frontend-client)
  - [Setup & Connection](#setup--connection)
  - [Event Handling](#event-handling-1)
  - [Emitting Events](#emitting-events-1)
  - [Connection Management](#connection-management)
  - [Error Handling](#error-handling-1)
  - [Advanced Client Features](#advanced-client-features)

---

# Backend (Server)

## Setup & Basic Connection

### Installation & Setup
```javascript
npm install socket.io express

// Express + Socket.IO setup
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const server = createServer(app);
const io = new Server(server, {
  cors: {
    origin: "http://localhost:3000",
    methods: ["GET", "POST"]
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Basic Connection Handler
```javascript
io.on('connection', (socket) => {
  console.log('User connected:', socket.id);
  
  socket.on('disconnect', () => {
    console.log('User disconnected:', socket.id);
  });
});
```

## Socket Object Properties

### Essential Socket Properties
```javascript
io.on("connection", (socket) => {
  console.log(socket.id);           // unique socket ID
  console.log(socket.rooms);        // Set of rooms this socket has joined
  console.log(socket.handshake);    // connection details
  console.log(socket.connected);    // boolean: is connected?
  console.log(socket.disconnected); // boolean: is disconnected?
  console.log(socket.data);         // custom storage per socket
});
```

| Property | Description |
|----------|-------------|
| `socket.id` | Unique ID for this connection |
| `socket.rooms` | Set of room IDs this socket is in |
| `socket.handshake` | HTTP headers, auth data, cookies from initial connect |
| `socket.data` | Custom storage per socket (userId, deviceId, etc.) |
| `socket.connected` | Boolean: is socket still connected? |
| `socket.disconnected` | Boolean: is socket disconnected? |

### Handshake Information
```javascript
socket.handshake.headers    // HTTP headers
socket.handshake.time       // connection timestamp
socket.handshake.address    // client IP
socket.handshake.query      // query parameters
socket.handshake.auth       // authentication data
socket.handshake.cookies    // cookies
```

### Custom Socket Data
```javascript
// Store user information
socket.data.userId = "user123";
socket.data.username = "john_doe";
socket.data.deviceId = "device1";
socket.data.role = "admin";
```

## Event Handling

### Listen to Events
```javascript
socket.on('message', (data) => {
  console.log('Received:', data);
});

socket.on('chat_message', (message, callback) => {
  console.log('Chat message:', message);
  // Acknowledge receipt
  callback('Message received');
});

// Listen to any event
socket.onAny((eventName, ...args) => {
  console.log(`Event: ${eventName}`, args);
});

// Listen once
socket.once('initial_data', (data) => {
  console.log('Initial data received once:', data);
});
```

### Remove Event Listeners
```javascript
// Remove specific listener
socket.off('message', handlerFunction);

// Remove all listeners for an event
socket.removeAllListeners('message');

// Remove all listeners
socket.removeAllListeners();
```

## Emitting Events

### To Specific Socket(s)
```javascript
// To this socket only
socket.emit('event_name', payload);

// To a specific socket by ID
io.to('socketId123').emit('event_name', payload);

// With acknowledgment
socket.emit('event_name', payload, (response) => {
  console.log('Client responded:', response);
});
```

### Broadcasting
```javascript
// To all clients except sender
socket.broadcast.emit('event_name', payload);

// To all clients including sender
io.emit('event_name', payload);

// To all clients in a namespace
io.of('/admin').emit('event_name', payload);
```

## Room Management

### Joining/Leaving Rooms
```javascript
// Join single room
socket.join('room1');

// Join multiple rooms
socket.join(['room1', 'room2', 'room3']);

// Leave room
socket.leave('room1');

// Check which rooms socket is in
console.log(socket.rooms); // Set { socket.id, 'room1', 'room2' }
```

### Emitting to Rooms
```javascript
// To all in room (including sender)
io.to('room1').emit('event_name', payload);

// To all in room except sender
socket.to('room1').emit('event_name', payload);

// To multiple rooms
io.to('room1').to('room2').emit('event_name', payload);
socket.to(['room1', 'room2']).emit('event_name', payload);
```

### Room-based Broadcasting Patterns
```javascript
// Broadcast to room except sender
socket.to('room1').emit('user_joined', { user: socket.data.username });

// Emit to room including sender
io.to('room1').emit('room_message', { message: 'Hello everyone!' });

// Complex room targeting
socket.to('room1').to('room2').except('room3').emit('event', data);
```

## Socket Inspection & Management

### Get Sockets in Room
```javascript
// Get all sockets in a room (Socket.IO 4+)
const socketsInRoom = await io.in('room1').fetchSockets();

socketsInRoom.forEach(s => {
  console.log(s.id, s.data, s.rooms);
  s.emit('room_info', { roomSize: socketsInRoom.length });
});

// Count sockets in room
const roomSize = io.sockets.adapter.rooms.get('room1')?.size || 0;
console.log(`Room1 has ${roomSize} members`);
```

### Advanced Socket Filtering
```javascript
// Find specific user's sockets
const allSockets = await io.fetchSockets();
const userSockets = allSockets.filter(s => s.data.userId === 'user123');

// Send to specific user across all devices
userSockets.forEach(s => s.emit('user_notification', notification));

// Send to specific device of a user
const deviceSocket = userSockets.find(s => s.data.deviceId === 'mobile');
if (deviceSocket) {
  deviceSocket.emit('mobile_push', data);
}
```

### Room Information
```javascript
// Get all rooms
const rooms = io.sockets.adapter.rooms;

// Iterate through all rooms
for (let [roomName, roomSockets] of rooms) {
  console.log(`Room ${roomName} has ${roomSockets.size} sockets`);
}

// Check if room exists
const roomExists = io.sockets.adapter.rooms.has('room1');
```

## Middleware

### Connection Middleware
```javascript
// Authentication middleware
io.use((socket, next) => {
  const token = socket.handshake.auth.token;
  
  if (isValidToken(token)) {
    socket.data.userId = getUserIdFromToken(token);
    next();
  } else {
    next(new Error('Authentication failed'));
  }
});

// Rate limiting middleware
const rateLimiter = new Map();
io.use((socket, next) => {
  const clientId = socket.handshake.address;
  const now = Date.now();
  
  if (!rateLimiter.has(clientId)) {
    rateLimiter.set(clientId, []);
  }
  
  const connections = rateLimiter.get(clientId);
  const recentConnections = connections.filter(time => now - time < 60000);
  
  if (recentConnections.length > 10) {
    next(new Error('Rate limit exceeded'));
  } else {
    recentConnections.push(now);
    rateLimiter.set(clientId, recentConnections);
    next();
  }
});
```

### Event Middleware
```javascript
// Middleware for specific events
socket.use((packet, next) => {
  const [eventName, data] = packet;
  
  // Log all events
  console.log(`Event: ${eventName}`, data);
  
  // Validate event data
  if (eventName === 'chat_message' && !data.text) {
    return next(new Error('Message text required'));
  }
  
  next();
});
```

## Namespaces

### Creating Namespaces
```javascript
// Default namespace
const io = new Server(server);

// Custom namespaces
const adminNamespace = io.of('/admin');
const chatNamespace = io.of('/chat');

adminNamespace.on('connection', (socket) => {
  console.log('Admin connected:', socket.id);
});

chatNamespace.on('connection', (socket) => {
  console.log('Chat user connected:', socket.id);
});
```

### Namespace Operations
```javascript
// Emit to all in namespace
adminNamespace.emit('admin_broadcast', data);

// Emit to room in namespace
chatNamespace.to('room1').emit('chat_message', message);

// Get sockets in namespace
const adminSockets = await adminNamespace.fetchSockets();
```

## Error Handling

### Connection Errors
```javascript
io.on('connection', (socket) => {
  socket.on('error', (error) => {
    console.error('Socket error:', error);
  });
  
  socket.on('disconnect', (reason) => {
    console.log('Disconnected:', socket.id, 'Reason:', reason);
    
    if (reason === 'io server disconnect') {
      // Server forcefully disconnected the socket
    } else if (reason === 'io client disconnect') {
      // Client disconnected
    }
  });
});
```

### Emit with Error Handling
```javascript
socket.emit('risky_operation', data, (response) => {
  if (response.error) {
    console.error('Operation failed:', response.error);
  } else {
    console.log('Success:', response.data);
  }
});
```

## Broadcasting Patterns

### Common Broadcasting Scenarios
```javascript
// User joins room
socket.on('join_room', (roomId) => {
  socket.join(roomId);
  socket.to(roomId).emit('user_joined', {
    userId: socket.data.userId,
    username: socket.data.username
  });
});

// User leaves room
socket.on('leave_room', (roomId) => {
  socket.leave(roomId);
  socket.to(roomId).emit('user_left', {
    userId: socket.data.userId,
    username: socket.data.username
  });
});

// Typing indicators
socket.on('typing_start', (roomId) => {
  socket.to(roomId).emit('user_typing', {
    userId: socket.data.userId,
    isTyping: true
  });
});

// Private messaging
socket.on('private_message', (targetUserId, message) => {
  const targetSockets = await io.fetchSockets();
  const userSockets = targetSockets.filter(s => s.data.userId === targetUserId);
  
  userSockets.forEach(s => {
    s.emit('private_message', {
      from: socket.data.userId,
      message: message,
      timestamp: new Date()
    });
  });
});
```

---

# Frontend (Client)

## Setup & Connection

### Installation & Basic Setup
```javascript
// Install
npm install socket.io-client

// Import
import { io } from 'socket.io-client';

// Or CDN
<script src="/socket.io/socket.io.js"></script>
```

### Connection Options
```javascript
// Basic connection
const socket = io();

// With URL
const socket = io('http://localhost:3000');

// With options
const socket = io('http://localhost:3000', {
  transports: ['websocket', 'polling'],
  timeout: 20000,
  forceNew: true,
  autoConnect: false,
  auth: {
    token: 'your-auth-token'
  },
  query: {
    userId: '123',
    room: 'general'
  }
});

// Namespace connection
const adminSocket = io('/admin');
const chatSocket = io('/chat');
```

### Connection Management
```javascript
// Manual connection
socket.connect();

// Disconnect
socket.disconnect();

// Check connection status
console.log(socket.connected); // boolean
console.log(socket.id);         // socket ID when connected
```

## Event Handling

### Listen to Events
```javascript
// Basic event listener
socket.on('message', (data) => {
  console.log('Received:', data);
});

// With acknowledgment
socket.on('chat_message', (message, callback) => {
  console.log('Chat:', message);
  callback('Message received by client');
});

// Listen once
socket.once('welcome', (data) => {
  console.log('Welcome message:', data);
});

// Listen to any event
socket.onAny((eventName, ...args) => {
  console.log(`Event: ${eventName}`, args);
});
```

### Connection Events
```javascript
socket.on('connect', () => {
  console.log('Connected:', socket.id);
});

socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  
  if (reason === 'io server disconnect') {
    // Server disconnected, reconnect manually
    socket.connect();
  }
  // else: client will reconnect automatically
});

socket.on('connect_error', (error) => {
  console.error('Connection failed:', error);
});
```

## Emitting Events

### Basic Emitting
```javascript
// Simple emit
socket.emit('message', 'Hello Server!');

// With data object
socket.emit('chat_message', {
  text: 'Hello everyone!',
  timestamp: new Date(),
  userId: currentUser.id
});

// With acknowledgment
socket.emit('save_data', data, (response) => {
  if (response.success) {
    console.log('Data saved successfully');
  } else {
    console.error('Save failed:', response.error);
  }
});
```

### Timeout for Acknowledgments
```javascript
socket.timeout(5000).emit('slow_operation', data, (err, response) => {
  if (err) {
    console.error('Operation timed out');
  } else {
    console.log('Response:', response);
  }
});
```

## Connection Management

### Reconnection Settings
```javascript
const socket = io({
  reconnection: true,        // enable reconnection
  reconnectionAttempts: 5,   // max attempts
  reconnectionDelay: 1000,   // initial delay
  reconnectionDelayMax: 5000, // max delay
  randomizationFactor: 0.5   // randomization
});

// Reconnection events
socket.on('reconnect', (attemptNumber) => {
  console.log('Reconnected after', attemptNumber, 'attempts');
});

socket.on('reconnect_attempt', (attemptNumber) => {
  console.log('Reconnection attempt:', attemptNumber);
});

socket.on('reconnect_error', (error) => {
  console.error('Reconnection failed:', error);
});

socket.on('reconnect_failed', () => {
  console.error('All reconnection attempts failed');
});
```

### Manual Reconnection
```javascript
// Force reconnection
if (!socket.connected) {
  socket.connect();
}

// Disconnect and reconnect
socket.disconnect().connect();
```

## Error Handling

### Error Events
```javascript
socket.on('error', (error) => {
  console.error('Socket error:', error);
});

socket.on('connect_error', (error) => {
  console.error('Connection error:', error);
  
  // Handle specific errors
  if (error.message === 'Authentication failed') {
    // Redirect to login
    window.location.href = '/login';
  }
});
```

### Try-Catch with Async Operations
```javascript
async function sendMessage(message) {
  try {
    const response = await new Promise((resolve, reject) => {
      socket.emit('chat_message', message, (ack) => {
        if (ack.error) {
          reject(new Error(ack.error));
        } else {
          resolve(ack);
        }
      });
    });
    
    console.log('Message sent:', response);
  } catch (error) {
    console.error('Failed to send message:', error);
  }
}
```

## Advanced Client Features

### Custom Socket Instance
```javascript
class ChatClient {
  constructor(url, options = {}) {
    this.socket = io(url, options);
    this.setupEventHandlers();
  }
  
  setupEventHandlers() {
    this.socket.on('connect', () => {
      console.log('Chat client connected');
    });
    
    this.socket.on('message', this.handleMessage.bind(this));
  }
  
  handleMessage(message) {
    console.log('New message:', message);
  }
  
  sendMessage(text) {
    this.socket.emit('chat_message', {
      text,
      timestamp: new Date()
    });
  }
  
  joinRoom(roomId) {
    this.socket.emit('join_room', roomId);
  }
  
  disconnect() {
    this.socket.disconnect();
  }
}

// Usage
const chatClient = new ChatClient('http://localhost:3000');
```

### React Hook for Socket.IO
```javascript
import { useEffect, useState } from 'react';
import { io } from 'socket.io-client';

function useSocket(url, options = {}) {
  const [socket, setSocket] = useState(null);
  const [connected, setConnected] = useState(false);
  
  useEffect(() => {
    const newSocket = io(url, options);
    
    newSocket.on('connect', () => setConnected(true));
    newSocket.on('disconnect', () => setConnected(false));
    
    setSocket(newSocket);
    
    return () => {
      newSocket.close();
    };
  }, [url]);
  
  return { socket, connected };
}

// Usage in component
function ChatComponent() {
  const { socket, connected } = useSocket('http://localhost:3000');
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    if (!socket) return;
    
    socket.on('chat_message', (message) => {
      setMessages(prev => [...prev, message]);
    });
    
    return () => {
      socket.off('chat_message');
    };
  }, [socket]);
  
  const sendMessage = (text) => {
    if (socket && connected) {
      socket.emit('chat_message', { text, timestamp: new Date() });
    }
  };
  
  return (
    <div>
      <div>Status: {connected ? 'Connected' : 'Disconnected'}</div>
      {/* Chat UI */}
    </div>
  );
}
```

### Buffered Events
```javascript
// Events sent before connection will be buffered
const socket = io({ autoConnect: false });

socket.emit('early_event', 'data'); // buffered
socket.connect(); // events sent after connection

// Disable buffering
const socket = io({ 
  autoConnect: false,
  buffer: false 
});
```

### Volatile Events
```javascript
// Events that can be dropped if client is not ready
socket.volatile.emit('mouse_position', { x: 100, y: 200 });

// Useful for high-frequency events like mouse movement
// where missing a few events doesn't matter
```

---

## Quick Reference

### Server Events Summary
| Action | Method |
|--------|--------|
| Emit to this socket | `socket.emit(event, data)` |
| Emit to room | `io.to(room).emit(event, data)` |
| Emit to room except sender | `socket.to(room).emit(event, data)` |
| Broadcast to all except sender | `socket.broadcast.emit(event, data)` |
| Join room | `socket.join(room)` |
| Leave room | `socket.leave(room)` |
| Get room sockets | `await io.in(room).fetchSockets()` |
| Room socket count | `io.sockets.adapter.rooms.get(room)?.size` |

### Client Events Summary
| Action | Method |
|--------|--------|
| Connect | `socket.connect()` |
| Disconnect | `socket.disconnect()` |
| Emit event | `socket.emit(event, data)` |
| Listen to event | `socket.on(event, handler)` |
| Listen once | `socket.once(event, handler)` |
| Remove listener | `socket.off(event, handler)` |
| Check connection | `socket.connected` |
| Get socket ID | `socket.id` |