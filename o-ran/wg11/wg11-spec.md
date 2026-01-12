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

## 2. Security Test Categories & Scenarios @SPEC3



###  Runtime behavior testing
### Interface misuse cases
### Misbehaving but authenticated components

---

## 3. Threat Assumptions & Attacker Model
### Assumed attacker capabilities
### Preconditions for security tests
### In-scope vs out-of-scope attacks

---

## 4. Testing Logic & Objectives
### What WG11 wants to validate
### What “secure behavior” means in tests
### Conceptual test flow (not implementation)

WG11 assumes **protocol-compliant but malicious behavior** is possible.

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
