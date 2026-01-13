# WG11 Cybersecurity Testing

## Documents Reviewed
> - O-RAN.WG11.TS.SRCS.O-R004-v13.00 (@SPEC1) 
> - O-RAN.WG11.TS.SecProtSpec.O-R004-v13.00 (@SPEC2)
> - O-RAN.WG11.TR.Threat-Modeling.O-R004-v07.00 (@SPEC3)
> - (Optional skim) O-RAN.WG11.TR.Security-Near-RT-RIC-xApps.O-R004-v06.00  (@SPEC4)

---

## Objective
The goal of this note is to explore the official O-RAN WG11 security specifications to understand **what security requirements exist**, **how they are protected**, and **what can be tested in practice**, especially for near-RT RIC-related components.

This is an exploration pass to build a **security knowledge map**, not a deep implementation yet.

---

## 1. Security Test Scope & Target Components @SPEC1
> This section defines the Near-RT RIC components, interfaces, and security controls that are in scope for security testing, as specified by WG11.

### 1.1 Near-RT RIC & xApp @SPEC1

#### 1.1.1 Requirements
This section summarizes the official O-RAN WG11 security requirements for the Near-RT RIC platform, xApps, and their associated interfaces. These requirements define what security properties must be ensured, not how they are implemented. </br>
WG11 assumes that components may be authenticated but still malicious at runtime, so security must go beyond basic protocol protection.

##### 1.1.1.1 xApp Identity & Registration Requirements
WG11 requires that xApps are uniquely identifiable and bound to credentials during their lifecycle.
- During xApp registration, each xApp ID must be associated with authentication credentials `(REQ-SEC-XAPP-3)`
- xApp identifiers must be globally unique `(REQ-SEC-XAPP-4)`

xApps cannot be anonymous. Identity collisions or impersonation must be prevented at the platform level.

##### 1.1.1.2 Near-RT RIC Access Control & Data Protection

The Near-RT RIC must enforce strong access control and data protection internally.

- The Near-RT RIC shall support authentication and authorization for data access
`(REQ-SEC-NEAR-RT-1)`
- Data access control must follow the principle of least privilege
`(REQ-SEC-NEAR-RT-2)`
- Near-RT RIC databases must provide confidentiality and integrity protection
`(REQ-SEC-NEAR-RT-2A)`


Even authenticated xApps should only access the minimum data and services required for their function.

##### 1.1.1.3 xApp ↔ Near-RT RIC API Security
Communication between xApps and the Near-RT RIC platform APIs must be strongly protected.
- Communication shall be mutually authenticated
`(REQ-SEC-NEAR-RT-3)`
- The Near-RT RIC shall provide an authorization framework for API consumption, aligned with operator policies
`(REQ-SEC-NEAR-RT-4)`

Authentication alone is insufficient—authorization must be enforced per service and per xApp.

##### 1.1.1.4 A1 Interface Authorization & Resilience
WG11 defines security expectations for the A1 interface, including resilience against attacks.
- Near-RT RIC shall support authorization as:
    - A1 Policy consumer/server `(A1-P)`
    - A1 Enrichment Information client `(A1-EI)`
`(REQ-SEC-NEAR-RT-5)`
- Near-RT RIC shall be able to recover from volumetric DDoS attacks across the A1 interface without catastrophic failure
`(REQ-SEC-NEAR-RT-6)`

The Near-RT RIC must remain operational even under attack conditions.

##### 1.1.1.5 Content-Based Attack Detection (A1, Y1, E2)

WG11 explicitly requires protection against content-related attacks, even when messages are protocol-compliant.
- Detect and defend against content-related attacks across:
    - A1 interface `(REQ-SEC-NEAR-RT-7)`
    -  Y1 interface `(REQ-SEC-NEAR-RT-8)`
    - E2 interface `(REQ-SEC-NEAR-RT-9)`
- WG11 notes that common examples include:
    - Injection attacks
    - Buffer overflow attacks

