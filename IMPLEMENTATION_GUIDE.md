# GigFlow Real-Time Notifications & Transactional Integrity Implementation

## Overview

This document describes the implementation of two critical features:

1. **Transactional Integrity** - Preventing race conditions during the hiring process
2. **Real-time Updates** - Socket.io integration for instant notifications

---

## 1. Transactional Integrity (Race Conditions Prevention)

### Problem

When two users (e.g., project admins) click "Hire" on different freelancers simultaneously, both requests could succeed, violating the constraint that only one freelancer should be hired per gig.

### Solution: MongoDB Transactions

#### How It Works

- Uses MongoDB Session and Transactions to ensure atomic operations
- All database modifications happen together or not at all
- Prevents dirty reads, phantom reads, and race conditions

#### Implementation in `bidController.js` - `hireBid()` function

```javascript
// Step 1: Start a transaction session
const session = await Bid.startSession();
session.startTransaction();

try {
  // Step 2: Check if gig is still open within the transaction
  // This is CRITICAL - the check happens inside the transaction
  if (gig.status !== "open") {
    await session.abortTransaction();
    return res.status(400).json({
      success: false,
      message: "This gig is already assigned",
    });
  }

  // Step 3: Update selected bid to hired
  const hiredBid = await Bid.findByIdAndUpdate(
    bidId,
    { status: "hired", hiredAt: new Date() },
    { new: true, session }
  );

  // Step 4: Reject all other bids
  await Bid.updateMany(
    { gigId: bid.gigId, _id: { $ne: bidId } },
    { status: "rejected", rejectedAt: new Date() },
    { session }
  );

  // Step 5: Update gig status to assigned
  const updatedGig = await Gig.findByIdAndUpdate(
    bid.gigId,
    { status: "assigned" },
    { new: true, session }
  );

  // Step 6: Commit transaction (all changes succeed together)
  await session.commitTransaction();
} catch (error) {
  // On error: rollback all changes
  await session.abortTransaction();
} finally {
  await session.endSession();
}
```

#### Race Condition Prevention Flow

```
Request A                          Request B
(Admin 1 hires Freelancer 1)      (Admin 2 hires Freelancer 2)
         |                              |
    Check gig.status                Check gig.status
    (OPEN) ✓                        (OPEN) ✓
         |                              |
    Lock acquired                   Waiting for lock
         |                              |
    Update bid 1 → hired            [BLOCKED]
    Update other bids → rejected
    Update gig → assigned
         |
    Commit transaction
    Lock released
                                        |
                                   Lock acquired
                                        |
                                   Check gig.status
                                   (ASSIGNED) ✗
                                   Abort & return error
```

#### Key Benefits

- Only one hire succeeds per gig
- All related data stays consistent
- No orphaned bids or invalid states
- Automatic rollback on failure

---

## 2. Real-Time Updates via Socket.io

### Problem

Freelancers need instant notifications when hired, without page refresh.

### Solution: Socket.io Integration

#### Architecture Overview

```
Client                    Server
  |                         |
  |--- Socket Connect ----> |
  |                    io.on("connection")
  |                         |
  |--- register_user -----> | Stores userId → socketId mapping
  |                         |
  |                    [Hire Action on another client]
  |                         |
  |                    io.to(socketId).emit("hire_notification")
  |                         |
  |<---- hire_notification --|
  |                         |
  Display toast notification
```

### Implementation Details

#### Backend Setup - `server.js`

```javascript
const http = require("http");
const socketIO = require("socket.io");

// Create HTTP server and attach Socket.io
const server = http.createServer(app);
const io = socketIO(server, {
  cors: {
    origin: process.env.FRONTEND_URL,
    methods: ["GET", "POST"],
    credentials: true,
  },
});

// Store active user connections
const userSockets = new Map(); // userId → socketId

// Handle socket connections
io.on("connection", (socket) => {
  socket.on("register_user", (userId) => {
    userSockets.set(userId, socket.id);
    console.log(`User ${userId} registered with socket ${socket.id}`);
  });

  socket.on("disconnect", () => {
    if (socket.userId) {
      userSockets.delete(socket.userId);
    }
  });
});

// Make io and userSockets available to routes
app.use((req, res, next) => {
  req.io = io;
  req.userSockets = userSockets;
  next();
});

// Change app.listen to server.listen
server.listen(PORT, () => {
  console.log(`Server is running successfully`);
});
```

#### Notification Service - `services/notificationService.js`

Centralized service for sending notifications:

```javascript
class NotificationService {
  initialize(io, userSockets) {
    this.io = io;
    this.userSockets = userSockets;
  }

  notifyFreelancerHired(freelancerId, gigTitle, bidData) {
    const socketId = this.userSockets.get(freelancerId);
    if (socketId) {
      this.io.to(socketId).emit("hire_notification", {
        type: "hired",
        message: `You have been hired for "${gigTitle}"!`,
        gigId: bidData.gigId,
        budget: bidData.budget,
        clientName: bidData.clientName,
        timestamp: new Date().toISOString(),
      });
    }
  }
}
```

#### Frontend Setup - `hooks/useNotifications.js`

Custom React hook for managing socket connections:

```javascript
export const useNotifications = (userId, isAuthenticated) => {
  const [notifications, setNotifications] = useState([]);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    if (!isAuthenticated || !userId) return;

    // Connect to Socket.io server
    const socketInstance = io("http://localhost", {
      reconnection: true,
      reconnectionDelay: 1000,
      reconnectionAttempts: 5,
    });

    socketInstance.on("connect", () => {
      setIsConnected(true);
      // Register this user
      socketInstance.emit("register_user", userId);
    });

    // Listen for hire notifications
    socketInstance.on("hire_notification", (notification) => {
      addNotification(notification);
    });

    return () => socketInstance.disconnect();
  }, [userId, isAuthenticated]);

  return { notifications, isConnected, removeNotification };
};
```

