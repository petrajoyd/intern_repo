# WG-11 Cybersecurity Fundamentals
WG11 is about making sure that opening up the RAN doesn’t open up security holes. It sets the ground rules so multiple vendors, cloud platforms, and AI applications can work together without breaking trust or safety.
WG11 does not implement security mechanisms itself. Instead, it:
- Defines what must be protected
- Identifies where threats can occur
- Specifies how trust should be established and maintained
- Ensures security is consistent across all O-RAN components and interfaces

These fundamentals guide security decisions for:
- O-RAN network elements (O-RU, O-DU, O-CU, RIC, SMO)
- O-RAN interfaces (Open Fronthaul, E2, A1, O1, O2)
- Cloud-native platforms and AI-based applications

## WG11 Purpose
WG11 (Working Group 11) in the O-RAN Alliance focuses on security across the entire O-RAN ecosystem. Its main goal is to make sure that an open, disaggregated, and multi-vendor RAN can still be deployed securely, without introducing new risks compared to traditional (closed) RAN systems.
</br>
Because O-RAN opens interfaces, virtualizes network functions, and introduces AI-based control, WG11 defines security requirements, architectures, and principles that apply across components, interfaces, and lifecycle stages (deployment, operation, and updates).

### 1. Security challenges of open, multi-vendor RAN
In traditional RAN, most components come from a single vendor, so security assumptions are centralized. In O-RAN, this changes. </br>
The key challenges usually happens because: 
1.  Multiple vendors supply O-RAN Components (O-RU, O-DU, etc), hence trust cannot be assumed between components
2.  Open interfaces exposed more attack surfaces
3.  More disaggregated architecture means more signaling, more APIs, and more things to protect.
4.  Different security maturity levels across different vendors makes the vulnerable ones become the attack point.

What WG11 tries to do:
1. Define common security requirements across different vendors
2. Ensure interoperability without sacrificing security
3. Avoid vendor lock-in without increasing risk

### 2. Cloud-native & AI-driven risks
O-RAN is built on cloud-native principles, and this introduces a new class of risks that don't exist (or are linited) in traditional RAN.</br>
Thoe risks are:
1. Network functions run as VMs or containers which means its vulnerable to container escapes, and misconfigurations.
2. Heavy use of Kubernetes and orchestration systems means control-plane compromise can impact the whole RAN
3. Dynamic scaling and lifecycle management makes onboarding or updates can intoduce malicious components

AI-driven risks
1. RIC uses AI/ML models to control RAN behavior. Risks include
    - Model poisoning (training data manipulation)
    - Malicious or buggy xApps/rApps
    - Unintended control actions affecting network performance or availability.
2. AI decisions might be hard to explain, hard to audit. Which makes the operational and security complexity increaces.

WG11 jobs are to:
- Apply Zero Trust principles even inside the RAN
- Secure the AI lifecycle (model, data, deployment)
- Define controls and monitoring for cloud-native components

## WG11 Scope
WG11 scope defines what areas of security WG11 is responsible for in the O-RAN ecosystem. Instead of focusing on a single component, WG11 looks at security end-to-end, covering architecture, interfaces, cloud infrastructure, and the supply chain.

The scope is broad because O-RAN itself is highly distributed, multi-vendor, and cloud-native.

### 1. Threat modeling
Threat modeling is about understanding what can go wrong before it actually happens.

In O-RAN:
- There are many components (O-RU, O-DU, O-CU, RIC, SMO)
- Many open interfaces
- Multiple vendors and cloud platforms

WG11 uses threat modelling to:
- Identify attack surfaces across the RAN
- Understand who the attackers might be (external, insider, compromised vendor)
- Analyze how attacks could propagate across components
- Prioritize risks based on impact and likelihood

**You can’t secure everything equally**, so threat modeling helps decide what matters most.

### 2. Security architecture
Security architecture defines how security is structured across O-RAN, not just individual mechanisms.

WG11 security architecture focuses on:
- End-to-end protection across RAN components
- Clear security boundaries between functional blocks
- Consistent security behavior across vendors
- Integration of security into management, control, and user planes

### 3. Trust & identity
In a multi-vendor RAN, trust cannot be assumed.
WG11 addresses:
- How components identify each other
- How trust is established, verified, and revoked
- How identities are managed across vendors and domains