Security mechanisms must consider runtime behavior and traffic patterns, not only protocol validity.

##### 1.1.1.6 Interface-Level Cryptographic Protection
WG11 mandates standard cryptographic protections for critical interfaces.

- A1 interface shall support:
    - Confidentiality
    - Integrity
    - Replay protection `(REQ-SEC-NEAR-RT-10)`
- Mutual authentication is required on the A1 interface `(REQ-SEC-NEAR-RT-11)`

##### 1.1.1.7 Data Protection Across the Lifecycle

The Near-RT RIC shall protect data:
- At rest
- In use
- In transit
`(REQ-SEC-NEAR-RT-12)`

Security must cover the entire data lifecycle, not only network transport.

#### 1.1.2 Security Controls
This subsection describes the security controls defined for the Near-RT RIC APIs and related procedures, as specified in `O-RAN.WG11.TS.SRCS`. The controls are identified using WG11 security control identifiers (SEC-CTL-NEAR-RT-x) and are mapped to the protection objectives of authentication, authorization, confidentiality, integrity, and replay protection.

##### 1.1.2.1 API Security - Authentication
Authentication of Near-RT RIC APIs is specified under `SEC-CTL-NEAR-RT-1`, `SEC-CTL-NEAR-RT-2`, and `SEC-CTL-NEAR-RT-2A`.
</br>
`SEC-CTL-NEAR-RT-1` requires that transactional APIs (REST and gRPC) support mutual TLS (mTLS) authentication using `X.509v3` certificates. This ensures bidirectional authentication between the Near-RT RIC platform and xApps, preventing unauthorized API access and impersonation.
</br>
For time-critical APIs that cannot rely on TLS, `SEC-CTL-NEAR-RT-2` mandates the use of IPsec with IKEv2 certificate-based authentication. In this case, authentication implicitly grants authorization due to protocol limitations (e.g., SCTP-based interfaces such as E2).
</br>
Additionally, `SEC-CTL-NEAR-RT-2A` explicitly requires mTLS support on the A1 interface, ensuring mutual authentication between Near-RT RIC and Non-RT RIC.

##### 1.1.2.2 API Security - Authorization
Authorization controls for Near-RT RIC APIs are defined primarily under `SEC-CTL-NEAR-RT-3`, `SEC-CTL-NEAR-RT-3A`, `SEC-CTL-NEAR-RT-3B`, and `SEC-CTL-NEAR-RT-3C`. </br>

`SEC-CTL-NEAR-RT-3` specifies that transactional APIs (REST and gRPC) shall support the OAuth 2.0 authorization framework, with:
- Near-RT RIC platform modules acting as resource owner / resource server
- xApps acting as OAuth clients
- Client Credentials Grant used, combined with mTLS for client authentication

`SEC-CTL-NEAR-RT-3A` and `SEC-CTL-NEAR-RT-3B `enforce the **principle of least privilege**, requiring that xApps are granted only the minimal set of permissions necessary to perform authorized actions.

`SEC-CTL-NEAR-RT-3C` defines mandatory authorization enforcement in key Near-RT RIC API procedures, including:
- API discovery (restricting visibility of published APIs)
- E2 Subscription API (restricting xApps to specific E2 Nodes and RAN functions)
- E2 Control API (allowing only authorized xApps to initiate control actions)
- E2 Guidance API (restricting access to conflict mitigation guidance)
- SDL API procedures (requiring successful registration and authorization prior to use)

##### 1.1.2.3 API Security - Confidentiality & Integrity
Confidentiality and integrity protections for Near-RT RIC APIs are defined under `SEC-CTL-NEAR-RT-6` and `SEC-CTL-NEAR-RT-7`.
</br>
`SEC-CTL-NEAR-RT-6` mandates that transactional APIs (REST and gRPC) support TLS to provide encryption and integrity protection for messages exchanged between the Near-RT RIC platform and xApps.
</br>
For time-critical APIs where TLS is not supported, `SEC-CTL-NEAR-RT-7` requires the use of IPsec, ensuring equivalent confidentiality and integrity guarantees at the network layer. Together, these controls ensure that API communications cannot be eavesdropped, modified, or replayed without detection.

