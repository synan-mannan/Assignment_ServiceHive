# GigFlow - Complete File Listing

## Directory Structure

```
Interview_assignment/
├── SETUP_GUIDE.md
├── PROJECT_SUMMARY.md
├── FILE_LISTING.md (this file)
│
├── gigflow-backend/
│   ├── package.json
│   ├── .env.example
│   ├── .gitignore
│   ├── README.md
│   │
│   └── src/
│       ├── server.js
│       │
│       ├── config/
│       │   └── db.js
│       │
│       ├── models/
│       │   ├── User.js
│       │   ├── Gig.js
│       │   └── Bid.js
│       │
│       ├── controllers/
│       │   ├── authController.js
│       │   ├── gigController.js
│       │   └── bidController.js
│       │
│       ├── routes/
│       │   ├── authRoutes.js
│       │   ├── gigRoutes.js
│       │   └── bidRoutes.js
│       │
│       └── middleware/
│           ├── auth.js
│           └── errorHandler.js
│
└── gigflow-frontend/
    ├── package.json
    ├── vite.config.js
    ├── tailwind.config.js
    ├── postcss.config.js
    ├── index.html
    ├── .gitignore
    ├── README.md
    │
    └── src/
        ├── main.jsx
        ├── App.jsx
        ├── index.css
        │
        ├── components/
        │   ├── Navbar.jsx
        │   └── ProtectedRoute.jsx
        │
        ├── pages/
        │   ├── Register.jsx
        │   ├── Login.jsx
        │   ├── BrowseGigs.jsx
        │   ├── PostGig.jsx
        │   └── GigDetails.jsx
        │
        ├── store/
        │   ├── index.js
        │   ├── authSlice.js
        │   ├── gigsSlice.js
        │   └── bidsSlice.js
        │
        └── services/
            └── api.js
```

## Complete File Count

**Total Files: 39**

### Root Directory (3 files)

- SETUP_GUIDE.md
- PROJECT_SUMMARY.md
- FILE_LISTING.md

### Backend Files (17 files)

**Root Level:**

1. gigflow-backend/package.json
2. gigflow-backend/.env.example
3. gigflow-backend/.gitignore
4. gigflow-backend/README.md

**src/ Directory:** 5. gigflow-backend/src/server.js

**src/config/:** 6. gigflow-backend/src/config/db.js

**src/models/:** 7. gigflow-backend/src/models/User.js 8. gigflow-backend/src/models/Gig.js 9. gigflow-backend/src/models/Bid.js

**src/controllers/:** 10. gigflow-backend/src/controllers/authController.js 11. gigflow-backend/src/controllers/gigController.js 12. gigflow-backend/src/controllers/bidController.js

**src/routes/:** 13. gigflow-backend/src/routes/authRoutes.js 14. gigflow-backend/src/routes/gigRoutes.js 15. gigflow-backend/src/routes/bidRoutes.js

**src/middleware/:** 16. gigflow-backend/src/middleware/auth.js 17. gigflow-backend/src/middleware/errorHandler.js

### Frontend Files (19 files)

**Root Level:**

1. gigflow-frontend/package.json
2. gigflow-frontend/vite.config.js
3. gigflow-frontend/tailwind.config.js
4. gigflow-frontend/postcss.config.js
5. gigflow-frontend/index.html
6. gigflow-frontend/.gitignore
7. gigflow-frontend/README.md

**src/ Directory:** 8. gigflow-frontend/src/main.jsx 9. gigflow-frontend/src/App.jsx 10. gigflow-frontend/src/index.css

**src/components/:** 11. gigflow-frontend/src/components/Navbar.jsx 12. gigflow-frontend/src/components/ProtectedRoute.jsx

**src/pages/:** 13. gigflow-frontend/src/pages/Register.jsx 14. gigflow-frontend/src/pages/Login.jsx 15. gigflow-frontend/src/pages/BrowseGigs.jsx 16. gigflow-frontend/src/pages/PostGig.jsx 17. gigflow-frontend/src/pages/GigDetails.jsx

