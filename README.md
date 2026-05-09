# NexaCorp CTF - Complete Penetration Testing Writeup

> A FUN comprehensive 5-stage API vulnerability exploitation chain demonstrating real-world breach methodology.

## 🎯 Challenge Overview

**Name:** NexaCorp: Zero Credentials  
**Difficulty:** Intermediate to Advanced  
**Type:** CTF (Capture The Flag) - API Security  
**Platform:** INE (Infosec Institute)  
**Tools Used:** Curl, Burp Suite, Postman, Python

---

## 📋 Attack Chain Overview

```
Stage 1: Reconnaissance
    ├─ Exploit: GraphQL Introspection
    └─ Obtain: recon_token
          ↓
Stage 2: Initial Access
    ├─ Exploit: OAuth Redirect URI Bypass
    └─ Obtain: authorization_code → access_token
          ↓
Stage 3: Privilege Escalation
    ├─ Exploit: Mass Assignment (BOPLA)
    └─ Obtain: super-admin role
          ↓
Stage 4: Lateral Movement
    ├─ Exploit: SSRF via PDF Rendering
    └─ Obtain: AWS credentials & hints
          ↓
Stage 5: System Compromise
    ├─ Exploit: JWT Algorithm Confusion
    └─ Obtain: Full vault access
          ↓
SYSTEM COMPROMISED ✓
```

---
## 🛠️ Tools & Techniques Used

### Primary Tools
- **Burp Suite Community Edition** - Request interception and modification
- **Postman** - API request building and testing
- **Firefox** - Browser with proxy configuration
- **jwt.io** - JWT encoding/decoding

### Key Techniques
1. **HTTP Request Interception** - Capturing and modifying requests
2. **Parameter Tampering** - Modifying values to find vulnerabilities
3. **Endpoint Discovery** - Finding API endpoints through introspection
4. **Token Manipulation** - Forging and modifying JWTs
5. **SSRF Exploitation** - Abusing server-side requests

---

## 📊 Vulnerability Classification

| Flag | CWE | OWASP API | Severity |
|------|-----|-----------|----------|
| 1 | CWE-200 | API5:2023 | HIGH |
| 2 | CWE-290 | API2:2023 | HIGH |
| 3 | CWE-915 | API3:2023 | CRITICAL |
| 4 | CWE-918 | API5:2023 | CRITICAL |
| 5 | CWE-347 | API2:2023 | CRITICAL |

---

## 🎓 Key Learnings

### What Each Vulnerability Teaches

1. **GraphQL Introspection**
   - Always disable introspection in production
   - Security through obscurity is not security
   - Debug endpoints should never exist in prod

2. **OAuth Bypass**
   - String matching is dangerous for security
   - Use exact equality, not partial matching
   - Validate all security-critical values strictly

3. **Mass Assignment**
   - Never use Object.assign() with user input
   - Implement allowlisting, not blacklisting
   - Explicitly define modifiable fields

4. **SSRF**
   - Never pass user input to rendering engines
   - Sanitize HTML aggressively
   - Restrict outbound requests from servers

5. **JWT Confusion**
   - Pin algorithms server-side
   - Don't trust client-specified values
   - Whitelist acceptable algorithms
---

## 📚 Resources

### Documentation
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [GraphQL Security Best Practices](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)

### Tools
- [Burp Suite Community](https://portswigger.net/burp/community)
- [Postman](https://www.postman.com/)
- [JWT.io](https://jwt.io)

---

## 📝 Notes

- **Lab Platform:** INE (Infosec Institute)
- **Completion Date:** May 6-7, 2026
- **Difficulty Rating:** Intermediate to Advanced
- **Recommended For:** Anyone learning API security and penetration testing