<p align="center">
    <img src="../../NTN/notes-png/Table 5.1.png" alt="Summary of the Security Controls for Near-RT RIC APIs" />
    <br />
    <em>Table 5.1: Summary of the Security Controls for Near-RT RIC APIs</em>
</p>

Table 5.1 summarizes the mapping between API transport protocols and the applicable security controls `SEC-CTL-NEAR-RT-1` through `SEC-CTL-NEAR-RT-7`.

Specifically:

- gRPC and REST APIs rely on mTLS for authentication and OAuth 2.0 for authorization, with confidentiality and integrity provided by TLS.
- SCTP-based APIs (e.g., E2) rely on IKEv2/IPsec, providing authentication, confidentiality, and integrity at the network layer, with no native authorization mechanism.

This table demonstrates the protocol-aware application of Near-RT RIC security controls rather than a one-size-fits-all approach.

##### 1.1.2.4 Security Controls For The `Y1` Interface Protocol Structure Solution 1
Security controls applied to the `Y1` interface are derived from the general Near-RT RIC API security framework, particularly `SEC-CTL-NEAR-RT-1`, `SEC-CTL-NEAR-RT-3`, and `SEC-CTL-NEAR-RT-6`. </br>
Under Solution 1, the `Y1` protocol structure integrates:
- Authentication of Y1 consumers using certificate-based mechanisms
- Authorization enforcement to restrict access to exposed RAN analytics
- Confidentiality and integrity protection for analytics data in transit

These controls ensure that Y1 analytics exposure is limited to authorized consumers and protected against interception or manipulation.

##### 1.1.2.5 xApp Registration - Security Procedure
The xApp registration security procedure is closely related to `REQ-SEC-XAPP-3`, `REQ-SEC-XAPP-4`, and the enforcement mechanisms defined in `SEC-CTL-NEAR-RT-1` and `SEC-CTL-NEAR-RT-3`.

<p align="center">
    <img src="../../NTN/notes-png/Figure 5.1.3.2 2.png" alt="" />
    <br />
    <em>Figure 5.1.3.2 2: Security procedure for xApp registration</em>
</p>

During registration:

- The xApp identifier (xApp ID) is securely associated with authentication credentials (`REQ-SEC-XAPP-3`)
- xApp IDs are created to ensure uniqueness (`REQ-SEC-XAPP-4`)
- The Near-RT RIC validates the xApp’s identity before granting API access
- Authorization policies are bound to the xApp prior to service consumption

Only xApps that successfully complete this procedure are permitted to access Near-RT RIC APIs, ensuring that all API interactions originate from authenticated and authorized applications, as illustrated in Figure 5.1.3.2-2.

### 1.2 E2 Interface
The E2 interface connects the Near-RT RIC to E2 Nodes and is classified by WG11 as a time-critical interface. Due to protocol constraints, E2 does not support application-layer security mechanisms such as TLS or OAuth.

Authentication for the E2 interface is mandated by `SEC-CTL-NEAR-RT-2`, which requires IPsec with IKEv2 and certificate-based authentication. Successful authentication implicitly authorizes the E2 Node to communicate with the Near-RT RIC.

Confidentiality and integrity for E2 traffic are provided by `SEC-CTL-NEAR-RT-7`, which mandates IPsec protection of all E2 messages. This prevents eavesdropping, message tampering, and replay attacks.

Fine-grained authorization is not supported on the E2 interface; therefore, security relies on network-layer protection and strict trust boundary enforcement.

### 1.3 Trust boundaries defined by WG11
WG11 defines explicit trust boundaries to separate components operating under different trust assumptions.

A trust boundary exists between the Near-RT RIC platform and xApps. xApps are treated as untrusted workloads, requiring strong authentication (`SEC-CTL-NEAR-RT-1`), authorization (`SEC-CTL-NEAR-RT-3`), and transport security (`SEC-CTL-NEAR-RT-6`) for all API access.

