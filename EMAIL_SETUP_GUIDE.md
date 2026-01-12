# Email Notifications Setup Guide

## Overview

Email notifications have been integrated into GigFlow. When a freelancer is hired or rejected, they receive professional HTML emails with all relevant details.

---

## Email Types

### 1. **Hire Notification Email**

**Sent to:** Freelancer (when hired)
**Includes:**

- Congratulations message
- Project title
- Budget amount
- Client name
- Link to project details
- Call-to-action button

### 2. **Rejection Notification Email**

**Sent to:** Other bidders (when not selected)
**Includes:**

- Professional rejection message
- Project title
- Tips for future bids
- Encouragement to browse more gigs
- Call-to-action button

### 3. **Hire Confirmation Email**

**Sent to:** Client (when freelancer hired)
**Includes:**

- Confirmation message
- Project title
- Hired freelancer name
- Budget amount
- Next steps

---

## Configuration

### Step 1: Get Email Credentials

#### Option A: Using Gmail (Recommended for Testing)

1. Go to [Google Account Security Settings](https://myaccount.google.com/security)
2. Enable "2-Step Verification" if not already enabled
3. Go to "App passwords" section
4. Select "Mail" and "Windows Computer" (or your device)
5. Google will generate a 16-character password
6. Copy this password (you'll use it as EMAIL_PASSWORD)

#### Option B: Using Custom SMTP Server

If you have a custom SMTP server:

- Host: your-smtp-host.com
- TLS Port for secure connections
- Username: your-email@your-domain.com
- Password: your-smtp-password

---

### Step 2: Update .env File

Create or update `gigflow-backend/.env` with:

```env
# Email Configuration (Gmail)
EMAIL_USER=your-gmail@gmail.com
EMAIL_PASSWORD=your-16-char-app-password

# Or for custom SMTP (uncomment if using custom)
# SMTP_HOST=smtp.your-server.com
SMTP_HOST=smtp.mailtrap.io
# SMTP_SECURE=false
# SMTP_USER=your-email@your-domain.com
# SMTP_PASSWORD=your-password

# Frontend URL (for email links)
FRONTEND_URL=http://localhost
```

---

### Step 3: Verify Email Configuration

The email service will verify the connection on startup. You should see in your console:

```
Email service is ready to send messages
```

If you see an error, check:

- Email credentials are correct
- Gmail has app-specific password generated (for Gmail)
- 2-Step verification is enabled (for Gmail)
- SMTP settings are correct (for custom servers)

---

## Email Service API

### `emailService.sendHireEmail(email, name, details)`

Sends hire notification email to freelancer.

**Parameters:**

```javascript
{
  email: "freelancer@example.com",
  name: "John Freelancer",
  details: {
    gigTitle: "Build iOS App",
    budget: 5000,
    clientName: "Jane Client",
    gigId: "507f1f77bcf86cd799439011"
  }
}
```

**Returns:**

```javascript
{
  success: true,
  messageId: "message-id-from-server"
}
```

---

### `emailService.sendRejectionEmail(email, name, details)`

Sends rejection notification to other bidders.

**Parameters:**

```javascript
{
  email: "freelancer@example.com",
  name: "John Freelancer",
  details: {
    gigTitle: "Build iOS App",
    gigId: "507f1f77bcf86cd799439011"
  }
}
```

---

### `emailService.sendHireConfirmationEmail(email, name, details)`

Sends confirmation email to client.

**Parameters:**

```javascript
{
  email: "client@example.com",
  name: "Jane Client",
  details: {
    gigTitle: "Build iOS App",
    freelancerName: "John Freelancer",
    budget: 5000
  }
}
```

---

## Implementation Details

### File Structure

```
src/
├── services/
│   ├── notificationService.js (Real-time notifications)
│   └── emailService.js        (Email notifications) ← NEW
└── controllers/
    └── bidController.js       (Updated to use email service)
```

### How It Works

**When a freelancer is hired:**

1. Transaction completes successfully
2. Real-time socket notification sent
3. Email to hired freelancer (asynchronously)
4. Email confirmation to client (asynchronously)
5. Rejection emails to other bidders (asynchronously)
6. Response sent to client

**Email sending is asynchronous**, so it doesn't block the API response.

---

## Testing Email Notifications

### Test Locally with Gmail

1. Update `.env` with your Gmail credentials
2. Start the server
3. Verify console shows: "Email service is ready to send messages"
4. Hire a freelancer through the UI
5. Check your email inbox (may take 30 seconds - 2 minutes)

### Use Test Email Service (Mailtrap/Mailhog)

For development without sending real emails:

**Using Mailhog (Free, Docker-based):**

```bash
# Install and run Mailhog
docker run -p 1025:1025 -p 8025:8025 mailhog/mailhog
```

Update `.env`:

```env
SMTP_HOST=localhost
SMTP_HOST=live.smtp.mailtrap.io
SMTP_SECURE=false
SMTP_USER=test@example.com
SMTP_PASSWORD=anything
```

View emails at: http://localhost:8025

**Using Mailtrap (Web-based):**

1. Sign up at [mailtrap.io](https://mailtrap.io)
2. Get SMTP credentials from Mailtrap dashboard
3. Update `.env` with Mailtrap credentials
4. All emails will appear in your Mailtrap inbox

---

## Email Templates

### Customization

The email templates are built into `emailService.js`. To customize:

1. Open `src/services/emailService.js`
2. Modify the `htmlContent` in the relevant method
3. Update styles, colors, or text
4. Test by hiring a freelancer

**Example: Change header color**

```javascript
// Find this line in htmlContent
.header { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }

// Change to your color
.header { background: linear-gradient(135deg, #ff6b6b 0%, #ee5a6f 100%); }
```

### Template Structure

Each email includes:

- **Header** - Branded section with title
- **Content** - Main information about the action
- **Call-to-Action** - Button to take action
- **Footer** - Copyright and support info

---

## Error Handling

### Email Sending Failures

If an email fails to send:

- Error is logged to console
- API response is still successful (doesn't block hiring)
- User can be notified through UI later

**Check console for errors:**

```
Error sending hire email to freelancer@example.com: [error message]
```

**Common issues:**

| Error                        | Solution                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| Invalid email credentials    | Check EMAIL_USER and EMAIL_PASSWORD in .env                  |
| Gmail app password incorrect | Generate new app password, remove spaces                     |
| SMTP connection refused      | Check SMTP_HOST is correct                       |
| TLS error                    | Verify your SMTP credentials                   |
| Rate limited                 | Some providers limit email frequency - wait before resending |

---

## Production Deployment

### Requirements

- **Email Provider:** Gmail, SendGrid, Mailgun, or custom SMTP
- **Environment Variables:** EMAIL_USER, EMAIL_PASSWORD configured
- **Frontend URL:** Set FRONTEND_URL for email links

### Best Practices

1. **Use Transactional Email Service**

   - Gmail: Good for development, limited for production
   - SendGrid: Recommended for production (free tier: 100 emails/day)
   - Mailgun: Good alternative (free tier: 5,000 emails/month)

2. **Monitor Email Delivery**

   - Track bounced emails
   - Monitor spam complaints
   - Log all email activities

3. **Set Up Email Templates**

   - Create branded templates
   - Test on multiple email clients
   - Use responsive design

4. **GDPR Compliance**
   - Add unsubscribe option
   - Handle email preferences
   - Log consent

---

## SendGrid Integration (Production Recommended)

### Setup SendGrid

1. Sign up at [sendgrid.com](https://sendgrid.com)
2. Create API key
3. Update code in `emailService.js`:

```javascript
// Replace the transporter initialization with:
this.transporter = nodemailer.createTransport({
  host: "smtp.sendgrid.net",
SMTP_HOST: 'live.smtp.mailtrap.io',
  auth: {
    user: "apikey",
    pass: process.env.SENDGRID_API_KEY,
  },
});
```

4. Update `.env`:

```env
SENDGRID_API_KEY=your-sendgrid-api-key
```

---

## Troubleshooting

### Emails not sending

**Step 1:** Verify email service initialized

```javascript
// In browser console after starting server
// Should see: "Email service is ready to send messages"
```

**Step 2:** Check .env variables

```bash
# Verify these are set and not empty
echo $EMAIL_USER
echo $EMAIL_PASSWORD
```

**Step 3:** Enable Gmail Less Secure Access

- Gmail app passwords require 2-Step verification
- Regenerate app password if older than 30 days

**Step 4:** Check email provider logs

- Gmail: Check "Less secure app access" activity
- SendGrid: Check activity log for bounce/blocked emails
- Custom SMTP: Check server logs

### Emails going to spam

**Solutions:**

1. Add proper sender verification (DKIM, SPF)
2. Use branded domain email (not Gmail @gmail.com)
3. Add unsubscribe link
4. Use consistent sender name
5. Reduce email frequency

---

## Files Modified

- `src/services/emailService.js` - NEW (Email sending logic)
- `src/controllers/bidController.js` - MODIFIED (Added email notifications)
- `package.json` - Already has nodemailer

---

## Next Steps

1. **Setup:** Choose email provider and get credentials
2. **Configure:** Add EMAIL_USER and EMAIL_PASSWORD to .env
3. **Test:** Hire a freelancer and check your email
4. **Monitor:** Check console for any email errors
5. **Customize:** Edit templates if needed
6. **Scale:** Switch to SendGrid for production

---

## Support

For issues:

1. Check console for error messages
2. Verify email credentials in .env
3. Test email provider directly
4. Check email provider spam folder
5. Review troubleshooting section above

---

**Email notifications are now live!**
