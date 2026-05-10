# FLAG 2: OAuth Redirect URI Bypass

[← Back to README](README.md) | [Next: FLAG 3 →](FLAG-3-privilege-escalation.md)

<img width="808" height="138" alt="image" src="https://github.com/user-attachments/assets/144c73fb-ff0f-47c0-b2a3-a4a45fc55e45" />

---

## 🎯 Challenge Summary

| Aspect | Details |
|--------|---------|
| **Vulnerability Type** | Authentication Bypass |
| **Points** | 450 pts |
| **Prerequisite** | Need recon_token from FLAG 1 |

---

## 📖 What is OAuth 2.0?

OAuth is like using your Google account as a VIP backstage pass so you don’t have to create your 847th password. 😭

### Normal OAuth Flow

```
User clicks "Login with NexaCorp"
         ↓
Browser goes to NexaCorp authorization page
         ↓
User enters credentials
         ↓
User clicks "Allow access"
         ↓
Server generates authorization code
         ↓
Browser redirects back with code
         ↓
App exchanges code for access_token
         ↓
User is now logged in ✓
```

### The redirect_uri Parameter

The `redirect_uri` tells the server: "After user approves, send them here"

**Good practice:**
```
Only allow: https://app.nexacorp.io/callback
Anything else: REJECT
```

**Why this matters:**
- If attacker could change redirect_uri to their server
- Authorization code would be sent to attacker
- Attacker would own the user's account

---

## 🔍 The Vulnerability

### What Was Wrong?

The server validated redirect_uri using `startsWith()` instead of exact match:

```javascript
// VULNERABLE CODE
if (!redirectUri.startsWith("https://app.nexacorp.io/callback")) {
  throw Error("Invalid redirect_uri");
}

// This allows BOTH:
✓ https://app.nexacorp.io/callback
✓ https://app.nexacorp.io/callback?attacker.com
✓ https://app.nexacorp.io/callback.attacker.com
```

### Why This Happens

- Developers assume startsWith() is safe
- They don't think about URL manipulation
- Lack of security code review
- Copy-pasted from insecure examples

---

## 🚀 Exploitation

### Prerequisites

I used:
- `recon_token` from FLAG 1
### Created First Request (Got Authorization Code)

**In Postman:**

```
Method: GET
URL: http://nexacorp.ine.local:1337/oauth/authorize
```
oauth/authorize is usually the default endpoint to get the authorization code

### Added Query Parameters
Used the "Params" tab and added:
```
KEY               VALUE
─────────────────────────────────────────────
client_id         nexacorp-prod
redirect_uri      https://app.nexacorp.io/callback
response_type     code
state             xyz123
recon_token       [my token from FLAG 1]
```
```
```

### Response with authourization code
In "Headers" tab of response:
```
Location: https://app.nexacorp.io/callback?code=4f7eedf5457741c3b6527fe5a543c3bb&state=xyz123
```

**Extracted the authorization code:**
```
code = 4f7eedf5457741c3b6527fe5a543c3bb
```

---

### Exchanged Code for Access Token

**Created new Postman request:**

```
Method: POST
URL: http://nexacorp.ine.local:1337/oauth/token
```

**Body:**
```
KEY              VALUE
───────────────────────────────────
grant_type       authorization_code
code             4f7eedf5457741c3b6527fe5a543c3bb
redirect_uri     https://app.nexacorp.io/callback
client_id        nexacorp-prod
```

### Flag and Access Token Retrieved!

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 28800,
  "scope": "openid profile",
  "flag": "FLAG{04uth_r3d1r3ct_ur1_byp4ss}",
  "_note": "redirect_uri validation used startsWith() — any URI starting with the valid prefix was accepted."
}
```
- The `access_token` (you'll need this for FLAG 3!)

---

## 💡 Why This Works

### The Bug

```javascript
// VULNERABLE
if (!redirectUri.startsWith("https://app.nexacorp.io/callback")) {
  // Allow the request
}

// The server only checks if URL STARTS with the valid one
// It doesn't check what comes AFTER!
```

### Example Valid Redirects (All will get a bypass!)

```
✓ https://app.nexacorp.io/callback
✓ https://app.nexacorp.io/callback?
✓ https://app.nexacorp.io/callback?extra=data
✓ https://app.nexacorp.io/callback/../attacker.com
✓ https://app.nexacorp.io/callback@attacker.com
```

### How to Fix It

```javascript
// SECURE - Exact URL validation
if (redirectUri !== "https://app.nexacorp.io/callback") {
  throw Error("Invalid redirect_uri");
}
```

---

## 📊 Authorization Flow Diagram

```
┌──────────┐                    ┌──────────────┐               ┌─────────┐
│  Browser │                    │  NexaCorp    │               │Attacker │
│  (User)  │                    │  Server      │               │         │
└──────────┘                    └──────────────┘               └─────────┘
     │                               │                             │
     │ 1. User clicks login          │                             │
     ├──────────────────────────────>│                             │
     │                               │                             │
     │ 2. Server asks user to        │                             │
     │    approve access             │                             │
     │<──────────────────────────────┤                             │
     │                               │                             │
     │ 3. User approves              │                             │
     ├──────────────────────────────>│                             │
     │                               │                             │
     │ 4. Server generates code      │                             │
     │                               │                             │
     │ 5. Server redirects to URI    │                             │
     │    (should be: callback)      │                             │
     │    (if vulnerable: attacker)  │                             │
     │<──────────────────────────────┤                             │
     │    with authorization code    │                             │
     │                               │                             │
     │ 6. Browser goes to:           │                             │
     │    attacker.com?code=XXX      │                             │
     ├────────────────────────────────────────────────────────────>│
     │                               │                             │
     │                               │                  7. Attacker has code!
     │                               │                     Can exchange for token
     │                               │                     and login as user
```

---

## 🛡️ Key Takeaways

1. **String matching is dangerous for security**
   - Use exact equality (`===`), not partial (`startsWith()`)

2. **OAuth requires strict validation**
   - Redirect URIs must match exactly
   - No partial matches allowed
   - Use allowlist of valid URIs

3. **This is a common vulnerability**
   - Many OAuth implementations have this bug
   - Usually found during security audits
   - Easy to fix once discovered
