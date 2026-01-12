# GigFlow - Setup Guide

## Prerequisites

- Node.js v14+ ([Download](https://nodejs.org/))
- MongoDB (local or [MongoDB Atlas](https://www.mongodb.com/cloud/atlas))

Verify:

```bash
node --version
npm --version
```

## Backend Setup

```bash
cd gigflow-backend
npm install
cp .env.example .env
```

Edit `.env`:

```
PORT=5000
MONGODB_URI=mongodb://localhost:27017/gigflow
JWT_SECRET=your_secret_key_here
NODE_ENV=development
FRONTEND_URL=http://localhost
```

Start MongoDB (if local):

```bash
# macOS
brew services start mongodb-community

# Windows - auto-starts as service

# Linux
sudo systemctl start mongod
```

Start backend:

```bash
npm run dev     # Development
# or
npm start       # Production
```

Expected: `Server is running successfully`

## Frontend Setup

Open new terminal:

```bash
cd gigflow-frontend
npm install
npm run dev
```

Expected: `Frontend is running`

## Test the App

1. **Register** - Click "Register" and create account
2. **Post Gig** - Click "Post Gig", fill details, submit
3. **Create Another User** - Logout/incognito, register new account
4. **Submit Bid** - Find gig, click details, submit bid
5. **Hire** - Login as first user, view bid, click "Hire"
6. **Verify** - Status changes to "assigned"

## Deploy

### Backend

1. Get production MongoDB URI (MongoDB Atlas)
2. Update `.env` with production values
3. Deploy to: Heroku, Railway, Render, or AWS

```bash
npm start
```

### Frontend

```bash
npm run build    # Creates dist/ folder
```

Deploy `dist/` to: Vercel, Netlify, or AWS S3

Update backend URL in production.

## Troubleshooting

| Problem                  | Solution                                      |
| ------------------------ | --------------------------------------------- |
| MongoDB connection fails | Ensure MongoDB is running, verify URI in .env |
| CORS errors              | Verify both services are running              |
| Port already in use      | Kill process or use different port            |

---

## API Reference

| Endpoint              | Method | Auth | Purpose                |
| --------------------- | ------ | ---- | ---------------------- |
| /api/auth/register    | POST   | No   | Create account         |
| /api/auth/login       | POST   | No   | Login                  |
| /api/auth/logout      | POST   | Yes  | Logout                 |
| /api/auth/me          | GET    | Yes  | Current user           |
| /api/gigs             | GET    | No   | List jobs (searchable) |
| /api/gigs             | POST   | Yes  | Create job             |
| /api/gigs/:id         | GET    | No   | Job details            |
| /api/bids             | POST   | Yes  | Submit bid             |
| /api/bids/:gigId      | GET    | Yes  | View bids (owner)      |
| /api/bids/:bidId/hire | PATCH  | Yes  | Hire (owner)           |