Another trust boundary exists between the Near-RT RIC and E2 Nodes. E2 Nodes are outside the Near-RT RIC trust domain, and all communication across this boundary must be protected using IPsec as defined in `SEC-CTL-NEAR-RT-2` and `SEC-CTL-NEAR-RT-7`.

A further trust boundary is defined across the A1 interface between the Near-RT RIC and Non-RT RIC, where mutual authentication using mTLS is required under `SEC-CTL-NEAR-RT-2A`.

These trust boundaries enforce a zero-trust security model, requiring explicit security controls at every interface crossing.

---

## 2. Security Test Categories & Scenarios (@SPEC2 & @SPEC3)
WG11 defines security testing not as classical penetration testing, but as behavioral validation of O-RAN components under realistic operational and threat conditions. The focus is on whether components continue to enforce security assumptions while remaining standards-compliant and authenticated.

###  Runtime behavior testing
Runtime behavior testing evaluates how O-RAN components behave during normal operation under dynamic, potentially adversarial conditions, rather than static configuration checks.

According to @SPEC2, this category focuses on validating:
- Security enforcement after successful onboarding
- Behavior under valid but unexpected runtime inputs
- Correct handling of load, timing, and state transitions

For example, WG11 highlights the importance of testing whether:
- An xApp can continuously issue control messages that legally conform to E2AP but cause scheduler instability at the O-DU
- Near-RT RIC enforces rate limiting, prioritization, and isolation between multiple xApps at runtime
- Resource exhaustion scenarios (CPU, memory, threads) are detected and mitigated without service collapse

### Interface misuse cases
Interface misuse testing focuses on legitimate interfaces being used in unintended or abusive ways, without violating syntax, authentication, or protocol compliance.

WG11 emphasizes that O-RAN’s openness increases the risk of @SPEC3:
- Semantic misuse
- Protocol edge-case exploitation
- Abuse of optional or vendor-specific behaviors

Typical interfaces under test include:
- E2 (Near-RT RIC ↔ O-DU / O-CU)
- A1 (Non-RT RIC ↔ Near-RT RIC)
- O1 (SMO ↔ managed nodes)
- Open Fronthaul management plane

Examples of interface misuse scenarios include:
- Repeated E2 control actions that remain standards-compliant but lead to oscillatory RAN behavior
- Abuse of optional fields in E2AP messages to bypass expected validation logic
- A1 policy updates that conflict or overlap, causing ambiguous enforcement at Near-RT RIC
- Excessive or malformed O1 telemetry requests that degrade management-plane availability

WG11 explicitly treats these as security-relevant, even though:
- TLS is intact
- Certificates are valid
- Messages are syntactically correct

The test objective is to verify that:
- Interfaces enforce semantic validation
- Rate limits and sanity checks exist
- One component cannot degrade system behavior by exploiting interface flexibility

### Misbehaving but authenticated components
This category is central to WG11’s threat model.
WG11 assumes that authentication alone is insufficient in an open, multi-vendor RAN ecosystem. A component may:
- Be properly certified
- Hold valid credentials
- Pass onboarding and integrity checks

but sill behave maliciously or negligently

Misbehaving component testing evaluates scenarios where:
- An xApp intentionally or unintentionally issues harmful control logic
- A Near-RT RIC forwards excessive or conflicting commands
- An O-DU reports misleading or selectively incomplete telemetry

Key assumptions defined by WG11:
- The attacker may control a trusted component
- Attacks originate inside the trust boundary
- Behavior deviates from intent, not from protocol compliance

Concrete examples include:
- An authenticated xApp continuously overriding scheduler decisions to starve specific UEs
- A compromised RIC manipulating E2 indications to mislead higher-layer optimization logic
- An O-DU accepting control inputs but applying them selectively or inconsistently

Security testing in this category validates:
- Component isolation (xApp sandboxing)
- Policy conflict resolution
- Kill-switches and revocatio mechanisms
- Detection of anomalous but valid behavior patterns

