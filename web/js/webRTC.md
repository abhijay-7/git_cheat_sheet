# WebRTC Cheatsheet

## Overview
WebRTC (Web Real-Time Communication) enables peer-to-peer audio, video, and data sharing between browsers without plugins.

## Core Concepts

### Three Main Components
- **MediaStream**: Captures audio/video from user devices
- **RTCPeerConnection**: Manages peer-to-peer connection
- **RTCDataChannel**: Enables bidirectional data transfer

### Connection Flow
1. **Signaling**: Exchange connection info via signaling server
2. **Offer/Answer**: Create and exchange session descriptions
3. **ICE**: Find best network path through NAT/firewalls
4. **Media Flow**: Direct peer-to-peer communication

## Essential APIs

### Getting User Media
```javascript
// Get camera and microphone
const stream = await navigator.mediaDevices.getUserMedia({
  video: true,
  audio: true
});

// Get screen share
const screenStream = await navigator.mediaDevices.getDisplayMedia({
  video: true,
  audio: true
});

// List available devices
const devices = await navigator.mediaDevices.enumerateDevices();
```

### RTCPeerConnection Setup
```javascript
const configuration = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' },
    {
      urls: 'turn:your-turn-server.com:3478',
      username: 'user',
      credential: 'pass'
    }
  ]
};

const pc = new RTCPeerConnection(configuration);
```

### Connection Establishment
```javascript
// Caller side
async function createOffer() {
  const offer = await pc.createOffer();
  await pc.setLocalDescription(offer);
  // Send offer to remote peer via signaling
  sendToRemotePeer({ type: 'offer', sdp: offer });
}

// Callee side
async function handleOffer(offer) {
  await pc.setRemoteDescription(offer);
  const answer = await pc.createAnswer();
  await pc.setLocalDescription(answer);
  // Send answer back via signaling
  sendToRemotePeer({ type: 'answer', sdp: answer });
}

// Both sides handle ICE candidates
pc.onicecandidate = (event) => {
  if (event.candidate) {
    sendToRemotePeer({ type: 'ice-candidate', candidate: event.candidate });
  }
};

async function handleIceCandidate(candidate) {
  await pc.addIceCandidate(candidate);
}
```

## Media Handling

### Adding Local Stream
```javascript
// Add tracks to peer connection
stream.getTracks().forEach(track => {
  pc.addTrack(track, stream);
});
```

### Receiving Remote Stream
```javascript
pc.ontrack = (event) => {
  const remoteVideo = document.getElementById('remoteVideo');
  remoteVideo.srcObject = event.streams[0];
};
```

### Track Management
```javascript
// Replace video track (e.g., switch camera)
const videoTrack = stream.getVideoTracks()[0];
const sender = pc.getSenders().find(s => s.track?.kind === 'video');
await sender.replaceTrack(newVideoTrack);

// Remove track
pc.removeTrack(sender);

// Mute/unmute
track.enabled = false; // mute
track.enabled = true;  // unmute
```

## Data Channels

### Creating Data Channel
```javascript
// Caller creates channel
const dataChannel = pc.createDataChannel('messages', {
  ordered: true
});

dataChannel.onopen = () => console.log('Data channel opened');
dataChannel.onmessage = (event) => console.log('Received:', event.data);

// Send data
dataChannel.send('Hello World');
dataChannel.send(JSON.stringify({ type: 'chat', message: 'Hi!' }));
```

### Receiving Data Channel
```javascript
// Callee receives channel
pc.ondatachannel = (event) => {
  const channel = event.channel;
  
  channel.onopen = () => console.log('Data channel opened');
  channel.onmessage = (event) => {
    console.log('Received:', event.data);
  };
};
```

## Connection States & Events

### Key Event Listeners
```javascript
pc.onconnectionstatechange = () => {
  console.log('Connection state:', pc.connectionState);
  // States: new, connecting, connected, disconnected, failed, closed
};

pc.oniceconnectionstatechange = () => {
  console.log('ICE state:', pc.iceConnectionState);
  // States: new, checking, connected, completed, failed, disconnected, closed
};

pc.onsignalingstatechange = () => {
  console.log('Signaling state:', pc.signalingState);
  // States: stable, have-local-offer, have-remote-offer, have-local-pranswer, have-remote-pranswer, closed
};
```

### Connection Monitoring
```javascript
// Get connection statistics
const stats = await pc.getStats();
stats.forEach(report => {
  if (report.type === 'inbound-rtp' && report.kind === 'video') {
    console.log('Video bytes received:', report.bytesReceived);
  }
});
```

## Common Patterns

### Complete Peer Connection Setup
```javascript
class WebRTCConnection {
  constructor(signalingSocket) {
    this.pc = new RTCPeerConnection({
      iceServers: [{ urls: 'stun:stun.l.google.com:19302' }]
    });
    this.socket = signalingSocket;
    this.setupEventHandlers();
  }

  setupEventHandlers() {
    this.pc.onicecandidate = (event) => {
      if (event.candidate) {
        this.socket.send(JSON.stringify({
          type: 'ice-candidate',
          candidate: event.candidate
        }));
      }
    };

    this.pc.ontrack = (event) => {
      document.getElementById('remoteVideo').srcObject = event.streams[0];
    };

    this.socket.onmessage = async (event) => {
      const data = JSON.parse(event.data);
      
      switch (data.type) {
        case 'offer':
          await this.handleOffer(data.offer);
          break;
        case 'answer':
          await this.handleAnswer(data.answer);
          break;
        case 'ice-candidate':
          await this.pc.addIceCandidate(data.candidate);
          break;
      }
    };
  }

  async createOffer() {
    const offer = await this.pc.createOffer();
    await this.pc.setLocalDescription(offer);
    this.socket.send(JSON.stringify({ type: 'offer', offer }));
  }

  async handleOffer(offer) {
    await this.pc.setRemoteDescription(offer);
    const answer = await this.pc.createAnswer();
    await this.pc.setLocalDescription(answer);
    this.socket.send(JSON.stringify({ type: 'answer', answer }));
  }

  async handleAnswer(answer) {
    await this.pc.setRemoteDescription(answer);
  }
}
```