**src/store/:** 18. gigflow-frontend/src/store/index.js 19. gigflow-frontend/src/store/authSlice.js 20. gigflow-frontend/src/store/gigsSlice.js 21. gigflow-frontend/src/store/bidsSlice.js

**src/services/:** 22. gigflow-frontend/src/services/api.js

---

## File Descriptions

### Documentation Files

#### SETUP_GUIDE.md

Comprehensive guide covering:

- Project overview
- Prerequisites
- Backend setup instructions
- Frontend setup instructions
- Testing procedures
- Common issues and solutions
- Production deployment
- Project structure reference

#### PROJECT_SUMMARY.md

Complete project documentation including:

- Project completion overview
- File structure and organization
- Architecture details
- API endpoints reference
- Key features implemented
- Technology choices
- Dependencies listing
- Code quality assessment
- Security implementation
- Database schema
- Testing checklist

#### FILE_LISTING.md (this file)

Directory structure and file descriptions

---

## Backend Files Description

### Configuration

**server.js** - Main Express application setup

- CORS configuration
- Middleware setup (JSON, URL-encoded, cookies)
- Route registration
- Error handler middleware
- Server initialization

**.env.example** - Environment template

- PORT, MONGODB_URI, JWT_SECRET, NODE_ENV, FRONTEND_URL

**package.json** - Dependencies and scripts

- express, mongoose, bcryptjs, jsonwebtoken, dotenv, cors, cookie-parser
- nodemon for development

**.gitignore** - Git ignore rules

**README.md** - Backend documentation and API reference

### Database Configuration

**config/db.js** - MongoDB connection

- Mongoose connection with error handling
- Connection logging

### Data Models

**models/User.js**

- name, email, password fields
- Password hashing middleware
- Password comparison method

**models/Gig.js**

- title, description, budget, ownerId, status fields
- Status enum: ['open', 'assigned']

**models/Bid.js**

- gigId, freelancerId, message, status fields
- Status enum: ['pending', 'hired', 'rejected']

### Business Logic

**controllers/authController.js**

- register() - Create new user account
- login() - Authenticate user and issue JWT
- getMe() - Get current authenticated user
- logout() - Clear authentication cookie

**controllers/gigController.js**

- getGigs() - Fetch all open gigs with optional title search
- createGig() - Create new gig (authenticated)
- getGig() - Get specific gig by ID

**controllers/bidController.js**

- submitBid() - Submit bid for a gig
- getBidsForGig() - Get all bids for a gig (owner only)
- hireBid() - Hire freelancer with atomic transaction

### API Routes

**routes/authRoutes.js**

- POST /api/auth/register
- POST /api/auth/login
- GET /api/auth/me (protected)
- POST /api/auth/logout (protected)

**routes/gigRoutes.js**

- GET /api/gigs (public, searchable)
- GET /api/gigs/:id (public)
- POST /api/gigs (protected)

**routes/bidRoutes.js**

- POST /api/bids (protected)
- GET /api/bids/:gigId (protected, owner only)
- PATCH /api/bids/:bidId/hire (protected, owner only)

### Middleware

**middleware/auth.js**

- protect() - JWT verification from HttpOnly cookie
- authorize() - Role-based authorization

**middleware/errorHandler.js**

- Global error handling
- MongoDB error transformation
- Consistent error responses

---

## Frontend Files Description

### Configuration

**package.json** - Dependencies and build scripts

- React, React-DOM, React Router
- Redux Toolkit, React-Redux
- Axios, Tailwind CSS
- Vite, PostCSS, Autoprefixer

**vite.config.js** - Vite configuration

- React plugin
- Port 5173
- API proxy to backend

**tailwind.config.js** - Tailwind CSS configuration

**postcss.config.js** - PostCSS with Tailwind

**index.html** - Main HTML entry point

**.gitignore** - Git ignore rules

**README.md** - Frontend documentation

### Application Entry

**main.jsx** - React entry point

