# 🌐 HTTP Status Codes Cheat Sheet

HTTP status codes indicate how a server responded to a client's request.

---

# 📂 1xx — Informational

| Code | Meaning |
|------|----------|
|100|Continue|
|101|Switching Protocols|
|102|Processing|

These are rarely encountered during web application testing.

---

# ✅ 2xx — Success

| Code | Meaning |
|------|----------|
|200|OK|
|201|Created|
|202|Accepted|
|204|No Content|

### Pentesting Notes

- 200 → Resource exists
- 201 → New object created
- 204 → Action succeeded but no response body

---

# ↪️ 3xx — Redirection

| Code | Meaning |
|------|----------|
|301|Moved Permanently|
|302|Found|
|303|See Other|
|304|Not Modified|
|307|Temporary Redirect|
|308|Permanent Redirect|

### Pentesting Notes

Useful for:

- Open Redirect testing
- Authentication bypass
- OAuth testing

---

# ❌ 4xx — Client Errors

| Code | Meaning |
|------|----------|
|400|Bad Request|
|401|Unauthorized|
|403|Forbidden|
|404|Not Found|
|405|Method Not Allowed|
|406|Not Acceptable|
|408|Request Timeout|
|409|Conflict|
|413|Payload Too Large|
|415|Unsupported Media Type|
|429|Too Many Requests|

### Pentesting Notes

401

- Authentication required

403

- Resource exists
- Access denied
- Test for IDOR or ACL bypass

404

- May reveal hidden resources

405

- Check allowed HTTP methods

429

- Indicates rate limiting

---

# 💥 5xx — Server Errors

| Code | Meaning |
|------|----------|
|500|Internal Server Error|
|501|Not Implemented|
|502|Bad Gateway|
|503|Service Unavailable|
|504|Gateway Timeout|

### Pentesting Notes

500

- Often leaks stack traces
- Useful for fuzzing

502

- Reverse proxy issue

503

- Maintenance
- Sometimes useful for DoS testing (where authorised)

---

# 🎯 Most Important Codes for Pentesters

| Code | Why Important |
|------|---------------|
|200|Successful request|
|201|Object created|
|204|Request processed|
|301|Permanent redirect|
|302|Temporary redirect|
|400|Malformed request|
|401|Authentication required|
|403|Authorization failure|
|404|Hidden endpoint discovery|
|405|Allowed methods|
|429|Rate limiting|
|500|Potential information disclosure|

---

# 💡 Quick Tips

- 401 → Authentication issue
- 403 → Authorization issue
- 404 → Doesn't always mean resource doesn't exist
- 500 → Inspect carefully for leaked information
- 302 → Common during login workflows

---

# References

- RFC 9110
- MDN HTTP Documentation