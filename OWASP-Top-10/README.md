# OWASP Top 10 Learning Notes

This folder contains learning notes and examples for the OWASP Top 10 vulnerabilities.

## Topics

- [Broken Access Control](#a01-broken-access-control)
- Cryptographic Failures
- Injection
- Insecure Design
- Security Misconfiguration
- Vulnerable Components
- Identification and Authentication Failures
- Software and Data Integrity Failures
- Security Logging and Monitoring Failures
- Server-Side Request Forgery (SSRF)

## A01:2021 — Broken Access Control

### What it is

Broken access control occurs when an application fails to enforce whether a user may perform a particular action on a specific resource. Authentication answers **“Who are you?”**; authorization answers **“What are you allowed to do?”** A valid login does not guarantee that a user is authorized to access every record or function.

OWASP A01:2021 covers failures such as bypassing authorization checks, accessing another user’s data, changing permissions, and reaching restricted functions or files. It is an authorization failure with potential consequences for confidentiality, integrity, and availability.

### Common attack scenarios

- **IDOR / horizontal privilege escalation:** A user changes or supplies a different record reference and accesses another user’s invoice, profile, or document because ownership is not checked.
- **Vertical privilege escalation:** A standard user invokes an administrative function or gains access to higher-privilege actions.
- **Function-level authorization failure:** A restricted action is hidden in the interface, but the server accepts a direct request to perform it.
- **Property-level authorization failure:** A request modifies fields the user should not control, such as a role, owner, price, or approval status.
- **Forced browsing:** A user reaches a restricted endpoint or exposed file by requesting its location directly.
- **Cross-tenant access:** A user in one customer or organization context retrieves or modifies another tenant’s data because tenant boundaries are not enforced.

### Risks and impact

- Disclosure or modification of sensitive or personal information
- Unauthorized transactions, account changes, or administrative actions
- Privilege escalation and broader compromise of the application
- Data loss or service disruption
- Privacy, regulatory, contractual, and reputational consequences

For a CISSP perspective, relate this category to least privilege, need-to-know, separation of duties, and the confidentiality, integrity, and availability objectives. Effective authorization also supports accountability when decisions and denials are logged appropriately.

### Mitigations

- Enforce authorization **server-side on every request**, including API calls; never rely on a hidden button or client-side check.
- Use **deny by default** and grant only explicitly approved actions.
- Check permissions against both the user’s role and the specific resource, including its owner and tenant.
- Centralize and consistently apply authorization policies; test all routes and service-layer operations.
- Apply least privilege to users, administrators, and service accounts, and separate sensitive duties where appropriate.
- Allow-list fields a user may read or update. Do not blindly bind client-supplied properties to internal objects.
- Treat unpredictable or indirect identifiers as defense in depth, not as a substitute for authorization checks.
- Protect state-changing browser requests with appropriate CSRF defenses, and require re-authentication or step-up authentication for high-impact actions when warranted.
- Log authorization failures and monitor suspicious access patterns without recording sensitive data unnecessarily.

### Common testing methods

Perform these checks only in systems you own or are explicitly authorized to assess.

1. **Create an authorization matrix:** Map roles and user contexts to resources and actions (read, create, update, delete, administer).
2. **Test horizontal boundaries:** Use approved test accounts to verify that one user cannot access another user’s records.
3. **Test vertical boundaries:** Verify that lower-privileged accounts cannot call higher-privilege functions, even when bypassing the UI.
4. **Exercise endpoints and methods:** Check API routes, alternate HTTP methods, downloads, exports, and workflows—not just visible pages.
5. **Test object and property authorization:** Attempt controlled access to records outside the test user’s scope and updates to protected fields.
6. **Test unauthenticated and session boundaries:** Confirm protected resources reject unauthenticated, expired, and logged-out sessions.
7. **Add negative automated tests:** Assert that unauthorized requests are denied and cause no data disclosure or state change.
8. **Review implementation and logs:** Inspect authorization checks, tenant filters, and denial handling; confirm failures are observable without leaking sensitive information.

A useful assessment outcome records the affected role, resource, action, expected policy, observed result, impact, and evidence—without retaining unnecessary sensitive data.
