# Server-Side Request Forgery (SSRF)

## Definition

Server-Side Request Forgery (SSRF) is a vulnerability in which an attacker can influence a server-side component to make network requests to unintended destinations. The application may fetch a URL, image, webhook, document, or other resource using its own network access and trust, allowing an attacker to reach systems that are not directly accessible from the internet.

SSRF is commonly classified as **CWE-918**. It may be direct (the response is returned to the requester) or blind (the request is made but its response is not exposed). The core issue is insufficient validation and control of the destination and network activity—not simply the presence of a URL input.

## Attack examples

These scenarios are illustrative. Assess them only in systems you own or are explicitly authorized to test.

- **Internal service access:** A URL-fetching feature is induced to request an internal administrative endpoint or service that trusts traffic from the application network.
- **Cloud metadata exposure:** A vulnerable workload is induced to contact its cloud provider's instance metadata service. If available credentials or configuration are returned, they may grant access to cloud resources.
- **Protocol or parser abuse:** A URL handler accepts unexpected schemes or interprets ambiguous URL syntax in a way that reaches local resources or unintended network services.
- **Redirect-based bypass:** The application validates an initial, apparently acceptable URL, then follows a redirect to a disallowed destination.
- **Blind SSRF:** A server-side request triggers an observable side effect—such as an outbound DNS lookup, a callback, or a change in a controlled test endpoint—even though the application does not return the response body.
- **Internal service chaining:** An attacker uses the vulnerable application as a network pivot to probe or invoke services reachable only from its hosting environment.

## Risks

- **Unauthorized access to internal systems:** Firewalls and network segmentation may trust the application server's network location.
- **Credential and secret exposure:** Cloud metadata or internal endpoints may disclose tokens, keys, or sensitive configuration.
- **Privilege escalation in cloud environments:** Exposed workload credentials can enable access beyond the vulnerable application, depending on their permissions.
- **Data disclosure or modification:** Internal APIs may return sensitive records or permit actions when called from a trusted network.
- **Service disruption:** Requests to sensitive or resource-intensive services can affect availability.
- **Reconnaissance and lateral movement:** Differences in response, timing, or connection behavior may reveal internal hosts and services.
- **Hard-to-detect activity:** Blind SSRF can be difficult to identify from application responses alone and may appear as ordinary outbound traffic.

Impact depends on reachable destinations, the application's network position, available credentials, and the permissions of any exposed services.

## Mitigations

1. **Avoid arbitrary URL fetching.** Prefer fixed, application-controlled destinations or a narrowly defined set of approved hosts.
2. **Use strict allowlists.** Validate the parsed scheme, hostname, and port against explicit business requirements. Reject user-info components, unexpected schemes, and ambiguous or malformed URLs. Do not rely on substring or suffix checks.
3. **Validate resolved addresses.** Resolve hostnames and reject loopback, private, link-local, multicast, reserved, and otherwise prohibited address ranges for both IPv4 and IPv6. Account for multiple DNS answers and re-check the destination at connection time to reduce DNS rebinding and time-of-check/time-of-use risks.
4. **Control redirects.** Disable redirects where possible. Otherwise, validate every redirect target using the same policy and enforce a small redirect limit.
5. **Restrict egress at the network layer.** Use firewall rules, egress proxies, or service-mesh policies to limit which destinations and ports the application can reach. Application validation should not be the only control.
6. **Protect cloud metadata services.** Apply provider-supported metadata protections and require the strongest available request protections. Do not rely on metadata protections as a substitute for SSRF prevention.
7. **Use least privilege.** Limit workload identity and service credentials to the minimum permissions required; avoid embedding secrets in responses or broadly trusted internal services.
8. **Harden the HTTP client.** Set connection and read timeouts, response-size limits, safe protocol handlers, and resource limits. Avoid forwarding sensitive headers, cookies, or credentials to user-selected destinations.
9. **Monitor and log safely.** Record outbound destination and policy decisions without logging secrets or sensitive response bodies. Alert on unexpected destinations, blocked private-address attempts, and unusual request patterns.
10. **Test URL normalization consistently.** Ensure validation and the HTTP client interpret hostnames, IP literals, ports, encodings, and redirects consistently; use well-maintained URL parsing libraries.

## Testing techniques

Perform testing only with written authorization and a defined scope. Prefer a staging environment and controlled endpoints; avoid contacting production metadata services, third-party systems, or internal hosts unless the authorization explicitly permits it.

- **Review data flow:** Trace user-controlled values into HTTP clients, image/document processors, webhook handlers, URL previews, importers, and other components that make outbound requests.
- **Establish a safe baseline:** Use a tester-controlled endpoint to verify whether the feature makes a server-side request, what methods and headers it sends, and whether responses are exposed.
- **Check destination validation:** Test representative allowed and disallowed hostnames and addresses, including loopback, private and link-local ranges, IPv4/IPv6 forms, alternate numeric representations, and malformed or ambiguous URLs. Use benign test targets and confirm the application blocks prohibited destinations.
- **Test redirects:** From a controlled, allowlisted endpoint, return a redirect to a controlled disallowed test destination and verify that the application revalidates or refuses the redirect.
- **Assess DNS handling:** In a controlled environment, verify that validation covers resolved addresses and that the actual connection cannot be redirected to a prohibited address after validation.
- **Check blind behavior:** Use an approved callback or interaction-monitoring service to detect outbound DNS or HTTP interactions when the application does not display a response. Avoid using callback payloads that access sensitive resources.
- **Verify egress controls:** Confirm at the network boundary that prohibited destinations and ports are blocked, including when application-level checks are bypassed in a test environment.
- **Review cloud protections safely:** Inspect configuration and identity permissions rather than attempting to retrieve real metadata credentials. Use provider-approved test procedures where applicable.
- **Validate remediation:** Repeat the same tests after fixes and check logs, alerts, redirect handling, timeouts, and response-size controls.
- **Document findings:** Record the affected feature, prerequisites, demonstrated impact, scope and authorization, evidence with secrets redacted, and practical remediation. Distinguish confirmed behavior from potential impact.

### Consultant and CISSP study points

For an assessment, connect SSRF to **input validation, secure architecture, least privilege, network segmentation, egress filtering, cloud workload identity, logging and monitoring, and secure software development**. Explain both likelihood and impact in context: an SSRF flaw does not automatically imply credential theft, but its severity can rise substantially when the server can reach sensitive services or holds powerful credentials. Recommend layered controls rather than relying on a single URL filter.
