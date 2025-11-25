# Socket.io Integration Guide

## Server Setup

The WebSocket server is now fully integrated with your chat app backend.

## Environment Variables

Add to your `.env` file:
```
CLIENT_URL=http://localhost:5173
```

## Client-Side Integration

### 1. Install Socket.io Client
```bash
npm install socket.io-client
```

### 2. Connect to Socket.io Server

```javascript
import { io } from 'socket.io-client';

// Connect with userId from authentication
const socket = io('http://localhost:8080', {
  query: {
    userId: currentUser._id // Pass the logged-in user's ID
  }
});

// Listen for connection
socket.on('connect', () => {
  console.log('Connected to server');
});
```

### 3. Listen for Real-Time Events

**Receive New Messages:**
```javascript
socket.on('newMessage', (message) => {
  // Handle incoming message
  console.log('New message:', message);
  // Update your UI with the new message
});
```

**Track Online Users:**
```javascript
socket.on('getOnlineUsers', (userIds) => {
  // userIds is an array of user IDs that are currently online
  console.log('Online users:', userIds);
  // Update your UI to show who's online
});
```

### 4. Handle Disconnection

```javascript
socket.on('disconnect', () => {
  console.log('Disconnected from server');
});
```

## How It Works

1. **User connects:** When a user logs in, connect to Socket.io with their userId
2. **Online status:** Server automatically tracks and broadcasts online users
3. **Real-time messages:** When someone sends a message, the recipient receives it instantly via Socket.io
4. **Auto cleanup:** When user disconnects/logs out, they're removed from online users

## API Changes

### Send Message (`POST /api/message/send/:id`)
- Now emits messages in real-time to the recipient
- Validates that message contains text or image
- Checks if recipient exists

### Get Messages (`GET /api/message/:id`)
- Added pagination support: `?page=1&limit=50`
- Sorted by creation time (oldest first)

### Get Users (`GET /api/message/users`)
- Returns all non-banned users (not just online)
- Use Socket.io's `getOnlineUsers` event to track who's actually online

## Example Frontend Flow

```javascript
// On login/app mount
const socket = io('http://localhost:8080', {
  query: { userId: user._id }
});

// Listen for online users
socket.on('getOnlineUsers', (onlineUserIds) => {
  setOnlineUsers(onlineUserIds);
});

// Listen for new messages
socket.on('newMessage', (message) => {
  if (message.senderId === selectedChat._id) {
    setMessages(prev => [...prev, message]);
  }
  // Optionally show notification if chat is not active
});

// On logout/unmount
socket.disconnect();
```

## Features Implemented

✅ Real-time message delivery
✅ Online/offline status tracking
✅ Automatic cleanup on disconnect
✅ Message validation
✅ Pagination for message history
✅ Database indexes for performance
✅ All bugs fixed

## Notes

- Online status is now properly saved to database
- Socket.io handles real-time updates
- Users appear in sidebar regardless of online status
- Use Socket.io events to show online indicators in UI
