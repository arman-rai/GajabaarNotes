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
Level 2: The Magic Door
Flag: flag-57936b
### Prompt Injection & Social Engineering

**What happened:** The agent was manipulated through carefully crafted prompts to reveal tool descriptions and execute privileged operations.

**Why it's dangerous:** AI agents can be tricked into bypassing their own security restrictions through social engineering techniques, even when tool descriptions are hidden by default.

**Prevention:**

- Implement strict input validation and sanitization for all user inputs
- Use role-based access control (RBAC) at the tool level, not just in prompts
- Never rely solely on prompt-based security controls
- Log and monitor all tool invocations for suspicious patterns
- Design tools with built-in authorization checks independent of agent instructions

**Key lesson:** If security depends on the agent "following instructions," it's not secure. Enforce security at the system level.

---
Level 3: The Lost Archives: Part 1
lol got rickrolled.