- Redux Provider wrapper
- React.StrictMode
- App component render

**App.jsx** - Main component with routing

- BrowserRouter setup
- Route definitions
- Route protection

### Styling

**index.css** - Tailwind CSS directives

### Components

**components/Navbar.jsx**

- Navigation bar
- Login/Register/Logout buttons
- Links to main pages
- User name display
- Responsive design

**components/ProtectedRoute.jsx**

- Route protection wrapper
- Authentication check
- Redirect to login if needed

### Pages

**pages/Register.jsx**

- Registration form
- Name, email, password fields
- Password confirmation
- Redux auth dispatch
- Error handling
- Redirect on success

**pages/Login.jsx**

- Login form
- Email, password fields
- Redux auth dispatch
- Error handling
- Redirect on success

**pages/BrowseGigs.jsx**

- Display all open gigs
- Search by title functionality
- Gig cards with details
- Link to gig details
- Loading and error states
- Redux gigs dispatch

**pages/PostGig.jsx**

- Create new gig form
- Title, description, budget fields
- Protected route component
- Form validation
- Redux gigs dispatch
- Success redirect

**pages/GigDetails.jsx**

- Full gig information display
- Bid submission form (non-owners)
- Bids list (owners only)
- Hire functionality (owners)
- Atomic status updates
- Loading and error states
- Redux dispatch for multiple slices

### State Management

**store/index.js**

- Redux store configuration
- Combine reducers from all slices

**store/authSlice.js**

- user, token, isAuthenticated state
- Auth actions: login, register, logout
- Error and loading states

**store/gigsSlice.js**

- gigs array, currentGig, loading state
- Gig actions: fetch, create
- Error handling

**store/bidsSlice.js**

- bids array, loading state
- Bid actions: fetch, submit, hire
- Error handling

### API Services

**services/api.js**

- Axios instance with base URL
- withCredentials for cookies
- authAPI object with register, login, logout, getMe
- gigsAPI object with getGigs, getGig, createGig
- bidsAPI object with submitBid, getBidsForGig, hireBid

---

## File Statistics

### Lines of Code (Approximate)

**Backend:**

- server.js: ~40 lines
- Models: ~120 lines
- Controllers: ~300 lines
- Routes: ~35 lines
- Middleware: ~50 lines
- Config: ~15 lines
- **Total: ~560 lines**

**Frontend:**

- Components: ~150 lines
- Pages: ~850 lines
- Store: ~350 lines
- Services: ~80 lines
- App/Main: ~50 lines
- Styles: ~15 lines
- **Total: ~1,495 lines**

**Total Project: ~2,055 lines**

---

## Key Implementation Details

### Backend Highlights

- Atomic MongoDB transactions for hiring
- Password hashing with proper salt rounds
- JWT tokens in HttpOnly cookies
- CORS configured for frontend
- Comprehensive error handling
- Input validation on all endpoints
- Protected routes with middleware

### Frontend Highlights

- Redux state management
- Protected routes component
- Axios API client with error handling
- Form validation
- Loading and error states
- Responsive Tailwind design
- localStorage for token persistence

---

## How to Use This Project

1. **Clone/Copy both repositories**
2. **Follow SETUP_GUIDE.md** for installation
3. **Read individual READMEs** in each repo
4. **Refer to PROJECT_SUMMARY.md** for architecture overview
5. **Use this FILE_LISTING.md** to understand structure

---

## Deployment Checklist

- [ ] Copy and customize `.env` files
- [ ] Install all dependencies (`npm install`)
- [ ] Set up MongoDB (local or Atlas)
- [ ] Start backend server
- [ ] Start frontend dev server
- [ ] Test complete workflow
- [ ] Build for production
- [ ] Deploy to hosting platform
- [ ] Update environment variables
- [ ] Verify all endpoints work

---

## Next Steps

1. Start with SETUP_GUIDE.md
2. Set up backend first
3. Set up frontend
4. Test all features
5. Deploy when ready

Happy coding!
