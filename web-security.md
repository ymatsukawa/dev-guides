# Web Security
This document describes the essence of Web Security and the recommendations and deprecations that support it.

## Essence of Web Security

In one sentence, Web Security is the practice of protecting web applications and their users from unauthorized access, data breaches, and malicious attacks while ensuring continuous availability of services. When implementing web security, follow these philosophies:

- [ ] **Ensure CIA Triad (Confidentiality, Integrity, Availability)**: Always evaluate data protection needs and apply appropriate controls for confidentiality through encryption, integrity through authentication codes, and availability through resilience design
- [ ] **Never Trust Unvalidated User Input**: Treat all external input as untrusted data regardless of source, validate against explicit allowlists, and properly escape/sanitize output based on context (SQL, HTML, OS commands)
- [ ] **Enforce Principle of Least Privilege**: Grant only the minimum necessary permissions for each actor to perform their role, make authorization decisions based on context, and implement zero-trust principles between components

## Recommendations

**Build security into the design from the earliest stages**:

Security and reliability are emergent properties of systems that must be considered from the initial design phase. Retrofitting security into existing systems is difficult, costly, and often results in systemic vulnerabilities. Early security consideration prevents architectural flaws that could delay security reviews and create cascading vulnerabilities throughout the system lifecycle.

Implement security thinking throughout the entire software development lifecycle by conducting threat modeling during design phases, integrating security controls into the architecture, and treating configuration as code with mandatory peer review. Use deployment methods like canary releases and staged rollouts to limit security risks. In zero-trust models, security should be fundamentally embedded in system operations as an enabler rather than a restriction.

**Leverage frameworks and automation for known vulnerability classes**:

Common vulnerability classes like SQL injection and Cross-Site Scripting (XSS) continue to plague applications because developers cannot be experts in all security domains. Instead of relying solely on developer vigilance, use security frameworks that enforce safe practices through type systems and compiler checks.

Implement prepared statements for all database queries using placeholders rather than string concatenation. Use security-aware frameworks that provide types like `TrustedSqlString` for SQL and `SafeHtml` for XSS prevention. Integrate static analysis tools (e.g., SonarQube) and dynamic analysis tools (e.g., OWASP ZAP) into your development pipeline. Build vulnerability management systems that intelligently filter false positives and learn from previous experiences. Generate source provenance and build attestations during the build process and verify them in deployment policies.

**Establish supply chain transparency and trustworthiness**:

Modern software overwhelmingly consists of open-source components, many containing unpatched vulnerabilities in unknown locations. Supply chain attacks bypass traditional perimeter defenses by compromising legitimate software updates that organizations themselves request. Single vulnerable components can exponentially expand risk profiles through reuse across ecosystems.

Generate Software Bills of Materials (SBOMs) to provide visibility into OSS components and their known vulnerabilities. Use Vulnerability Exploitability eXchange (VEX) documents to provide context about whether vulnerabilities actually affect your products. Obtain components only from trusted sources and establish internally verified component repositories. Harden build environments to the same security level as source code and products. Apply zero-trust principles throughout the software development lifecycle with fine-grained access controls independent of network location.

**Monitor production systems continuously with incident response plans**:

Prevention mechanisms will fail, making detection and recovery planning essential. Early incident detection can mean the difference between a minor security event and a complete breach. Recovery efforts inevitably involve human actions, and humans are unpredictable, making recovery design validation particularly important.

Implement audit logging for accountability, adding basic logging functionality to APIs that captures who did what and when, including unsuccessful operations to detect attack attempts. In production, send logs to SIEM systems for correlation and analysis. Design systems for understandability and manage complexity to reduce vulnerability likelihood. When granting debug access, implement appropriate security mechanisms balancing developer needs with data protection requirements. Maintain detailed records of intended system states to enable restoration to known-good configurations.

**Implement proper authentication and token management**:

Web-friendly authentication mechanisms must balance security with usability. HTTP Basic authentication's weaknesses necessitate modern token-based approaches that protect against various attack vectors while remaining practical for web applications.

Use OAuth 2.0 or similar modern token-based authentication with Bearer schemes in Authorization headers. For self-contained tokens like JWTs, define standard claims (exp, sub, iss, aud, jti) and validate issuer and audience claims to ensure tokens are intended for your API. Strictly validate JWT signature algorithms to prevent algorithm confusion attacks. When storing tokens in Web Storage instead of session cookies, implement complete XSS attack prevention. Encrypt JWTs containing sensitive attributes but avoid returning decryption failure details to users.

**Secure service-to-service communication**:

Microservice architectures require protection against eavesdropping and tampering of inter-service network traffic. Service mesh technologies provide comprehensive security without burdening individual services with security implementation details.

Use TLS/HTTPS for all service-to-service communication. Authenticate services using API keys or JWT bearer authentication where developer portals sign JWTs with private keys and APIs verify with public keys. In Kubernetes environments, deploy service meshes like Istio to enforce edge TLS termination and mutual TLS between microservices. Use Envoy proxies to handle JWT validation and mTLS handshakes. Define access control policies based on JWT attributes using AuthorizationPolicy CRDs.

