# Real-Time Notifications & Transactional Integrity - Setup & Testing

## New Features Added

### 1. **Transactional Integrity (Race Condition Prevention)**

- MongoDB Transactions prevent two simultaneous hire requests from both succeeding
- Only ONE freelancer can be hired per gig, no matter how many concurrent requests
- Automatic rollback on conflict

### 2. **Real-Time Notifications via Socket.io**

- Freelancers receive instant notifications when hired
- No page refresh needed
- Toast-style notifications with budget and client info
- Connection status indicator

---

## Installation Status

### Already Installed:

- `socket.io` (backend)
- `socket.io-client` (frontend)

### Files Created:

**Backend:**

- `src/services/notificationService.js` - Notification handler
- `src/server.js` - Updated with Socket.io server
- `src/controllers/bidController.js` - Enhanced with transactions
- `src/models/Bid.js` - Added hiredAt, rejectedAt fields

**Frontend:**

- `src/hooks/useNotifications.js` - Socket connection hook
- `src/components/Notifications.jsx` - Toast notifications
- `src/App.jsx` - Integrated notifications

---

## Quick Start

### 1. Install Lucide Icons (for notification icons)

```bash
cd gigflow-frontend
npm install lucide-react --save
```

### 2. Update Environment Variables

**`gigflow-backend/.env`**

```env
FRONTEND_URL=http://localhost:5173
PORT=5000
```

