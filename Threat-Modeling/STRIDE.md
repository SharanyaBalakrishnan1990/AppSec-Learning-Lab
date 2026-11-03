# STRIDE Threat Modeling — Study Notes

## What STRIDE means

**STRIDE** is a threat-classification mnemonic used during threat modeling to identify security properties that a system may fail to provide. It was developed at Microsoft and is commonly applied to architecture diagrams, data flows, trust boundaries, services, and data stores.

Use it to ask, for each component and data flow: *How could this be spoofed, tampered with, repudiated, disclosed, denied, or elevated?* STRIDE helps organize analysis; it is not a risk score or a substitute for assessing likelihood, impact, and context.

## The six categories

| Category | Security property at risk | Meaning |
|---|---|---|
| **S — Spoofing** | Authentication | Pretending to be another user, service, or system. |
| **T — Tampering** | Integrity | Unauthorized modification of data, code, requests, or configuration. |
| **R — Repudiation** | Non-repudiation / accountability | A party can deny an action because trustworthy evidence is absent or inadequate. |
| **I — Information Disclosure** | Confidentiality | Sensitive information is exposed to an unauthorized party. |
| **D — Denial of Service** | Availability | A system or resource is made unavailable or too slow for legitimate users. |
| **E — Elevation of Privilege** | Authorization | A user or process gains permissions beyond those intended. |

## Public-facing web application examples

Examples below are prompts for analysis, not findings about a particular application.

- **Spoofing:** An attacker takes over a customer account using stolen credentials or forges a session identity. A backend service accepts an unauthenticated or incorrectly authenticated caller.
- **Tampering:** A client changes a price, account identifier, or workflow state in a request; an attacker alters data in transit or modifies application configuration without authorization.
- **Repudiation:** A user denies changing a recovery email or placing an order, and the application has no reliable, appropriately protected audit trail to establish what happened.
- **Information Disclosure:** A page, API response, error message, log, or misconfigured object-storage location exposes personal data, credentials, or internal details. One user can access another user's records.
- **Denial of Service:** Excessive requests, expensive search/report operations, or resource exhaustion disrupts service for legitimate users.
- **Elevation of Privilege:** A standard user reaches an administrator function, changes another user's data, or bypasses a server-side authorization check by manipulating a request.

## Example threats for an API

Consider a public API with a mobile/web client, an API gateway, application services, and a database.

| STRIDE | API threat example |
|---|---|
| **Spoofing** | A caller reuses a stolen bearer token, or a service accepts a token with an invalid issuer, audience, or signature. |
| **Tampering** | A caller changes object identifiers or writable fields in a JSON body; a downstream service trusts client-supplied values that should be authoritative on the server. |
| **Repudiation** | The system cannot correlate a sensitive API action to a principal and request because audit events omit identity, timestamp, or request context, or can be altered by the actor. |
| **Information Disclosure** | An endpoint returns fields the caller does not need, leaks data across tenants, or exposes secrets and personal information in errors or logs. |
| **Denial of Service** | Unbounded pagination, large payloads, costly queries, or high request rates consume compute, database connections, or third-party quotas. |
| **Elevation of Privilege** | Broken object-level or function-level authorization lets a caller read or change another account's objects or invoke privileged operations. |

## Mitigations and design checks

Apply controls in layers and validate them at the relevant trust boundary. Select controls based on the data, architecture, and risk.

- **Spoofing:** Use strong authentication and MFA where appropriate; protect sessions and tokens; validate token signature, issuer, audience, expiry, and intended use; rotate and revoke credentials; authenticate service-to-service traffic; use TLS.
- **Tampering:** Enforce authorization and business rules on the server; validate input with allowlists and schemas; use parameterized database operations; protect integrity of messages and artifacts; use secure change control and code signing where appropriate; avoid trusting client-provided prices, roles, or ownership.
- **Repudiation:** Record security-relevant events with reliable timestamps and principal, action, target, and outcome; centralize and restrict access to logs; protect logs against alteration and define retention; avoid logging secrets or unnecessary personal data; synchronize system clocks.
- **Information Disclosure:** Apply least privilege and object-/field-level authorization; minimize collected and returned data; encrypt data in transit and at rest as appropriate; manage secrets securely; use safe error responses; redact sensitive logs; review caching, backups, and storage permissions.
- **Denial of Service:** Rate-limit and apply quotas; bound payload sizes, pagination, execution time, and concurrency; use timeouts, backpressure, and resource limits; protect expensive endpoints; monitor capacity and errors; use resilient dependency and recovery designs.
- **Elevation of Privilege:** Apply least privilege and deny-by-default authorization; check permissions on every server-side request and target object; separate duties and administrative paths; validate roles and claims from trusted sources; protect privileged workflows; test authorization across users, tenants, and roles.

## Applying STRIDE in practice

1. Define scope, assets, security objectives, and assumptions.
2. Draw the architecture and data-flow diagram, including external actors, processes, data stores, data flows, and **trust boundaries**.
3. Walk each element and flow through all six STRIDE categories; record plausible threat scenarios and affected assets.
4. Assess likelihood and impact, including privacy, business, and safety consequences; prioritize using the organization's risk method.
5. Choose mitigations, owners, and verification methods. Track residual risk and revisit the model when architecture or threats change.

**CISSP / AppSec reminder:** STRIDE identifies *types* of threats. Connect each scenario to the relevant security objective and control, verify controls with review and testing, and document accepted residual risk. No single control eliminates every threat.
