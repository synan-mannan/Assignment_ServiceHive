# GigFlow - Freelance Platform

## Quick Start

**Start here:** [QUICK_START.md](./QUICK_START.md) (5 minutes)

For more: [SETUP_GUIDE.md](./SETUP_GUIDE.md) | [PROJECT_SUMMARY.md](./PROJECT_SUMMARY.md)

---

## Project Structure

```
Interview_assignment/
├── gigflow-backend/          Node.js API (port 5000)
│   ├── src/
│   │   ├── models/          Data schemas
│   │   ├── controllers/     Business logic
│   │   ├── routes/          API endpoints
│   │   └── middleware/      Auth & errors
│   └── package.json
│
└── gigflow-frontend/         React app (port 5173)
    ├── src/
    │   ├── pages/           Route pages
    │   ├── components/      UI components
    │   ├── store/           Redux state
    │   └── services/        API client
    └── package.json
```

---

## Setup (2 Terminals)

**Terminal 1 - Backend:**

```bash
cd gigflow-backend
npm install
npm run dev
```

**Terminal 2 - Frontend:**

```bash
cd gigflow-frontend
npm install
npm run dev
```

Then open [http://localhost:5173](http://localhost:5173)

---

## Features

- Register & login users
- Post jobs and browse gigs
- Search jobs by title
- Submit bids on jobs
- Hire freelancers
- View bids (job owners only)
- Responsive design
- Secure authentication

---

## Tech Stack

**Backend:** Node.js, Express, MongoDB, JWT, bcryptjs
**Frontend:** React, Vite, Redux, Tailwind CSS, Axios

---

## API Endpoints

```
POST   /api/auth/register       Create account
POST   /api/auth/login          Login
POST   /api/auth/logout         Logout
GET    /api/auth/me             Current user

GET    /api/gigs                Browse jobs
POST   /api/gigs                Create job
GET    /api/gigs/:id            View job details

POST   /api/bids                Submit bid
GET    /api/bids/:gigId         View bids (owner only)
PATCH  /api/bids/:bidId/hire    Hire freelancer (owner only)
```