---

## 3. Threat Assumptions & Attacker Model (@SPEC3)
This section defines the security assumptions, attacker capabilities, and scope boundaries used by WG11 to design and evaluate O-RAN security test scenarios. The threat model explicitly acknowledges that O-RAN introduces new threat surfaces due to architectural openness, functional disaggregation, and cloud-native deployment.

### Assumed attacker capabilities
WG11 assumes attackers with varying levels of capability, access, and intent, ranging from low-skill external actors to highly resourced nation-state adversaries. The attacker is not limited to external threats; internal and authenticated entities are explicitly considered.

Based on the defined threat agents, assumed attacker capabilities include:
- Ability to exploit open and standardized interfaces (A1, E2, O1, O2, Open Fronthaul, Y1, R1)
- Capability to operate as an authenticated but misbehaving component, such as a compromised xApp, rApp, or management function
- Knowledge of O-RAN architecture, interfaces, and message flows
- Ability to launch attacks targeting Confidentiality, Integrity, and Availability (CIA) objectives
- Exploitation of software vulnerabilities, including those introduced through open-source components
- Capability to abuse virtualization and container orchestration layers (e.g., poor isolation, misconfiguration in O-Cloud)

WG11 explicitly includes threat agents such as:
- Cyber-criminals and script kiddies (limited sophistication, opportunistic)
- Insiders (authorized access, high impact)
- Hacktivists and cyber-terrorists (disruptive intent)
- Nation-state actors (advanced persistence, strategic objectives)

The model therefore assumes that authentication, encryption, and onboarding alone do not eliminate risk.

### Preconditions for security tests
WG11 security testing is performed under a set of explicit preconditions to ensure realism and consistency.

The following preconditions are assumed for all security tests:
- O-RAN components (SMO, RICs, O-CU, O-DU, O-RU, O-Cloud) are:
    - Properly deployed
    - Successfully onboarded
    - Authenticated using valid credentials
- Secure transport mechanisms (e.g., TLS/IPsec where specified) are enabled
- Interfaces conform to O-RAN and 3GPP specifications
- Components are operational and actively exchanging control and user-plane data

These preconditions intentionally exclude trivial failures such as:
- Missing certificates
- Disabled encryption
- Non-compliant protocol implementations

The objective is to test whether the system remains secure after trust is established, particularly under:
- Runtime stress
- Conflicting control logic
- Malicious-but-valid behavior

This reflects WG11’s assumption that most realistic attacks occur post-compromise or within trusted domains.

### In-scope vs out-of-scope attacks
WG11 clearly defines the scope boundaries of its threat analysis and security testing to focus on O-RAN–specific risks.

#### In-scope attacks
The following attack classes are considered in scope:
- Attacks exploiting O-RAN-specific interfaces (A1, E2, O1, O2, Open Fronthaul, Y1, R1)
- Threats originating from:
    - Inside the O-RAN system (e.g., compromised xApp, insider misuse)
    - Outside the system but targeting exposed interfaces
- Attacks against:
    - RAN availability (e.g., scheduler disruption, control-loop instability)
    - Data confidentiality (e.g., x/rApp access to subscriber or network data)
    - Integrity of control and synchronization planes
- Exploitation of:
    - Trust-chain weaknesses caused by disaggregation
    - Virtualization and container orchestration misconfigurations
    - Open-source software vulnerabilities
    - STRIDE-classified threats, including spoofing, tampering, repudiation, information disclosure, denial of service, and privilege escalation

These attacks are evaluated using the STRIDE framework and mapped to impacted assets and security objectives.

#### Out-of-scope attacks
The following are explicitly or implicitly out of scope for WG11 testing:
- Physical attacks on hardware (e.g., antenna tampering, site intrusion)
- RF-layer attacks unrelated to O-RAN architectural changes
- Core Network–only attacks not involving RAN or O-RAN interfaces
- Generic Internet-scale DDoS attacks not specific to O-RAN components
- Attacks requiring violation of basic deployment assumptions (e.g., no authentication, plaintext management traffic)

