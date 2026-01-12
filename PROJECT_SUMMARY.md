# GigFlow - Project Summary

Freelance platform: post jobs, submit bids, hire freelancers.

## Architecture

**Backend:** Node.js • Express • MongoDB • Mongoose • JWT
**Frontend:** React 18 • Vite • Redux • Tailwind • Router

---

## Backend Structure

```
src/
├── config/db.js          Database setup
├── models/               Data schemas
│   ├── User.js
│   ├── Gig.js
│   └── Bid.js
├── controllers/          Business logic
│   ├── authController.js
│   ├── gigController.js
│   └── bidController.js
├── routes/               API endpoints
│   ├── authRoutes.js
│   ├── gigRoutes.js
│   └── bidRoutes.js
├── middleware/           Auth & errors
│   ├── auth.js
│   └── errorHandler.js
└── server.js             Express app
```

### Backend Features

- JWT auth with HttpOnly cookies
- Password hashing (bcryptjs)
- MongoDB transactions for atomic operations
- Input validation
- CORS protection
- Protected routes
- Search functionality

---

## Frontend Structure

```
src/
├── pages/                Route pages
│   ├── BrowseGigs.jsx    List & search jobs
│   ├── GigDetails.jsx    View details, bids, hiring
│   ├── PostGig.jsx       Create job
│   ├── Login.jsx
│   └── Register.jsx
├── components/           Reusable UI
│   ├── Navbar.jsx
│   └── ProtectedRoute.jsx
├── store/                Redux state
│   ├── authSlice.js
│   ├── gigsSlice.js
│   └── bidsSlice.js
├── services/
│   └── api.js            API client
└── App.jsx               Main component
```

### Frontend Features

- User auth (register, login, logout)
- Browse jobs with search
- Post jobs (auth required)
- View job details
- Submit bids
- View bids (owner only)
- Hire freelancers (owner only)
- Protected routes
- Responsive design
- Redux state management

---

## API Endpoints

| Endpoint              | Method | Auth | Purpose           |
| --------------------- | ------ | ---- | ----------------- |
| /api/auth/register    | POST   | No   | Create account    |
| /api/auth/login       | POST   | No   | Login             |
| /api/auth/logout      | POST   | Yes  | Logout            |
| /api/auth/me          | GET    | Yes  | Current user      |
| /api/gigs             | GET    | No   | List jobs         |
| /api/gigs             | POST   | Yes  | Create job        |
| /api/gigs/:id         | GET    | No   | Job details       |
| /api/bids             | POST   | Yes  | Submit bid        |
| /api/bids/:gigId      | GET    | Yes  | View bids (owner) |
| /api/bids/:bidId/hire | PATCH  | Yes  | Hire (owner)      |

---

## Authentication

1. User registers with email/password
2. Password hashed with bcryptjs
3. Server creates JWT token (30 days)
4. Token stored in HttpOnly cookie
5. Middleware verifies token on protected routes
6. Token cleared on logout

---

## Database

### Users

```javascript
{
  _id, name, email, password(hashed), createdAt;
}
```

### Gigs

```javascript
{ _id, title, description, budget, ownerId, status: "open"|"assigned", createdAt }
```

### Bids

```javascript
{ _id, gigId, freelancerId, message, status: "pending"|"hired"|"rejected", createdAt }
```

---

## User Workflows

**Post Job:**

1. Login
2. Click "Post Gig"
3. Fill title, description, budget
4. Click "Post"
5. Job appears for others

**Submit Bid:**

1. Login
2. Browse jobs
3. Click job details
4. Enter bid message
5. Click "Submit Bid"

**Hire Freelancer:**

1. Login as job owner
2. View your posted job
3. See bids from freelancers
4. Click "Hire" on selected bid
5. Job status changes to "assigned"

---

## Tech Stack

**Backend:** Node.js, Express, MongoDB, Mongoose, JWT, bcryptjs, dotenv, cors
**Frontend:** React, Vite, Redux Toolkit, Tailwind CSS, React Router, Axios
**Deployment:** Heroku/Railway (backend), Vercel/Netlify (frontend)

---

## Security

- Passwords hashed with bcryptjs
- JWT in HttpOnly cookies (not accessible via JS)
- CORS configured to allow frontend only
- Input validation on all endpoints
- Protected routes require authentication
- Authorization checks (owner only features)
- MongoDB injection prevention

---

## Files

**Documentation:**

- README.md - Project overview
- QUICK_START.md - 5-minute setup
- SETUP_GUIDE.md - Detailed setup
- PROJECT_SUMMARY.md - This file (architecture)

**Backend:** gigflow-backend/
**Frontend:** gigflow-frontend/

### Gig Posting

- Authenticated users can create gigs
- Gig starts with "open" status
- Search functionality by title

### Bidding

- Freelancers can bid on open gigs
- Only one bid per freelancer per gig
- Bids stored with message and "pending" status

### Hiring

- Only gig owner can view bids
- Hiring uses MongoDB transactions for atomicity
- When freelancer hired:
  - Selected bid → "hired"
  - Other bids → "rejected"
  - Gig status → "assigned"
- Prevents race conditions with transaction support

---

## Technology Choices

### Backend

| Technology     | Reason                                           |
| -------------- | ------------------------------------------------ |
| Node.js        | Lightweight, event-driven, perfect for APIs      |
| Express.js     | Minimalist, production-ready web framework       |
| MongoDB        | Flexible schema, great for freelance data        |
| Mongoose       | Schema validation and ODM features               |
| JWT + HttpOnly | Secure token storage without XSS vulnerabilities |
| bcryptjs       | Battle-tested password hashing                   |