**Implement comprehensive CSRF protection**:

Session cookie-based APIs remain vulnerable to Cross-Site Request Forgery attacks where malicious sites leverage authenticated user sessions to perform unauthorized operations.

Apply SameSite cookie attributes (strict or lax) to ensure cookies are only sent with same-domain requests. Implement hash-based double-submit cookies requiring custom headers (e.g., X-CSRF-Token) that match session cookie hashes. Compare CSRF tokens using constant-time comparison functions to prevent timing attacks. For maximum protection, combine multiple CSRF defense mechanisms rather than relying on a single approach.

**Prevent Server-Side Request Forgery (SSRF) attacks**:

SSRF vulnerabilities allow attackers to access internal network services or cause unintended side effects by manipulating URL inputs that the server processes.

Strictly validate all user-provided URLs before processing. Match only against allowlists of known-safe URLs. Block private IP addresses including loopback, link-local, site-local, multicast, any-local, and unique-local addresses. Prevent DNS rebinding attacks by validating Host headers against expected hostnames. Implement network segmentation to limit the impact of successful SSRF attacks.

## Deprecations

**Avoid retrofitting security as an afterthought**:

Security is an emergent property that becomes exponentially more difficult and costly to implement after system design is complete. Architectural decisions made without security consideration create systemic vulnerabilities that cascade throughout the application. Late-stage security additions often conflict with existing functionality, require extensive refactoring, and leave gaps that attackers can exploit.

**Avoid embedding unvalidated user input directly in dynamic code**:

Direct string concatenation of user input into SQL queries, system commands, or dynamic code execution contexts creates injection vulnerabilities. Attackers can manipulate input to execute arbitrary commands, access unauthorized data, or compromise system integrity. Even with attempted sanitization, the complexity of encoding rules across different contexts makes this approach fundamentally unsafe.

**Avoid exposing technical details in error messages**:

Detailed error messages containing stack traces, framework versions, or internal system information provide attackers with reconnaissance data. This information helps identify specific vulnerabilities, plan targeted attacks, and understand system architecture. Production systems should return generic error messages while logging detailed information securely for authorized personnel only.

**Avoid exposing unintended HTTP methods**:

Imprecise HTTP method declarations that accept all methods when only specific ones are intended create security misconfigurations. This allows attackers to bypass security controls, access functionality through unexpected paths, or cause unintended state changes. Each endpoint should explicitly declare and validate acceptable HTTP methods.

**Avoid using session cookies without CSRF protection**:

Cookies automatically sent with cross-site requests enable CSRF attacks where malicious sites perform actions using victims' authenticated sessions. Without proper CSRF tokens or SameSite attributes, attackers can trick users into performing unintended actions like fund transfers or configuration changes while appearing legitimate to the server.

**Avoid trusting self-signed tokens without authentication**:

Tokens without cryptographic signatures or authentication tags can be trivially forged or modified by attackers. This allows elevation of privileges, impersonation of other users, or bypass of security controls. All tokens must include tamper-evident authentication mechanisms verified by recipients.

**Avoid storing tokens in Web Storage without XSS protection**:

Unlike HttpOnly cookies, Web Storage remains accessible to JavaScript, making tokens vulnerable to XSS-based exfiltration. A single XSS vulnerability can compromise all stored tokens, leading to account takeovers. If Web Storage must be used, comprehensive XSS prevention becomes critical.

**Avoid exposing microservices directly to clients**:

Direct client access to microservices multiplies authentication implementation, increases attack surface, and complicates security updates. Each service must independently implement security controls, leading to inconsistencies and increased maintenance burden. API gateways provide centralized security enforcement and simplified client interaction.

**Avoid leaving CORS misconfigured**:

Improper CORS configuration either blocks legitimate cross-origin requests or allows overly permissive access. This can prevent frontend applications from functioning or enable unauthorized cross-origin data access. CORS policies must be explicitly configured to balance functionality with security.

**Avoid accepting arbitrary URLs for server-side fetching**:

Processing user-provided URLs without validation enables SSRF attacks accessing internal services, cloud metadata endpoints, or causing denial of service. Attackers can map internal networks, access sensitive data, or pivot to other systems. All URLs must be validated against strict allowlists and internal address ranges must be blocked.

**Avoid ignoring replay attack vectors**:

Even authenticated messages can be maliciously replayed to cause unintended effects like duplicate transactions or state corruption. Short-lived timestamps alone provide insufficient protection within their validity window. Implement nonces or conditional requests to ensure message freshness and prevent replay attacks.

**Avoid exposing sensitive data in URLs**:

URLs persist in browser history, server logs, referrer headers, and proxy logs where sensitive data becomes exposed. Access tokens, session identifiers, or confidential parameters in URLs create multiple points of potential leakage. Use request headers or bodies for sensitive data transmission instead.