WG11 limits scope to ensure focus on architectural and systemic security implications introduced by O-RAN, rather than general telecom security issues.

---

## 4. Testing Logic & Objectives
WG11 security testing is designed to validate system-level security behavior of O-RAN architectures rather than individual component robustness in isolation. The emphasis is on how components interact, respond, and enforce trust assumptions at runtime under realistic operational conditions.

### What WG11 wants to validate
WG11 testing aims to validate that O-RAN components and interfaces behave securely after onboarding, authentication, and integration, and that the system remains resilient to threats introduced by openness and disaggregation.

Specifically, WG11 seeks to validate that:
- O-RAN components correctly enforce security boundaries despite functional disaggregation
- Open interfaces (A1, E2, O1, O2, Open Fronthaul, Y1) do not introduce uncontrolled propagation paths for attacks
- Near-RT RIC and xApps do not destabilize RAN behavior through valid but harmful control actions
- Trust relationships remain enforceable even when components are:
  - Multi-vendor
  - Cloud-hosted
  - Dynamically instantiated
- Security controls remain effective against:
    - Authenticated but misbehaving components
    - Runtime misuse of interface
    - Internal threat agents

WG11 does not aim to prove absolute security; instead, it validates that expected security assumptions hold under stress and misuse scenarios.

### What “secure behavior” means in tests
In WG11 context, secure behavior is defined behaviorally, not cryptographically.

A system is considered to behave securely if it:
- Maintains availability of RAN services under abnormal but standards-compliant inputs
- Prevents a single component (e.g., xApp, O-DU) from:
    - Exercising excessive control authority
    - Overriding system-wide policies
- Enforces semantic validation on control and management interfaces
- Detects and mitigates anomalous behavior, even when:
    - Messages are authenticated
    - Protocol syntax is correct
- Preserves confidentiality and integrity of:
    - Subscriber data
    - Network telemetry
    - Control-plane signaling
- Secure behavior therefore includes:
    - Rate limiting
    - Isolation between applications and functions
    - Conflict resolution mechanisms
    - Graceful degradation instead of cascading failure

WG11 explicitly recognizes that a secure system may still experience degradation, but must avoid uncontrolled or systemic collapse.

### Conceptual test flow (not implementation)
WG11 defines a conceptual testing logic that abstracts away from specific tools, vendors, or lab environments.

The conceptual test flow is as follows:
1. Establish Trusted Baseline
    - Deploy O-RAN components in a compliant configuration
    - Ensure authentication, authorization, and secure transport are enabled
2. Introduce Controlled Threat Conditions
    - Simulate misbehaving but authenticated components
    - Apply interface misuse scenarios using valid protocol messages
3. Observe runtime interactions
    - Monitor cross-component effects through open interfaces
    - Identify propagation of abnormal behavior across planes
4. Evaluate security responses
    - Verify enforcement of limits, isolation, and policy controls
    - Confirm absence of uncontrolled escalation or instability
5. Assess system-level impact
    - Determine whether confidentiality, integrity, and availability are preserved
    - Evaluate whether degradation remains localized and recoverable
6. Validate assumptions
    - Confirm that the original threat and trust assumptions remain valid
    - Identify gaps where assumptions fail under realistic conditions

This flow reflects WG11’s philosophy that security emerges from interaction control, not from isolated component hardening.

---

## 5. Near-RT RIC Security Focus
Specific observations for near-RT RIC:

### xApps are not fully trusted at runtime
### E2 interface is a major attack surface
### Security must not break near-real-time constraints
### Monitoring and anomaly detection are preferred over heavy inspection

---

## 6. Testing Perspective 
This section identifies **testable security aspects** from WG11 specs.

### Authentication failures
### Unauthorized access attempts
### Abnormal interface traffic
### Certificate misconfiguration
### Policy violations (RBAC)
### Logging and alert generation

---