### Frontend

| Technology    | Reason                                    |
| ------------- | ----------------------------------------- |
| React         | Component-based, large ecosystem          |
| Vite          | Fast build tool, instant HMR              |
| Tailwind CSS  | Utility-first, highly customizable        |
| Redux Toolkit | Simplified state management               |
| React Router  | Client-side routing with protected routes |
| Axios         | Promise-based HTTP client                 |

---

## 📦 Dependencies

### Backend

```json
{
  "express": "^4.18.2",
  "mongoose": "^7.5.0",
  "bcryptjs": "^2.4.3",
  "jsonwebtoken": "^9.0.2",
  "dotenv": "^16.3.1",
  "cors": "^2.8.5",
  "cookie-parser": "^1.4.6"
}
```

### Frontend

```json
{
  "react": "^18.2.0",
  "react-dom": "^18.2.0",
  "react-router-dom": "^6.14.2",
  "@reduxjs/toolkit": "^1.9.5",
  "react-redux": "^8.1.3",
  "axios": "^1.4.0"
}
```

---

## Quick Start Commands

### Backend

```bash
cd gigflow-backend
cp .env.example .env
npm install
npm run dev  # Development mode
npm start    # Production mode
```

### Frontend

```bash
cd gigflow-frontend
npm install
npm run dev    # Development mode
npm run build  # Production build
```

---

## Compliance with Requirements

### Assignment Requirements Met

#### Backend ✓

- Node.js + Express.js
- MongoDB with Mongoose
- JWT in HttpOnly cookies
- Models: User, Gig, Bid
- Auth APIs: register, login
- Gig APIs: GET (with search), POST
- Bid APIs: POST, GET (owner only)
- Hiring API: PATCH with atomic transactions
- .env.example provided
- Clear folder structure
- Middleware for auth & authorization

#### Frontend ✓

- React.js (Vite)
- Tailwind CSS
- Redux Toolkit (state management)
- Auth pages (Register, Login)
- Browse Gigs page (with search)
- Post Gig page (protected)
- Gig Details page (all features)
- Auth state management
- Gigs state management
- Bids state management
- API integration
- Fluid role behavior (user can post and bid)

#### General ✓

- Two separate repositories
- No bonus features added
- Follows exact tech stack
- Clean, minimal, production-ready code
- Complete READMEs with setup instructions
- Runnable code ready to deploy

---

## Code Quality

### Backend Code

- Clean MVC architecture (Models, Views via API, Controllers)
- Error handling throughout
- Input validation on all endpoints
- Proper HTTP status codes
- Consistent naming conventions
- Modular route structure

### Frontend Code

- Component-based architecture
- Redux state management
- Protected route component
- Error handling and user feedback
- Responsive design
- Consistent UI with Tailwind
- Reusable components

---

## 🔒 Security Implementation

Password hashing with bcryptjs
JWT tokens in HttpOnly cookies
CORS properly configured
Input validation on all forms
Protected API endpoints
Authorization checks (only gig owner can view bids/hire)
MongoDB injection prevention via Mongoose
XSS protection via React
Secure token expiration (30 days)

---

## Database Schema

### Users Collection

```javascript
{
  _id: ObjectId,
  name: String,
  email: String (unique),
  password: String (hashed),
  createdAt: Date,
  updatedAt: Date
}
```

### Gigs Collection

```javascript
{
  _id: ObjectId,
  title: String,
  description: String,
  budget: Number,
  ownerId: ObjectId (ref: User),
  status: "open" | "assigned",
  createdAt: Date,
  updatedAt: Date
}
```

### Bids Collection

```javascript
{
  _id: ObjectId,
  gigId: ObjectId (ref: Gig),
  freelancerId: ObjectId (ref: User),
  message: String,
  status: "pending" | "hired" | "rejected",
  createdAt: Date,
  updatedAt: Date
}
```

---

## User Flows

### Job Owner Flow

1. Register/Login
2. Post a gig
3. View bids on their gigs
4. Hire a freelancer
5. Gig automatically marked as assigned

### Freelancer Flow

1. Register/Login
2. Browse available gigs
3. Search for relevant gigs
4. View gig details
5. Submit bid with message
6. Wait for job owner response

### Fluid Role Flow

- Same user can be both job owner and freelancer
- Can post gigs AND bid on others' gigs
- No role restrictions

---

## Testing Checklist

- User Registration
- User Login
- Browse Gigs
- Search Gigs by Title
- View Gig Details
- Post a New Gig
- Submit Bid
- View Bids (Owner)
- Hire Freelancer
- Gig Status Change
- Bid Status Updates
- Protected Routes
- Error Handling
- CORS Functionality
- JWT Authentication

---

## Deployment Ready

Both repositories are production-ready for deployment to:

**Backend:**

- Heroku, Railway, Render, Fly.io, AWS EC2
- Any Node.js hosting platform

**Frontend:**

- Vercel, Netlify, AWS S3 + CloudFront
- GitHub Pages, Azure Static Web Apps
- Any static hosting platform

---

## Support Resources

Each repository contains comprehensive README files with:

- Installation instructions
- Setup commands
- API reference
- Feature documentation
- Troubleshooting guide
- Project structure
- Deployment guide

---

## Project Complete

The GigFlow application is fully implemented and ready for:

- Development testing
- Production deployment
- Further customization
- Scaling and enhancement

All code is clean, well-documented, and follows industry best practices.

**Created:** January 11, 2026
**Status:** Complete & Production-Ready