**`gigflow-frontend/.env.local`** (create if doesn't exist)

```env
VITE_BACKEND_URL=http://localhost:5000
```

### 3. Start Servers

**Terminal 1: Backend**

```bash
cd gigflow-backend
npm run dev
```

**Terminal 2: Frontend**

```bash
cd gigflow-frontend
npm run dev
```

---

## Testing Guide

### Test 1: Real-Time Notifications

**Objective:** Verify freelancer receives instant notification when hired

**Setup:**

1. Open Firefox (as Freelancer) - Login as User A
2. Open Chrome (as Client) - Login as User B

**Steps:**

1. **In Firefox (Freelancer):**

   - Navigate to any page
   - Open DevTools Console (F12)
   - Should see: `Socket connected: [socket-id]`
   - Should see: `User [userId] registered with socket [socket-id]`

2. **In Chrome (Client):**

   - Go to Browse Gigs
   - Click on a gig that has bids from User A
   - Click the "Hire" button on User A's bid

3. **Back in Firefox (Freelancer):**
   - Toast notification appears instantly
   - Message: "You have been hired for [Gig Title]!"
   - Shows budget and client name
   - Notification auto-dismisses after 5 seconds

**Success Criteria:**

- Notification appears without page refresh
- Correct gig title displayed
- Budget and client name shown
- Connection indicator shows `✓ Connected`

---

### Test 2: Race Condition Prevention 🔒

**Objective:** Verify only ONE freelancer is hired when two simultaneous hire requests occur

**Scenario:** Two project admins try to hire different freelancers at the same time

**Setup:**

1. Create a gig with 3+ bids
2. Login as gig owner
3. Open 2 browser windows (both as gig owner)

**Manual Test with Console:**

**Window 1 - Run in Console:**

```javascript
// Get actual bid IDs from the page or API first
const gigId = "ACTUAL_GIG_ID"; // From URL or API

// Simulate hiring Bid 1
setTimeout(() => {
  fetch("/api/bids/BID_1_ID/hire", {
    method: "PATCH",
    headers: {
      Authorization: `Bearer ${localStorage.getItem("token")}`,
      "Content-Type": "application/json",
    },
  })
    .then((r) => r.json())
    .then((d) => console.log("Bid 1 Result:", d));
}, 0);
```

**Window 2 - Run in Console (immediately after Window 1):**

```javascript
const gigId = "ACTUAL_GIG_ID";

// Simulate hiring Bid 2 simultaneously
setTimeout(() => {
  fetch("/api/bids/BID_2_ID/hire", {
    method: "PATCH",
    headers: {
      Authorization: `Bearer ${localStorage.getItem("token")}`,
      "Content-Type": "application/json",
    },
  })
    .then((r) => r.json())
    .then((d) => console.log("Bid 2 Result:", d));
}, 0);
```

**Expected Results:**

- Bid 1: `{ success: true, message: "Freelancer hired successfully", data: {...} }`
- Bid 2: `{ success: false, message: "This gig is already assigned", error: true }`

**Why?**

- Both requests start transaction
- Bid 1 acquires lock, checks gig.status="open" ✓
- Bid 1 updates: gig.status="assigned"
- Bid 1 commits
- Bid 2 acquires lock, checks gig.status="open" ✗ (now "assigned")
- Bid 2 aborts with error

**Success Criteria:**

- Exactly ONE bid status becomes "hired"
- All other bids become "rejected"
- Gig status is "assigned"
- Freelancer receives notification only for hired bid
- Rejected freelancers receive rejection notifications

---

### Test 3: Rejection Notifications

**Objective:** Verify rejected bidders are notified

**Steps:**

1. Open 3 browsers as different users: Freelancer 1, Freelancer 2, Client
2. Both freelancers bid on same gig (from Client)
3. Client hires Freelancer 1

**Expected:**

- Freelancer 1: Gets "hired" notification
- Freelancer 2: Gets "rejected" notification
- Both show different toast messages
- Both show timestamp

---

## Data Model Changes

### Bid Schema

```javascript
{
  _id: ObjectId,
  gigId: ObjectId,           // Gig reference
  freelancerId: ObjectId,    // Freelancer reference
  message: String,           // Bid message
  status: String,            // "pending" | "hired" | "rejected"
  hiredAt: Date,             // ← NEW: When hired
  rejectedAt: Date,          // ← NEW: When rejected
  createdAt: Date,           // Auto timestamps
  updatedAt: Date            // Auto timestamps
}
```

### Example Bid Document

```json
{
  "_id": "507f1f77bcf86cd799439011",
  "gigId": "507f1f77bcf86cd799439012",
  "freelancerId": "507f1f77bcf86cd799439013",
  "message": "I can do this project in 2 weeks",
  "status": "hired",
  "hiredAt": "2024-01-11T10:30:00Z",
  "rejectedAt": null,
  "createdAt": "2024-01-11T09:00:00Z",
  "updatedAt": "2024-01-11T10:30:00Z"
}
```

---

## Socket.io Events Reference

### Client → Server

**`register_user`** - Register user for notifications

```javascript
socket.emit("register_user", userId);
```

### Server → Client

**`hire_notification`** - Fired when user is hired OR bid rejected

Hire Event:

```javascript
{
  type: 'hired',
  message: 'You have been hired for "Build iPhone App"!',
  bidId: '507f1f77bcf86cd799439011',
  gigId: '507f1f77bcf86cd799439012',
  budget: 5000,
  clientName: 'John Developer',
  timestamp: '2024-01-11T10:30:00Z'
}
```

Rejection Event:

```javascript
{
  type: 'rejected',
  message: 'Your bid for "Build iPhone App" was not selected.',
  bidId: '507f1f77bcf86cd799439011',
  gigId: '507f1f77bcf86cd799439012',
  timestamp: '2024-01-11T10:30:00Z'
}
```

---

## Debugging

### Check Socket Connection

```javascript
// In browser console
console.log("Is connected:", io.connected);
console.log("Socket ID:", io.id);

// Check if user registered
console.log("User ID:", localStorage.getItem("userId"));
```

### Check Bid Status

```javascript
// Fetch a bid to verify status
fetch("/api/bids/BID_ID")
  .then((r) => r.json())
  .then((d) => console.log("Bid:", d.data));
```

### Check Server Logs

Look for:

- `Socket connected: [socket-id]`
- `User [userId] registered with socket [socket-id]`
- `Hire notification sent to freelancer...`

---

## 🚨 Common Issues & Solutions

### Issue: Notifications not appearing

**Solution:**

1. Check frontend is connected: `✓ Connected` badge should show
2. Check backend logs: Should see user registration
3. Verify user logged in: Check Redux store has auth.user
4. Ensure lucide-react installed: `npm install lucide-react`

### Issue: Toast component errors

**Solution:**

1. Make sure `lucide-react` is installed
2. Check imports in Notifications.jsx
3. Check Tailwind CSS configured properly

### Issue: Race condition test not working

**Solution:**

1. Use actual Bid IDs from API response
2. Run both fetch commands within 100ms of each other
3. Check MongoDB is running (needed for transactions)
4. Verify Mongoose session is created

### Issue: Real-time notification but on wrong user

**Solution:**

1. Verify you're using correct user IDs
2. Check `register_user` event being emitted
3. Ensure user IDs match in both frontend and backend
4. Check server-side userSockets map

---

## 📈 Monitoring

### What to Monitor in Production

1. **Socket Connections:**

   - Total connected users
   - Connection/disconnect rate
   - Reconnection attempts

2. **Notification Delivery:**

   - Notifications sent
   - Notifications delivered
   - Notification failures

3. **Transaction Performance:**
   - Transaction commit rate
   - Transaction rollback rate
   - Average transaction time

---

## Next Steps (Optional Enhancements)

1. **Persist Notifications:**

   - Store notifications in database
   - Show notification history

2. **Notification Preferences:**

   - Let users opt-out of certain notifications
   - Email notifications as fallback

3. **Batch Operations:**

   - Hire multiple freelancers in one transaction

4. **Analytics:**

   - Track hire success rate
   - Monitor bid acceptance time

5. **Redis Adapter:**
   - For horizontal scaling

---

## Verification Checklist

Use this to verify everything is working:

- [ ] Backend starts without errors
- [ ] Frontend starts without errors
- [ ] Can login successfully
- [ ] Can post a gig
- [ ] Can submit a bid
- [ ] Can see "Hire" button when logged as gig owner
- [ ] Socket shows `✓ Connected` badge
- [ ] Browser console shows socket connection messages
- [ ] Notification appears when hiring someone
- [ ] Toast shows correct gig title and budget
- [ ] Notification auto-dismisses after 5 seconds
- [ ] Can close notification with X button
- [ ] Bid status updates to "hired" in database
- [ ] Other bids change to "rejected"
- [ ] Gig status changes to "assigned"
- [ ] Race condition test: second hire fails
- [ ] Rejected freelancers receive rejection notifications

---

**All tests pass? You're ready to use the feature!**
