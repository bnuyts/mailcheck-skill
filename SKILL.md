# MailCheck Email Verification API Skill

## Overview
This skill provides comprehensive email verification capabilities through the MailCheck API. It enables agents to verify single emails, process bulk verifications, analyze email authenticity, and manage API keys securely.

## API Documentation
- **Base URL**: `https://api.mailcheck.dev/v1`
- **Authentication**: Bearer token (`sk_live_your_key`)
- **Rate Limits**: Varies by plan (Free: 10/min, Starter: 60/min, Pro: 120/min)

## Endpoints

### 1. Single Email Verification
**POST** `/verify`

Verifies a single email address through multiple validation steps.

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Response:**
```json
{
  "email": "user@example.com",
  "valid": true,
  "score": 90,
  "reason": "deliverable",
  "checks": {
    "syntax": "pass",
    "disposable": "pass",
    "mx": "pass",
    "smtp": "pass",
    "role": "pass",
    "free_provider": true
  },
  "details": {
    "mxHost": "mx1.example.com",
    "is_role": false,
    "is_free_provider": true,
    "free_provider_name": "Gmail",
    "is_disposable": false,
    "has_typo_suggestion": false,
    "risk_level": "low",
    "catch_all": false
  },
  "cached": false,
  "credits_remaining": 4850
}
```

**Validation Steps:**
1. **Syntax (15 points)** - RFC 5322 compliant format
2. **Disposable (20 points)** - Not from disposable/temporary providers
3. **MX Records (30 points)** - Valid mail server records exist
4. **SMTP (35 points)** - Mail server accepts the recipient
5. **Role-based (-10 points)** - Penalty for addresses like info@, admin@

**Score Thresholds:**
- **≥ 80**: Low risk (valid)
- **50-79**: Medium risk
- **< 50**: High risk (invalid)

---

### 2. Bulk Email Verification
**POST** `/verify/bulk`

Verifies up to 100 email addresses in a single request.

**Request Body:**
```json
{
  "emails": ["user1@example.com", "user2@gmail.com"],
  "webhook_url": "https://your-server.com/webhook"
}
```

**Response:**
```json
{
  "results": [
    {
      "email": "user1@example.com",
      "valid": false,
      "score": 15,
      "reason": "Disposable email provider",
      "checks": { "syntax": "pass", "disposable": "fail" }
    },
    {
      "email": "user2@gmail.com",
      "valid": true,
      "score": 100,
      "reason": "deliverable",
      "checks": { "syntax": "pass", "mx": "pass", "smtp": "pass" }
    }
  ],
  "total": 2,
  "unique_verified": 2,
  "credits_remaining": 47
}
```

---

### 3. Email Authenticity Analysis
**POST** `/verify/auth`

Analyzes email headers to detect spoofing, phishing, and CEO fraud.

**Request Body:**
```json
{
  "headers": "From: sender@example.com\nReceived: from ...",
  "trusted_domains": ["yourcompany.com"]
}
```

---

### 4. Account & Authentication
**GET** `/account` - Get account details and usage

**POST** `/signup` - Create new account

**POST** `/account/rotate-key` - Rotate API key

---

## Usage Examples

### Verify Single Email
```javascript
const result = await fetch('https://api.mailcheck.dev/v1/verify', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ email: 'user@example.com' }),
});

const data = await result.json();
if (data.valid) {
  console.log('Email is deliverable');
}
```

### Bulk Verification
```javascript
const result = await fetch('https://api.mailcheck.dev/v1/verify/bulk', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ emails: ['a@example.com', 'b@example.com'] }),
});

const data = await result.json();
const validEmails = data.results.filter(r => r.valid).map(r => r.email);
```

### Authenticity Check
```javascript
const result = await fetch('https://api.mailcheck.dev/v1/verify/auth', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ 
    headers: emailHeaders, 
    trusted_domains: ['company.com'] 
  }),
});

const data = await result.json();
if (data.trust_score < 70) {
  console.log('Suspicious email detected');
}
```

---

## Error Handling

All errors return JSON: `{ "error": "...", "message": "..." }`

| Status | Error | Description |
|--------|-------|-------------|
| 400 | ValidationError | Missing or invalid parameters |
| 401 | Unauthorized | Invalid API key |
| 403 | AccountSuspended | Account deactivated |
| 429 | RateLimitExceeded | Quota exceeded |
| 500 | InternalError | Server error |

---

## Best Practices

1. **Cache Results**: Cached verifications don't count against quota
2. **Use Bulk API**: More efficient for lists > 10 emails
3. **Check Risk Level**: Use `details.risk_level` for quick triage
4. **Handle Rate Limits**: Exponential backoff on 429
5. **Rotate Keys**: Periodic API key rotation for security
6. **Verify Webhooks**: Always verify webhook signatures

---

## Environment Setup

Create a `.env` file:
```bash
MAILCHECK_API_KEY=sk_live_your_api_key_here
```

---

## Rate Limits

| Plan | Requests/min | Monthly Limit |
|------|-------------|---------------|
| Free | 10 | 100 |
| Starter | 60 | 5,000 |
| Pro | 120 | 25,000 |
| Enterprise | Unlimited | Custom |

---

## Additional Features

### Typo Detection
Automatically detects common domain typos (e.g., gmial.com → gmail.com).

### Free Provider Detection
Identifies free email providers (Gmail, Yahoo, Outlook, etc.) for B2B lead quality assessment.

### Role-Based Detection
Flags role-based addresses (info@, admin@, sales@) with a 10-point penalty.

### Catch-All Detection
Identifies catch-all domains (10-point penalty).

---

## Support
- **Docs**: https://api.mailcheck.dev/docs
- **Dashboard**: https://api.mailcheck.dev/dashboard
- **Email**: support@mailcheck.dev
