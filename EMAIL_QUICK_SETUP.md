# Email Notifications Quick Setup

## 60-Second Setup

### Step 1: Gmail Configuration (1 minute)

1. Go to [Google Account Security](https://myaccount.google.com/security)
2. Find "App passwords" (requires 2-Step Verification)
3. Select "Mail" → "Windows Computer"
4. Copy the 16-character password

### Step 2: Update .env (30 seconds)

**File:** `gigflow-backend/.env`

```env
# Add these lines:
EMAIL_USER=your-email@gmail.com
EMAIL_PASSWORD=xxxx xxxx xxxx xxxx
FRONTEND_URL=http://localhost:5173
```

**Note:** Remove spaces from the password if copying

### Step 3: Start Server (30 seconds)

```bash
cd gigflow-backend
npm run dev
```

Look for this in console:

```
✓ Email service is ready to send messages
```

---

## Test It

1. Login as Client → Hire a Freelancer
2. Check your email (wait 30-60 seconds)
3. ✅ Email should arrive!

---

## What Gets Emailed

- **Freelancer:** "You've been hired!" email with project details
- **Client:** "Freelancer hired" confirmation email
- **Rejected bidders:** "Your bid wasn't selected" email

---

## Email Preview

### Hire Notification (for freelancer)

```
Subject: Congratulations! You've been hired for "Project Title"

- Project Title
- Budget: $5,000
- Client: John Developer
- [View Project Details] button
```

### Rejection Email (for other bidders)

```
Subject: Update on your bid for "Project Title"

- Professional rejection message
- Tips for future bids
- [Browse More Gigs] button
```

---

## Alternative: Mailtrap (Free Testing)

Don't want to use real Gmail?

1. Sign up: [mailtrap.io](https://mailtrap.io) (free)
2. Get SMTP credentials
3. Update `.env`:

```env
SMTP_HOST=live.smtp.mailtrap.io
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your-mailtrap-user
SMTP_PASSWORD=your-mailtrap-password
```

4. View all test emails: http://localhost:8025

---

## Troubleshooting

| Issue                 | Solution                                                |
| --------------------- | ------------------------------------------------------- |
| "Email service error" | Check EMAIL_USER and EMAIL_PASSWORD in .env             |
| No email arrives      | Verify email provider allows "less secure apps" or 2FA  |
| Email goes to spam    | Use SendGrid instead (see EMAIL_SETUP_GUIDE.md)         |
| Still not working?    | Check EMAIL_SETUP_GUIDE.md for detailed troubleshooting |

---

## That's It! ✅

Your GigFlow now sends emails when:

- ✅ Someone gets hired
- ✅ Someone gets rejected
- ✅ Client confirms hire

For more details, see: [EMAIL_SETUP_GUIDE.md](EMAIL_SETUP_GUIDE.md)