WG11 adresses those things because each network function must have a verifiable identity, trust have to be explicit and incompromiseable. 

### 4. Interface protection
O-RAN relies heavily on open interfaces, which are both a strength and a risk.
WG11 focuses on securing interfaces such as:
- Open Fronthaul
- E2
- A1
- O1
- O2

Interface protection includes:
- Authentication (who is allowed to connect)
- Authorization (what they are allowed to do)
- Encryption (protecting data in transit)
- Integrity protection (preventing tampering)

### 5. Cloud & container security
Most O-RAN functions run on cloud infrastructure using VMs and containers.

WG11 scope covers:
- Secure deployment of containers and VMs
- Isolation between workloads
- Protection of orchestration platforms (e.g., Kubernetes)
- Secure onboarding and lifecycle management of network functions

Risks WG11 considers:
- Misconfiguration
- Privilege escalation
- Compromised images or containers

The idea is to secure both the RAN functions and the platform they run on.

### 6. Supply-chain security
O-RAN introduces a long and complex supply chain:
- Hardware vendors
- Software vendors
- Open-source components
- AI/ML models and datasets

WG11 addresses:
- Integrity and authenticity of software and firmware
- Risks from third-party or open-source components
- Malicious updates or backdoors
- Dependency and version management

## Core Security Principles
These principles guide how security should be implemented across the O-RAN ecosystem.
They exist because O-RAN is open, cloud-native, multi-vendor, and distributed, which breaks traditional “trusted network” assumptions.

Think of these as rules of survival for open RAN.

### 1. Zero Trust
Never trust, always verify, in O-RAN
- No component is trusted by default
- Being “inside the network” means nothing
- Every access request must be verified

The whole idea is to state that:
- Trust is explicit, not assumed
- Identity matters more than location
- Access is continuously evaluated

Practical meaning:
- O-DU doesn’t trust O-CU just because it’s part of the same RAN
- RIC apps don’t get full access just because they’re deployed
- Every interaction is checked against policy

Why WG11 needs this:
- Open interfaces + multiple vendors = trust must be earned, not assumed

### 2. Mutual authentication
Mutual authentication means both sides verify each other’s identity. Simply all the components have to say “I trust you because I verified who you are — and you verified me too”

In O-RAN:
- Network functions authenticate each other
- Controllers authenticate controlled elements
- RIC apps authenticate platforms and vice versa

Benefits:
- Prevents impersonation attacks
- Stops rogue or fake components
- Strengthens Zero Trust enforcement

Typical mechanisms:
- Certificates
- Cryptographic identities
- Secure key exchange

### 3. Encryption
Encryption protects data confidentiality and integrity.
WG11 applies encryption to:
- Data in transit (between network functions)
- Data at rest (stored configurations, logs, models)
- Sensitive management and control traffic

Why it matters:
- Open interfaces expose traffic paths
- Cloud infrastructure may be shared
- Attackers may observe or tamper with traffic

Core goals:
- Prevent eavesdropping
- Prevent data manipulation
- Maintain trust across interfaces

### 4. RBAC
RBAC ensures that entities only get the permissions they actually need. Or in other word “You’re in, but only allowed to do this”.

In O-RAN:
- RIC apps have limited scopes
- Operators, admins, and services have different roles
- Automation tools don’t get human-level privileges

Why WG11 emphasizes RBAC:
- Reduces blast radius if something is compromised
- Prevents accidental misconfigurations
- Makes auditing and compliance easier

### 5. Secure onboarding
Secure onboarding is about how new components join the system safely.

This includes:
- New network functions
- New vendors
- New RIC applications
- Software updates and upgrades

WG11 focuses on:
- Identity verification before onboarding
- Trust establishment at first contact
- Policy enforcement from day one

Why this is critical:
- Compromised onboarding = compromised network
- First access is the most dangerous moment

### 6. Continuous monitoring
Security is not a one-time setup.

WG11 requires:
- Continuous visibility into system behavior
- Detection of anomalies and suspicious activity
- Logging, alerting, and auditing

In O-RAN:
- AI-driven systems can behave unexpectedly
- Cloud environments change dynamically
- Attacks may happen long after deployment

Continuous monitoring enables:
- Early attack detection
- Policy enforcement over time
- Feedback for improving security posture

