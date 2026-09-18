Level 1: The Context Gateway
Flag: flag-13c0ce
### Information Disclosure Vulnerability

**What happened:** Sensitive information (the scan_mode passkey) was leaked through system logs and tool descriptions.

**Why it's dangerous:** Information disclosure allows attackers to bypass authentication mechanisms, discover hidden functionality, or gain insights into system internals without proper authorization.

**Prevention:**

- Never log sensitive credentials, tokens, or secrets
- Sanitize tool descriptions to avoid revealing implementation details
- Use separate environments for development (with verbose logging) and production
- Implement proper access controls and least privilege principles

**Key lesson:** What seems like harmless diagnostic information can become a critical security vulnerability in the wrong hands.

---