### Error Handling
```javascript
// Wrap WebRTC calls in try-catch
try {
  const stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
} catch (error) {
  if (error.name === 'NotAllowedError') {
    console.log('User denied media access');
  } else if (error.name === 'NotFoundError') {
    console.log('No media devices found');
  }
}

// Handle connection failures
pc.oniceconnectionstatechange = () => {
  if (pc.iceConnectionState === 'failed') {
    // Attempt ICE restart
    pc.restartIce();
  }
};
```

## Server Requirements

### STUN/TURN Servers
```javascript
const iceServers = [
  // STUN server (for NAT traversal)
  { urls: 'stun:stun.l.google.com:19302' },
  
  // TURN server (for restrictive networks)
  {
    urls: 'turn:turnserver.com:3478',
    username: 'user',
    credential: 'password'
  }
];
```

### Signaling Server (Node.js + Socket.IO)
```javascript
const io = require('socket.io')(server);

io.on('connection', (socket) => {
  socket.on('join-room', (roomId) => {
    socket.join(roomId);
    socket.to(roomId).emit('user-joined', socket.id);
  });

  socket.on('offer', (data) => {
    socket.to(data.roomId).emit('offer', {
      offer: data.offer,
      from: socket.id
    });
  });

  socket.on('answer', (data) => {
    socket.to(data.to).emit('answer', {
      answer: data.answer,
      from: socket.id
    });
  });

  socket.on('ice-candidate', (data) => {
    socket.to(data.to).emit('ice-candidate', {
      candidate: data.candidate,
      from: socket.id
    });
  });
});
```

## Security Considerations

- **HTTPS Required**: WebRTC only works over secure connections
- **User Consent**: Always request permission for media access
- **TURN Authentication**: Secure TURN servers with time-limited credentials
- **Data Validation**: Validate all signaling messages
- **Origin Checks**: Verify message origins in signaling server

## Browser Compatibility

- **Modern Support**: Chrome 23+, Firefox 22+, Safari 11+, Edge 79+
- **Mobile**: iOS Safari 11+, Chrome Mobile 28+
- **Feature Detection**: Always check for WebRTC support

```javascript
if (!navigator.mediaDevices?.getUserMedia) {
  console.log('WebRTC not supported');
  return;
}
```

## Debugging Tips

### Enable WebRTC Logs
```javascript
// Chrome: chrome://webrtc-internals/
// Firefox: about:webrtc

// Console debugging
pc.addEventListener('connectionstatechange', () => {
  console.log('Connection state:', pc.connectionState);
});
```

### Common Issues
- **ICE Connection Failed**: Check STUN/TURN configuration
- **Media Not Flowing**: Verify track addition and HTTPS
- **Signaling Issues**: Ensure proper offer/answer exchange
- **Firewall Problems**: Test with TURN server

## Performance Optimization

```javascript
// Adaptive bitrate
const sender = pc.getSenders().find(s => s.track?.kind === 'video');
const params = sender.getParameters();
params.encodings[0].maxBitrate = 1000000; // 1 Mbps
await sender.setParameters(params);

// Video constraints
const stream = await navigator.mediaDevices.getUserMedia({
  video: {
    width: { ideal: 1280 },
    height: { ideal: 720 },
    frameRate: { ideal: 30 }
  },
  audio: true
});
```

## Quick Reference

### Connection States
- `new` → `connecting` → `connected` → `disconnected`/`failed` → `closed`

### Media Constraints
```javascript
{
  video: {
    width: { min: 640, ideal: 1280, max: 1920 },
    height: { min: 480, ideal: 720, max: 1080 },
    frameRate: { ideal: 30, max: 60 }
  },
  audio: {
    echoCancellation: true,
    noiseSuppression: true
  }
}
```

### Essential Methods
- `createOffer()` / `createAnswer()`
- `setLocalDescription()` / `setRemoteDescription()`
- `addIceCandidate()`
- `addTrack()` / `removeTrack()`
- `getStats()`

```
//code snippet to record video from console:
// Create a button in the page
const btn = document.createElement("button");
btn.textContent = "Start Screen Recording";
document.body.appendChild(btn);

btn.onclick = async () => {
  try {
    // Prompt user to share their screen
    const stream = await navigator.mediaDevices.getDisplayMedia({
      video: true,
      audio: true
    });

    // Set up recorder
    const recorder = new MediaRecorder(stream);
    const chunks = [];

    recorder.ondataavailable = e => chunks.push(e.data);
    recorder.onstop = () => {
      // Combine chunks into a blob
      const blob = new Blob(chunks, { type: "video/webm" });
      const url = URL.createObjectURL(blob);

      // Create a link to download the recording
      const a = document.createElement("a");
      a.href = url;
      a.download = "recording.webm";
      a.textContent = "Download recording";
      document.body.appendChild(a);
    };

    recorder.start();
    btn.textContent = "Stop Recording";

    btn.onclick = () => {
      recorder.stop();
      stream.getTracks().forEach(track => track.stop());
      btn.remove();
    };
  } catch (err) {
    console.error("Error starting screen recording:", err);
  }
};


```