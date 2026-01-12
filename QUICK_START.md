# GigFlow - Quick Start

## Prerequisites

- Node.js v14+
- MongoDB (local or [MongoDB Atlas](https://www.mongodb.com/cloud/atlas))

## Setup (2 Terminals)

**Terminal 1 - Backend:**

```bash
cd gigflow-backend
cp .env.example .env
npm install
npm run dev
```

Expected: `Server running successfully`

**Terminal 2 - Frontend:**

```bash
cd gigflow-frontend
npm install
npm run dev
```

Expected: `Frontend running successfully`

## Test It

1. Open the frontend app
2. Register → Create account
3. Post Gig → Add a job
4. Logout → Switch user
5. Register → Create second account
6. Browse Gigs → Find the job
7. Submit Bid → Apply
8. Login → As first user
9. Hire → Select freelancer

Done!

---

## Commands

```bash
# Backend
npm install       # First time only
npm run dev       # Start with auto-reload
npm start         # Production mode

# Frontend
npm install       # First time only
npm run dev       # Start dev server
npm run build     # Build for production
npm run preview   # Preview build
```

---

## Troubleshooting

| Problem               | Solution                                  |
| --------------------- | ----------------------------------------- |
| MongoDB won't connect | Ensure MongoDB is running, check .env URI |
| CORS errors           | Backend on 5000, frontend on 5173         |
| Port in use           | Check the terminal for available port     |
| Login not working     | Clear browser cookies/localStorage        |
| Can't see bids        | Login as job owner (different account)    |

---

## API Endpoints

| Method | Endpoint              | Purpose           |
| ------ | --------------------- | ----------------- |
| POST   | /api/auth/register    | Create account    |
| POST   | /api/auth/login       | Login             |
| POST   | /api/auth/logout      | Logout            |
| GET    | /api/auth/me          | Current user      |
| GET    | /api/gigs             | All jobs          |
| POST   | /api/gigs             | Create job        |
| GET    | /api/gigs/:id         | Job details       |
| POST   | /api/bids             | Submit bid        |
| GET    | /api/bids/:gigId      | View bids (owner) |
| PATCH  | /api/bids/:bidId/hire | Hire (owner)      |

---

## Deployment

**Backend:** Heroku, Railway, Render, AWS
**Frontend:** Vercel, Netlify, S3

See [SETUP_GUIDE.md](./SETUP_GUIDE.md) for details