#### Notification Component - `components/Notifications.jsx`

Displays toast-style notifications:

```javascript
const Notifications = ({ notifications, onRemove }) => {
  return (
    <div className="fixed top-4 right-4 z-50 space-y-3">
      {notifications.map((notification) => (
        <div key={notification.id} className="notification-toast">
          {notification.type === "hired" && (
            <div className="Congratulations">{notification.message}</div>
          )}
          {/* Budget and client info displayed */}
        </div>
      ))}
    </div>
  );
};
```

#### Integration in App - `App.jsx`

```javascript
function App() {
  const auth = useSelector((state) => state.auth);
  const { notifications, isConnected, removeNotification } = useNotifications(
    auth.user?._id,
    auth.isAuthenticated
  );

  return (
    <>
      <Navbar />
      <Notifications
        notifications={notifications}
        onRemove={removeNotification}
      />
      {isConnected && <div>✓ Connected</div>}
      {/* Routes */}
    </>
  );
}
```

---

## 3. Updated Data Models

### Bid Schema Enhancements

Added timestamp fields for better tracking:

```javascript
{
  gigId: ObjectId,
  freelancerId: ObjectId,
  message: String,
  status: "pending" | "hired" | "rejected",
  hiredAt: Date,        // ← NEW: When freelancer was hired
  rejectedAt: Date,     // ← NEW: When bid was rejected
  createdAt: Date,      // Existing
  updatedAt: Date,      // Existing
}
```

---

## 4. Notification Flow Example

### Scenario: Hiring a Freelancer

**User 1 (Client) Actions:**

1. Views bids for a gig
2. Clicks "Hire" on Freelancer A's bid

**Server Processing (Transactional):**

1. Start transaction
2. Lock and check gig status (must be "open")
3. Update Bid A → status: "hired", hiredAt: now
4. Update all other bids → status: "rejected", rejectedAt: now
5. Update Gig → status: "assigned"
6. Commit transaction

**Real-Time Notification:**

1. Server looks up Freelancer A's socket ID
2. Sends `hire_notification` event
3. Freelancer A's browser receives event instantly
4. Toast notification appears: "You have been hired for [Gig Title]!"

**If User 2 tries simultaneously:**

1. Both requests acquire transaction lock sequentially
2. First request succeeds and commits
3. Second request checks gig.status → sees "assigned"
4. Transaction aborts with error: "This gig is already assigned"

---

## 5. Configuration

### Environment Variables

Add to `.env`:

```env
# Backend
FRONTEND_URL=http://localhost
PORT=5000

# Frontend
VITE_BACKEND_URL=http://localhost
```

### Frontend Vite Configuration

In `gigflow-frontend/vite.config.js`:

```javascript
export default {
  define: {
    "import.meta.env.VITE_BACKEND_URL": JSON.stringify(
      process.env.VITE_BACKEND_URL || "http://localhost"
    ),
  },
};
```

---

## 6. Testing the Implementation

### Test Case 1: Simultaneous Hiring (Race Condition)

```bash
# Terminal 1: Start backend
npm run dev

# Terminal 2 & 3: Simulate concurrent hire requests
curl -X PATCH http://localhost/api/bids/bid1/hire \
  -H "Authorization: Bearer TOKEN1"

curl -X PATCH http://localhost/api/bids/bid2/hire \
  -H "Authorization: Bearer TOKEN2"

# Expected: One succeeds, one fails with "gig already assigned"
```

### Test Case 2: Real-time Notification

```javascript
// Open browser console, then:

// In one browser (Freelancer)
io("http://localhost").on("hire_notification", (data) => {
  console.log("Notified:", data);
});

// From another browser (Client) - hire freelancer
// Should see notification appear instantly in Freelancer's browser
```

---

## 7. Files Modified/Created

### Backend

- `src/server.js` - Socket.io server setup
- `src/services/notificationService.js` - Notification logic (NEW)
- `src/controllers/bidController.js` - Enhanced with transactions & notifications
- `src/models/Bid.js` - Added hiredAt & rejectedAt fields
- `package.json` - Added socket.io

### Frontend

- `src/App.jsx` - Integrated notifications system
- `src/components/Notifications.jsx` - Toast notifications (NEW)
- `src/hooks/useNotifications.js` - Socket connection hook (NEW)
- `package.json` - Added socket.io-client

---

## 8. Benefits & Security

### Transactional Integrity Benefits

- 🔒 **Atomicity**: All-or-nothing operations
- **Consistency**: Database never in invalid state
- **Prevents double-booking**: Only one hire per gig
- 🛡️ **Automatic rollback**: Errors don't leave partial updates

### Real-time Notification Benefits

- ⚡ **Instant updates**: No polling or page refresh needed
- 📱 **Better UX**: Users feel immediate feedback
- 💬 **Two-way communication**: Bi-directional messaging
- 🔌 **Auto-reconnection**: Handles network interruptions

---

## 9. Scalability Considerations

For production deployments with multiple servers:

```javascript
// Use Redis adapter for Socket.io
const { createAdapter } = require("@socket.io/redis-adapter");
const { createClient } = require("redis");

const pubClient = createClient();
const subClient = pubClient.duplicate();

io.adapter(createAdapter(pubClient, subClient));
```

This allows notifications across multiple server instances.

---

## Summary

**Transactional Integrity**: MongoDB transactions prevent race conditions  
**Real-time Notifications**: Socket.io enables instant communication  
**Seamless Integration**: Works with existing REST API  
**Production-Ready**: Error handling, auto-reconnect, graceful degradation
