
# Insecure Direct Object Reference (IDOR)

## ✅ Description

**Insecure Direct Object Reference (IDOR)** is a type of access control vulnerability that occurs when an application provides direct access to objects (like database records, files, or keys) based on user-supplied input—without properly verifying that the user is authorized to access the requested object.

For example, URLs like the following may expose sensitive data if no checks are in place:

```
https://example.com/user/profile?user_id=123
```

If a malicious user changes the `user_id` to another number, like `124`, and accesses another user's profile without authorization, this is an IDOR vulnerability.

---

## ⚠️ Why It’s Dangerous

- Unauthorized data access
- Information leakage
- User impersonation
- Privilege escalation

---

## 🧪 Example Scenario

### Request from a legitimate user:

```
GET /invoice/1023 HTTP/1.1
Host: vulnerable-website.com
Authorization: Bearer <token>
```

**Response:**

```json
{
  "invoice_id": 1023,
  "amount": "$250.00",
  "user": "user123"
}
```

If the user modifies the invoice ID:

```
GET /invoice/1024 HTTP/1.1
```

And **successfully receives another user's invoice**, this indicates IDOR.

---

## 🔐 Prevention Techniques

- **Use indirect references**: Replace IDs with random tokens (e.g., UUIDs).
- **Enforce access controls**: Always check object ownership on the server side.
- **Avoid exposing raw database IDs**.
- **Implement RBAC (Role-Based Access Control)**.
- **Conduct security testing (manual & automated)**.

---

## 🛠️ Testing for IDOR (Burp Suite Example)

1. Intercept request with Burp Suite.
2. Identify a parameter that directly references an object (e.g., `user_id`, `invoice_id`).
3. Modify the ID to see if unauthorized access is possible.
4. Confirm whether the server responds with unauthorized data.

---

## 📥 Download This Documentation

To download this file to your system as `idor.md`, run:

```bash
curl -o idor.md https://raw.githubusercontent.com/yourusername/secure-docs/main/idor.md
```

> 📌 **Note**: Replace the URL with the actual location where the `.md` file is hosted (e.g., on GitHub, GitLab, or your server).

---

## 📚 References

- [OWASP Top 10: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-639: Authorization Bypass Through User-Controlled Key](https://cwe.mitre.org/data/definitions/639.html)
